---
layout: post
title: 'Preloading libaries in your node repl'
description: >
  Have your oft-used libraries in the node repl pre-initialized.
permalink: '/posts/preloading-libraries-in-node-repl.html'
---

After grumbling countless times thinking 'why isn't lodash preloaded as soon as start my node repl?!', I bit the bullet and quickly hacked up a script to do just that. After perusing the [repl](https://nodejs.org/api/repl.html#repl_repl_features) doc, this is what I came up with.

<script src="https://gist.github.com/jithugopal/f386be73e7a22ad7f1ee.js"></script>
<noscript>
  <pre>
\#!/usr/local/bin/node
var libs = ['lodash'];
var repl = require('repl').start('> ');
var npmPath = repl.context.process.env['NPM_HOME'];
libs.forEach(function(lib) {
  // requiring libs from npm root
  repl.context[lib] = require([npmPath, 'lib/node_modules', lib].join('/'));
});
  </pre>
</noscript>


Ok fine, I should have mentioned that your packages should be installed globally in the first place. The npm install switch is `-g` in case you forgot.

```
$ npm install -g lodash
```

You need to have `NPM_HOME` set in your `PATH`. Also, it would have been neat if I could have got a `npm root -g` equivalent to get the `npm_modules` path instead of all the string munging.

To put the script in my `PATH`, I created an `nrepl` file in `/usr/local/bin` (I can see my clojure friends smirking). Of course, you got to change permissions too silly.

```
$ chmod u+x nrepl
$ mv nrepl /usr/local/bin
```

And to see if everything works.

```
$ nrepl
> lodash.each
[Function]
```

Yay!
