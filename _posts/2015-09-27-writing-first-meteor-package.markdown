---
layout: post
title: "Writing the first Meteor package"
date: 2015-09-27
categories: coding
---

The material on how to create your own [Meteor](http://meteor.com) package is abundant (first stop out there should always be [Atmosphere](https://atmospherejs.com/i/publishing)). It's easy to find an answer to almost any question, and even in case it's not as easy, StackOverflow should be at a shortcut all the time. But there's hardly a single tutorial for *complete newbs,* ones as I am to Meteor, so I compiled this little post to make sure there exists one.

## What is a Meteor package?

I love simple answers, those which ignite interest and awaken people and direct their search. So the answer is now easy as well.

A Meteor package is a set of files which, after the package has been installed into particular app, are virtually put into `client` or `server` or both folders so that variables, classes, methods and other things are available to the app's code.

So, what should have been written somewhere in bold is that it's like you copy package's files content and paste it respectively into client or server sections, or both. That easy.

A package can contain whatever its developer wants: Javascript sources, HTML templates, JSON objects, Mongo collection definitions, Markdown files, movies, anything. The developer has to explicitly define whether a particular file (i.e. its content) should be pushed by Meteor engine to client, to server, to both or nowhere. And some docs would be nice, but it's optional, of course.

## What a package should be about?

Anything repeatable and universal enough to have it used then in another app in the foreseeable future. A package usable by other developers would be even nicer to have, though.

Just take a look at amazing [`aldeed:autoform`](https://github.com/aldeed/meteor-autoform) or [`mizzao:tutorials`](https://github.com/mizzao/meteor-tutorials). It's insane how much effort their authors and contributors have put into them. However, not every package should be a top-level multitool installed 100k times. One should always start small.

## Where to start?

Start with a single line in terminal,

```bash
$ meteor create --package your-name:package-name # replace your-name with your Meteor username and package-name with the name of a package you're about to create
```

There will be `package.js`, the file that is invoked by Meteor every time the package is added to an app. This file controls everything about the package, it is package's meta-controller. What it does is it defined what files go where and what packages should this package rely on, in sort of declarative style. It is pretty easy to understand, especially with [official documentation](http://docs.meteor.com/#/full/packagejs).

## So, what is my cool package for?

I was working on a project that employed Meteor as the primary platform and required integration with PayLane, a European payment processing service. Despite the whole processing experience, from the app's experience, is just two redirects and a form between them, I decided to make a stand-alone package and develop it separately, simply to train the skill and see if anybody else finds it useful.

So, the workflow was quite simple, in just 5 (sometimes not as easy) steps:

1. Create a boilerplate for the future package using `meteor create --package rishatmuhametshin:paylane`.
2. Tumble and ask a lot of questions on StackOverflow.
3. Test it locally by constantly copying contents of the package to `packages/paylane` folder of the app and constantly alternating between `meteor add rishatmuhametshin:paylane` and `meteor remove rishatmuhametshin:paylane`.
4. Get it working, publish it according to [the guide](https://atmospherejs.com/i/publishing), and then publish some more version because prior ones were awful.
5. Clean app's `packages` folder and install the package from Atmosphere.

It's easy to track all the problems I stumbled on during the development of version `0.0.1` in [the repo](https://github.com/taxigy/meteor-paylane/commits/master). So I'll expand some notes made from observations.

## Challenges and solutions

First thing, I had a problem with getting my package working in the app. After installation, the success message appeared, but the installation process just didn't yield proper output, since `.meteor/packages` didn't contain any code from the package. The problem was that I didn't copy the package contents into app's `packages/paylane` folder but used symbolic link instead. It was worth me the whole day to understand that this was exactly the root of the problem. The process has been captured in [a StackOverflow post](http://stackoverflow.com/questions/32759052/error-unknown-package-in-top-level-dependencies-in-meteor-app).

I had an HTML template in the package:

```html
<template name="checkoutPaylane">
    ...
</template>
```

Since all the files are virtually put into the app as is, I expected the template to appear but it just didn't. It turned out that the package developer should explicitly define in the package settings that the latter rely on `templating` package:

```javascript
// RIGHT
Package.onUse(function(api) {
    api.versionsFrom('1.1.0.3');
    // ...
    api.use([
        'templating',
        // ...
    ], 'client');
    // ...
});
```

Including `templating` on client solved the issue. But then there was another one. The app threw an error message in client's console that `Template.checkoutPaylane.helpers` can't be accessed because it's a property of `undefined`. That was kind of weird but the solution was easy, too: the order of package loading matters, and template definition should go before template helpers and events definitions, hence HTMLs at the top of `api.addFiles`:

```javascript
// WRONG
api.addFiles([
    'paylane-client.js'
    'paylane-form.html',
], 'client');

// RIGHT
api.addFiles([
    'paylane-form.html',
    'paylane-client.js'
], 'client');
```

## The formula

So, I'll put out the formula to create a small and simple package, yet a working one. There are few easy steps:

1. Pretend the package to be just the code within the app. It will even help if you first implement it within the app itself, and only start moving it to a separate package afterwards.
1. First implement client-side things. Don't forget about `templating` package and the order of exported files!
1. Copy the package files into `packages/the-name-of-your-package` folder of your app every time you want to test the new commit, no symlinking! But make sure you do `meteor remove` and `meteor add` as well in case you stick to a single version during the development process, otherwise `meteor update your-name:your-package` just won't work.
1. Once the client-side code works, go for server-side. Make sure you use global variables (i.e. not defined with `var`), and set up ESlint or JSHint so that they're okay with that ([`no-undef: 0` rule](http://eslint.org/docs/rules/no-undef.html) for ESLint).
1. Keep things as simple as possible. Simple and working is better than a more dead than alive multitool.

Once there's a working thing, just publish it. A simple `README.md` will suffice. No need to make it perfect. Even if it has a lot of bugs, it's unlikely the package will be installed by somebody else into their stable app in production. The risk is low.

## An extra idea

As a part of one of deep searches during package development, I found great advice on software: [a pull request to an existing repo is better than forking and making one's own copy](https://github.com/arunoda/meteor-up/issues/451#issuecomment-114232657). So true!