---
layout: post
title: "Grunt boilerplate"
date: 2015-08-22
categories: coding
---

Step by step, I've been getting into Grunt, after using Gulp for quite long (which is counter-traditional story, though). Here, in this post I collect some recipes for Grunt, mostly because other references are too complicated for such a newb like me.

## Just convert styles

The good thing is that you don't need a whole Node project for such a simple task as converting SASS files to browser-compatible CSS.

The setup:

```bash
$ npm install grunt grunt-contrib-sass --save-dev
$ npm install grunt-contrib-watch --save
```

The config:

```javascript
module.exports = function (grunt) {
    grunt.initConfig({
        sass: {
            all: {
                files: [{
                    expand: true,
                    cwd: 'src/sass/',
                    src: ['*.scss'],
                    dest: 'dist/css/',
                    ext: '.css'
                }]
            }
        },
        watch: {
            sass: {
                files: ['html/sass/**/*.scss'],
                tasks: ['sass']
            }
        }
    });

    grunt.loadNpmTasks('grunt-contrib-sass');
    grunt.loadNpmTasks('grunt-contrib-watch');
};
```

Then simply run

```bash
$ grunt watch
```

and start editing. Every `.scss` file within `src` folder will be picked up and converted, all dependencies (imports) resolved.