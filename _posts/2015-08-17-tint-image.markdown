---
layout: post
title: "Image tint, gradient overlay, and some text on top of it"
date: 2015-08-17
categories: coding
crossposted: svbtle
---

A visual solution which is employed broadly around the web is to take an image, darken it, then lay a little more darkening gradient, and then put some white text on top of it. Here's a snippet that does exactly this thing.

At first, I got an `img` and had to use extra elements to tint the image:

```html
<!doctype html>
<html>
    <head>
        <style>
            body {
                margin: 0;
                padding: 0;
                font-size: 100%;
                font-family: sans-serif;
            }

            .parent {
                position: relative;
                /* have to explicitly define relative element's measures */
                height: 600px;
                width: 1000px;
            }

            .parent img {
                position: absolute;
            }

            .sibling {
                position: absolute;
                height: 200px;
                width: 100%;
                bottom: 0px;
                right: 0px;
                /* after an hour of experimenting */
                background-image: linear-gradient(
                    to bottom,
                    rgba(0, 0, 0, 0) 0%,
                    /* must resemble inner element's padding-top */
                    rgba(0, 0, 0, 0.15) 1em, 
                    rgba(0, 0, 0, 0.5) 30%,
                    rgba(0, 0, 0, 1) 100%);
            }

            .sibling span {
                display: block;
                width: auto;
                /* must resemble parent element's second gradient point */
                padding: 1em 3em;
                color: rgba(255, 255, 255, 0.8);
                font-family: "Helvetica Neue", sans-serif;
                font-size: 2em;
                font-style: italic;
            }
        </style>
    </head>
    <body>
        <div class="parent">
            <img src="http://lorempixel.com/1000/600/" alt="" />
            <div class="sibling">
                <span>Lorem ipsum dolor sit amet and so on</span>
            </div>
        </div>
    </body>
</html>
```

[The solution](http://jsfiddle.net/taxigy/p0upkvyb/) works pretty well, however, it takes extra DOM elements to achieve. And I love lean DOM, the leaner the better. So another option is to put an image onto element's background. The snippet:

```html
<!doctype html>
<html>
    <head>
        <style>
            html,
            body {
                margin: 0;
                padding: 0;
                font-size: 100%;
                width: 100%;
                height: 100%;
                font-family: sans-serif;
                z-index: 100;
            }

            .parent {
                display: block;
                position: relative;
                width: 100%;
                height: 100%;
                background: url(http://lorempixel.com/1000/600/);
                background-repeat: no-repeat;
                background-position: center center;
                background-size: cover;
                z-index: 0;
            }

            .parent::before {
                content: " ";
                display: block;
                position: absolute;
                bottom: 0;
                left: 0;
                width: 100%;
                height: 50%;
                background: -webkit-linear-gradient(top,
                    rgba(255,255,255,0) 0%,
                    rgba(0,0,0,0) 1%,
                    rgba(0,0,0,0.05) 25%,
                    rgba(0,0,0,0.5) 75%,
                    rgba(0,0,0,0.7) 100%);
                z-index: 2;
            }

            .parent::after {
                content: " ";
                display: block;
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background: rgba(0,0,0,0.5);
                z-index: 3;
            }
        </style>
    </head>
    <body>
        <div class="parent">
        </div>
    </body>
</html>
```

[Works well, too](http://jsfiddle.net/taxigy/qa1z2cu6/). The next thing to do is to put text over it, not a big deal.

That's it. Clean and simple.