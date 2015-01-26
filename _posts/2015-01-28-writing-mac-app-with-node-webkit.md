---
title: "Writing Mac App with nw.js"
date: 2015-01-28 11:00:00
layout: post
categories: javascript, mac
---

Last week I bought a new MacBook Air. In order to make myself feel like a
"power user", I decide to write a simple mac app.


## Why nw.js?

I don't know how to write Objective-C or Swift, and have no idea how to setup Xcode,
so it's better to use some more familiar technologies and nw.js just looks promising.


## Steps

Rather than showing how to write a [fancy][pinegrow] javascript app with nw.js,
I will demonstrate the simple setup & develop flow that I used.

My final app can be found at [here][enigma]. (And I won't tell what is it.)
OK, talk is cheap, just show you the steps:


### 1. Install Homebrew.

One [package manager][homebrew] to rule them all, isn't it?


But don't just use the one-line install script in homebrew's index page,
make sure you understand how it [works][homebrew-installation]
before you install it.


### 2. Install Node.js.

Easy as a pie. But I just found [brewformulas][brwformulas] is very useful.

```bash
$ brew install node
```

### 3. Install your favourite editor with homebrew.

Cool kids use Vim.

```bash
$ brew install mvim
```


### 4. Setup your working directory.

```bash
$ cd /path/to/your/working/dir/
$ npm init
```


### 5. Code like a boss!

But when you coding in javascript, do as the [Pro][jshint] do.


### 6. Add [node-webkit-builder][node-webkit-builder] to your package.

```bash
$ npm install node-webkit-builder --save-dev
```


### 7. Trial run!

```bash
$ ./node_modules/node-webkit-builder/bin/nwbuild -r .
```

But we don't need to re-compile our project every time,
[python-livereload][python-livereload] helps you develop on the fly.


### 8. Feeling good? Publish your awesome app!

```bash
$ ./node_modules/node-webkit-builder/bin/nwbuild -o build .
```

Don't forget to check out the official guide on [packagment and distribution][nw-package].


### 9. Install your app.

Drag and drop it into application folder. But the default icon may seem rigid,
you can follow this [guide][nw-icon] to update it.


### 10. Use it!

![enigma](/public/images/writing-mac-app-with-node-webkit/screenshot.1.png)


[pinegrow]: http://pinegrow.com/
[enigma]: https://github.com/bcho/enigma
[homebrew]: http://brew.sh
[homebrew-installation]: https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/Installation.md#installation
[brewformulas]: http://brewformulas.org/Node
[jshint]: http://jshint.com
[node-webkit-builder]: https://www.npmjs.com/package/node-webkit-builder
[python-livereload]: https://github.com/lepture/python-livereload
[nw-package]: https://github.com/nwjs/nw.js/wiki/How-to-package-and-distribute-your-apps
[nw-icon]: https://github.com/nwjs/nw.js/wiki/Icons
