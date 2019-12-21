---
title: "Copy Code to Clipboard"
date: 2019-12-17T16:09:33-06:00
draft: false
---

{{< simple_quotes quote="Imitation is not just the sincerest form of flattery - it's the sincerest form of learning." cite="George Bernard Shaw" >}}   

### Add a "Copy to Clipboard" button in Hugo

One of the benefits of using open source software is the ability to learn from others. I have used that to my advantage over my career and want to make sure to pay it forward (in any small way I can). 
<!--more-->
So as a new user to [Hugo](https://gohugo.io/) I wanted to add the "Copy-to-Clipboard" button to my posts which have code.

{{< img src="example_copy_to_clipboard_w_highlights.png" >}}

Lucky for me someone has already figured this out. [Danny Guo](https://www.dannyguo.com/) has a great post: [How to Add Copy to Clipboard Buttons to Code Blocks in Hugo](https://www.dannyguo.com/blog/how-to-add-copy-to-clipboard-buttons-to-code-blocks-in-hugo/).

The funny part however...is that I am so new to Hugo, I didn't know exactly what to do with his code. So this post is a concise summary of what to do with Danny's code.

### Javascript Copy Clipboard
The core of the logic is found in the javascript file: [copy-code-button.js](https://github.com/dguo/dannyguo.com/blob/master/static/js/copy-code-button.js)

Place this file in your Hugo Static folder: <BLOG>/static/js

{{< img src="copy-code-button-image.png" >}}

### CSS
Next we need to add the CSS code for the correct layout and view of the buttons: [post.css](https://github.com/dguo/dannyguo.com/blob/master/static/css/post.css)

Place this file in your Hugo Static folder: <BLOG>/static/css

{{< img src="post-css-image.png" >}}

### How to Use
So now that we have the javascript and css in place we still need to call the code for a given page. 

#### Option 1: Partials (TBD)
OK...I don't know how to do this one yet. I believe this is what Danny is using, but like I said I can't figure it out. I can search for the javascript in Danny's website files: [search for copy-code.js usage](https://github.com/dguo/dannyguo.com/search?q=copy-code-button.js&unscoped_q=copy-code-button.js)

I know that the code is getting called in a file: 

layouts/blog/single.html
```html
{{ define "js-extra" }}
    {{ if (findRE "<pre" .Content 1) }}
        <script src="/js/copy-code-button.js"></script>
    {{ end  }}
{{ end  }}
```
However I find the logic of Hugo Themes/Partials/Templates very confusing. So at this point this option is a dead end (for me). Others will undoubtedly understand the steps to take, but I am not there yet. If anyone stumbles across this post and wants to comment...feel free to reach out to me.

#### Option 2: Shortcode (This one works!)
Even though I couldn't figure out the whole Template/Partial framework I found the use of a [Shortcode](https://gohugo.io/content-management/shortcodes/) to be fairly straightforward. 

Create a new shortcode by adding the following code:
```html
<!DOCTYPE html>
<html lang="en">
<script src="/jb_blog/js/copy-code-button.js"></script>
</html>
```
to a new file *code-clipboard.html* in the layouts/shortcodes folder:

{{< img src="code-clipboard-folder-image.png" >}}

Then in any markdown file that you wish to have the button enabled you simply  add the following code to the bottom of that markdown file:
```html
{{</* code-clipboard */>}}
```
Then when the page is created through the Hugo publish process you will see the "copy" button enabled.

### Resources:

- [Danny Guo's Blog](https://www.dannyguo.com/)
- [Danny Guo's Github](https://github.com/dguo) 

{{< code-clipboard >}}
