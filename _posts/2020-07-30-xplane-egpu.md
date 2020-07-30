---
layout: post
title: "X-Plane on an eGPU"
date: 2020-07-30
tags: [xplane, egpu, linux]
# last_modified_at:
---

As a bit of a followup to the previous post, I would like to go over a few of my experiences running the eGPU with an actual application. The last post focused more on getting the eGPU to actually work with the Linux kernel and work with GNOME in general. This post will focus on how the eGPU performs with a relatively graphics-heavy application, X-Plane.

I have been an aviation enthusiast for a very long time. I became very interested in X-Plane a number of years ago since it supports Linux natively and is actively being developed.

<figure>
  <div class="image-container">
    <img src="/assets/xplane-egpu/airbus-landing.jpg" alt="Landing of an Airbus A319">
  </div>
</figure>

## Drivers

Embarking on this journey was timed a bit strategically. X-Plane originally only supported the OpenGL graphics API. It’s cross-platform but very, very old. A new beta, only released about a month-or-so ago, introduces support for a newer graphics API called Vulkan.

### OpenGL

I started working with OpenGL since I had always run the simulator on it before.

As mentioned in the last post, if the sane Linux defaults are used my Intel iGPU is set as the primary GPU and any application run is either rendered and composited on the iGPU and piped to the eGPU (without `DRI_PRIME=1` set) or is rendered on the eGPU, composited on the iGPU and then piped back to the eGPU for display (with `DRI_PRIME=1` set). Well, as you can imagine, this is not efficient at all. Running X-Plane with `DRI_PRIME=1` not only made the desktop lag (probably due to the communication bottleneck over the Thunderbolt connection) but also only achieved marginally better results than running directly on the iGPU, about 10-12 fps.

Running X-Plane on X11 with the eGPU set as the primary GPU achieved much better results, around 15-20 fps; better than the iGPU but not nearly enough to run the simulator in real-time (30 fps minimum). On Windows I achieved very similar results which hint that this is just due to limitations of the GPU itself (the RX 5500 XT is not a very powerful card).

### Vulkan

Vulkan was where things got interesting. First off, on Windows it performed perfectly. I was getting above 30 fps on the ground and even an amazing 60 fps at times in the air.

To make Vulkan backwards compatible with OpenGL drawing extensions, X-Plane makes use of a number of Vulkan calls that draw like OpenGL did and then synchronize the drawing of those plugins with the rest of the simulator that draws with Vulkan. Turns out [this is royally broken](https://forums.x-plane.org/index.php?/forums/topic/220903-1150b14-release-notes-its-now-out/&page=13&tab=comments#comment-1989395) on Linux with the AMDPRO drivers:

> "...all AMD drivers are fundamentally broken in how they do Vulkan/GL interop... Nvidia driver implements this correctly and has so for over a year at this point when I first started working on this piece of code."

> "So, as far as I can tell, AMD gets the synchronization part wrong."

Well, that's disappointing, to say the least. Essentially AMDGPU's OpenGL drivers are all broken when working with Vulkan. I first thought this was a Vulkan driver problem, but this seems to be present with Mesa's RADV implementation and AMD's AMDVLK implementation. Mesa [has a bug report](https://gitlab.freedesktop.org/mesa/mesa/-/issues/3257), and like mentioned in the post linked above, "the good news is that fixing this might be easier cuz it's open source and maybe someone who understand how to emit PM4 packets can fix things without waiting for AMD."

I was able to get Vulkan working if I disabled all plugins that draw and only used planes that were only 100% Vulkan compatible. The first time I did this I was able to achieve an even more impressive 40 fps on the ground.

<figure>
  <div class="image-container">
    <img src="/assets/xplane-egpu/cessna-vulkan.png">
  </div>
  <figcaption>Cessna in 40 fps in a stripped-down demo instance of X-Plane</figcaption>
</figure>

## AMDGPU-PRO

Supposedly the AMDGPU-PRO proprietary AMD drivers are not affected by this bug. For a short time, I was able to use the AMDGPU-PRO drivers to get even OpenGL drawing plugins to work with mixed results. The extremely popular [Zibo mod 737](https://forums.x-plane.org/index.php?/forums/topic/138974-b737-800x-zibo-mod-info-installation-download-links/) was able to draw perfectly and worked perfectly as far as I could tell. ToLiss's A319 did not work as well:

<figure>
  <div class="image-container">
    <img src="/assets/xplane-egpu/airbus-broken-vulkan.png">
  </div>
  <figcaption>A very broken A319 in Vulkan</figcaption>
</figure>

But, of course, we can't do anything about this since the drivers are proprietary. The proprietary drivers also broke after a reboot, which caused mutter to give up entirely. So, it had to go.

## Conclusion

Even with all the problems found, I feel driver support is bound to get immensely better. AMD every year gets more and more competitive with Nvidia, and their commitment to open-source software means that these problems can be tracked and worked on by a large number of people. I could have used an Nvidia card as an eGPU and largely bypassed most of the Vulkan problems by using the proprietary drivers, but that wouldn’t have aligned with what I value in open-source software.