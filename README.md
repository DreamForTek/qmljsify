# Qmljsify
Convert an NPM package into a QML friendly JavaScript file

It is still a prototype software. Use it at your own risk.

**Proven Working Packages**

    fecha (Lightweight version of moment)
    lodash(--no-minify)
    sprintf

Installation Instruction
=================

A pre-built version of qmljsify is available in Docker Hub. You may pull the docker image directly:

```
docker pull benlau/qmljsify
docker tag benlau/qmljsify qmljsify
alias qmljsify='docker run --rm -v $PWD:/data --user $(id -u):$(id -g) -i -t qmljsify'
```

Usage
=====

```
Usage: qmljsify [options] command package
qmljsify - Download and convert an NPM package to a QML friendly JavaScript file

Options:
  -h, --help   Displays this help.
  --no-minify  Turn off JavaScript Compression

Arguments:
  command      Command [Available Commands: convert]
  package      NPM package to be qmljsified
```


Example:

```
  qmljsify convert sprintf
```

Then it will fetch `sprintf` from NPM and create two files

```
  sprintf.orig.js # A compiled and minified sprintf library
  sprintf.js # A Wrapper of the compiled sprintf library for QML
```

That is what `sprintf.js` looks like:

```
.pragma library
Qt.include("sprintf.orig.js")
var object_stringify = QML.object_stringify;
var format = QML.format;
var cache = QML.cache;
var parse = QML.parse;
var sprintf = QML.sprintf;
var vsprintf = QML.vsprintf;
```

Then you could use it in your QML file:

```
import "./sprintf.js" as SPrintf

// [snipped]
SPrintf.sprintf("%d %d", 1 , 2);
```

Known Issues
============

1. It doesn't work on Windows.

2. setTimeout is not wrapped.

3. It may not works for some npm package.

Remarks: NPM library with only a single function is supported now (e.g. left-pad)

Build Instruction
=================

Prerequisite:

- webpack < 4.0
- npm
- nodejs

```
  cd app/qmljsify
  qpm install
  qmake
  make
  #Then copy qmljsify to your favor path
```

p.s Windows is not working yet.

Build Instruction (Docker)
==========================


```
cd qmljsify/docker
./build-docker-images.sh
alias qmljsify='docker run --rm -v $PWD:/data --user $(id -u):$(id -g) -i -t qmljsify'
```


TroubleShooting
===============

1) qml: SyntaxError: Expected token `;'

```
  $ qmljsify convert lodash
  qml: SyntaxError: Expected token `;'
```
Try --no-minify

```
  $ qmljsify convert --no-minify lodash
  lodash.orig.js saved
  lodash.js saved
```

2) SyntaxError: Expected token `identifier`

The Javascript loaded by `Qt.include` could not use "as" as a variable name. The minified Javascript may contain such kind of variable so it will raise the exception. You may try to use the `--no-minify` argument to create a non-minified QML friendly Javascript.

Brainstorming
------------

Proposed features:

1. --combine - Merge orig js and wrapper js into a single file. (It could be a solution of the bug of Qt.include)

2. Handle library with only a single function

3. --function name - Set the function name of the library that only provides a single function

4. Break down the "convert" function into multiple steps.

5. --retry-minify - Try to minify for few more times unless it doesn't use any variable name with `as'
