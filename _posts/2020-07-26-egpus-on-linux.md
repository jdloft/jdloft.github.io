---
layout: post
title: "eGPUs on Linux"
date: 2020-07-29
tags: [egpu, linux]
# last_modified_at:
---

Hardware support on Linux can be hit-or-miss at times. Although it has gotten significantly better over the years, one aspect continues to expose the wild west nature of drivers on Linux, GPUs.

If you aren't aware, external GPUs are a way to connect a desktop GPU to a laptop via a Thunderbolt interface. Thunderbolt 3 can support up to 40 Gbps on a four lane port
and 20 Gbps on a two lane port which ends up being enough to run most decent GPUs at a decent rate.

I bought a Sonnet eGFX Breakaway Box 350 and an AMD RX 5500 XT. I went with AMD because I thought the presence of open source drivers in Mesa would perhaps ease the use of the device. I was wrong.

## Hotplugging

First of all, hot plugging is a no-go. This isn't all that surprising. The PCI interface was never intended to be hot pluggable when it was first developed. After authorizing the device all that happens is the AMDGPU driver spits
out a few errors about initializing the device and then dies. Hot unplugging is even worse; it freezes the whole machine unless the AMDGPU driver is completely removed by running `rmmod`.

If the enclosure is plugged in while booting it initializes correctly and GDM even starts displaying on a display connected to it. After logging in I found I could close the lid to my laptop and use the device connected to
a monitor, external keyboard and mouse and largely forget that I was using a laptop at all. Success!

## Switching the primary GPU

Or not. What I didn't initially recognize is that all applications were still being rendered and composited on the internal Intel graphics card on the CPU. Even worse, on Wayland everything worked okay but on X11 when the laptop lid was shut the internal graphics card went into low power mode and the performance dropped to an unusable level. One guy on the eGPU.io forums [explained it perfectly](https://egpu.io/forums/thunderbolt-linux-setup/linux-wayland-need-to-know-current-state-how-to-and-if-it-benefits-egpu/)

> With no xorg configuration files in /etc/X11/…., it manages to autoconfigure to a working state, normally, which is useful for eGPUs where the configuration changes (on the other hand, if you specify configuration, and it cannot find said device on said bus, it just implodes, instead of ignoring and moving on). However, it does seem to select a primary GPU. If you change monitor configuration to only display on the monitor on the primary GPU, it runs perfectly smooth - even with eGPUs. If you mirror, it becomes a tiny bit jumpy. If you extend, it is a little slower on the monitor on the secondary GPU. If you select the monitor on the secondary GPU alone, it becomes a literal slideshow (sub-1 FPS and completely unusable). I am guessing it is rendering on the primary GPU which goes into low power mode when its monitor is switched off, making it unusable.

Obviously if the desktop comes to a complete standstill if the laptop's iGPU goes into low power it means that the iGPU is still being used for some part of the rendering process. I could start specific applications on the eGPU by setting `DRI_PRIME=1` but I was still getting some performance hit due to this bottleneck.

[A bug report for mutter](https://gitlab.gnome.org/GNOME/mutter/-/issues/348) seemed to confirm this. "Currently Mutter cannot put anything on composite-bypass on secondary GPUs (the eGPU), which means there is always a copy from iGPU to the eGPU." Well that's no good. Regardless of any program running, ones completely rendered on the iGPU and ones on the eGPU via `DRI_PRIME` would be bottlenecked by compositing on the iGPU.

There were solutions. None of which were particularly elegant.

1. I could disable the iGPU entirely by blacklisting the Intel i915 module on boot by appending `modprobe.blacklist=i915` to the kernel parameters and use either Wayland or X11
2. I could just use X11 and swap out my `xorg.conf` file to specify the iGPU or the eGPU as the primary GPU

Number 1 wasn't great because the internal display would be useless. Well, it was useless either way but I didn't like the feeling of completely disabling the module.

Number 2 was made much more elegant by employing the use of a script posted on the eGPU.io forums, [egpu-switcher](https://github.com/hertg/egpu-switcher). This script essentially just detected if the eGPU was plugged in on boot, with a systemd service, and linked `/etc/X11/xorg.conf` to one of two different scripts in that same directory, `xorg.conf.egpu` or (optionally, if your internal GPU needs it) `xorg.conf.internal`. All I had to do was run the install script and it installed the service, detected the GPU and wrote `xorg.conf.egpu`:

{% highlight xorg %}
Section "Module"
    Load           "modesetting"
EndSection

Section "Device"
    Identifier     "Device0"
    Driver         "amdgpu"
    BusID          "64:0:0"
    Option         "AllowEmptyInitialConfiguration"
    Option         "AllowExternalGpus" "True"
EndSection
{% endhighlight %}

Now all that was required was to reboot every time I wanted to switch between using the eGPU or the iGPU. Kind of.

The GPU wasn't every really reliable. Sometimes I found it dead in the morning having suffered some sort of nasty error with AMDGPU. Maybe the power went out or the drivers are just unreliable. I'll never know.

## Back to Wayland

But I still really wanted to use Wayland. It is the future™, you know.

Wayland is currently hard coded to prefer hardware rendering devices, then the device that was booted with (set in the BIOS), and then just any other device as fallback.[^1] However, there may be options in the (distant) future.

There is [this pull request](https://gitlab.gnome.org/GNOME/mutter/-/merge_requests/1057) that is a bit of a hack but that would prioritize any external GPU that is hardware accelerated and has an output device attached. It has some problems, as you can see from the comments, but most all of them have to do with hotplugging. And yes, hotplugging is a very big problem. Everything blows up if you unplug the GPU.

[There may be a udev flag](https://gitlab.gnome.org/GNOME/mutter/-/issues/831) added that would allow for prioritizing a certain GPU without changing the underlying, hardcoded logic for everyone.

A solution mentioned in a few of these eGPU-related bug reports, would be to render each application on the GPU that the application shows up on. I don't think this would be ideal; doing so would introduce odd edge cases. What would happen if an application is being dragged between displays? Would it freeze while the buffer is copied to another GPU? What about a window that is partially rendered on two GPUs?

From what I can see on Windows, all applications are indeed rendered on one GPU and then piped to the other. Windows on the other hand is able to hotplug between different GPUs and prioritizes the eGPU. I believe that the ideal solution would to prioritize eGPUs and support switching primary GPUs on-the-fly. This is very difficult[^2] and would take some time.


***

[^1]: <https://gitlab.gnome.org/GNOME/mutter/-/blob/a9a9a0d1c593ee27b449234fd6fbb66e36d5a6ab/src/backends/native/meta-renderer-native.c#L3767-3804>
[^2]: See <https://gitlab.gnome.org/GNOME/mutter/-/merge_requests/1057#note_714200> and <https://gitlab.gnome.org/GNOME/mutter/-/issues/348#note_714183>