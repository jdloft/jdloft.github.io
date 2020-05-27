---
layout: page
title: Styles
---

<p class="message">
  <small>This page serves to document the style of various elements used in the site's design.</small>
</p>

## <small>THIS IS A HEADER</small>

This is the font, size, color and spacing of regular ol' paragraph <p> text. Using Markdown makes it dead simple
to make text *italic*, **bold** or ***bold and italic***.

term
: *definition*

If you want to reference a footnote it appears like this[^1] in a sentence.

<figure class="screenshot">
  <div class="image-container">
    <img src="/assets/sample.jpg" alt="Alt text">
    <span class="image-attribution">Image credit: <a href="http://site.for.attr"><strong>Site</strong></a></span>
  </div>
  <figcaption>This is a caption.</figcaption>
</figure>

Quotes look nice:

>"My fellow Americans, we are and always will be a nation of immigrants. We were strangers once, too."
>
><cite>~ Barack Obama</cite>

So do tables:

| This          | Is A          | Table |
| ------------- |:-------------:| -----:|
| Column 1      | happens to be | left-aligned |
| Column 2      | is the one    |  that's centered |
| Column 3      | is actually    |    right-aligned |

Here are the buttons whose class names and colors can be easily customized:

<span class="added">added</span> This span class is named "added" but you can change it.

<span class="removed">removed</span> This span class is named "removed" but you can change it.

<span class="updated">updated</span> This span class is named "updated" but you can change it.

<span class="fixed">fixed</span> This span class is named "fixed" but you can change it.

<span class="soon">coming soon</span> This span class is named "soon" but you can change it.


Remember to buy:
1. Milk
2. Bread
3. Eggs

Don't forget:
- To review all code
- Write up the changelog
- Make coffee when you're done

To Do:
- [x] Update Homebrew
- [ ] Update all of your gem dependencies
- [x] Drag your feet on updating gem dependencies 

Here are the message boxes with the optional close button enabled:

<p class="box"><span class="closebtn" onclick="this.parentElement.style.display='none';">&times;</span><small><b>Message:</b> This is a sentence inside of a message box.</small></p>
<p class="box-green"><span class="closebtn" onclick="this.parentElement.style.display='none';">&times;</span><small><b>Success!</b> This is a sentence inside of a message box.</small></p>
<p class="box-yellow"><span class="closebtn" onclick="this.parentElement.style.display='none';">&times;</span><small><b>Caution!</b> This is a sentence inside of a message box.</small></p>
<p class="box-orange"><span class="closebtn" onclick="this.parentElement.style.display='none';">&times;</span><small><b>Warning!</b> This is a sentence inside of a message box.</small></p>
<p class="box-red"><span class="closebtn" onclick="this.parentElement.style.display='none';">&times;</span><small><b>Danger!</b> This is a sentence inside of a message box.</small></p>
<p class="box-purple"><span class="closebtn" onclick="this.parentElement.style.display='none';">&times;</span><small><b>Note:</b> This is a sentence inside of a message box.</small></p>
<p class="box-blue"><small><b>Information:</b> This is a sentence inside of a message box.</small></p>

Try to only eat an entire bag of candy once a ~~week~~ month.

If you include any inline `code` it looks awesome.

A code block highlights the [syntax](https://en.wikipedia.org/wiki/Syntax_(programming_languages)) and displays the language:

{% highlight ruby %}
def test(a=1,b=2,c=a+b)
  puts "#{a},#{b},#{c}"
end
test        =>  1,2,3
test 5      =>  5,2,7
test 4, 6   =>  4,6,10
test 3, 4, 6   =>  3,4,6
{% endhighlight %}

{% highlight html %}
<html>
  <body><p>This is HTML!</p></body>
</html>
{% endhighlight %}

```python
def example():
  print("This is Python in fenced code block")
```

***

[^1]: When a footnote is clicked or tapped on it will be highlighted and outlined Wikipedia-style.