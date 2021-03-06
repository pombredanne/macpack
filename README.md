# Overview

Bundles up all dependencies used by an executable (or library) into a single
folder, and patches all of the executables to look in that directory so you
don't have to rely on your users having certain libraries installed.

Part of my motivation in making this was to learn Python 3's
[async/await](https://docs.python.org/3/library/asyncio-task.html), too. The
processes that are used to patch stuff and query dependencies all run in
parallel so it should be super fast!

# Usage and requirements

You need:

* Python 3 (`brew install python3`) because it uses async/await and asyncio
* Xcode CLI tools (I think)

The easiest way to get going is by installing through pip + PyPi:

```
pip3 install macpack
macpack <your executable here>
```

You can also run it out of the project directory by doing:

```
macpack/patcher.py <your executable here>
```

It should print the dependency tree like this example:

```
$ macpack ~/Code/node-canvas/build/Release/canvas.node
Patching /Users/caleb/Code/node-canvas/build/Release/canvas.node
16 total non-system dependencies
1       libpixman-1.0.dylib -> 1
2       libcairo.2.dylib -> 2, 1, 16, 9, 3
3       libpng16.16.dylib -> 3
4       libpangocairo-1.0.0.dylib -> 4, 5, 2, 14, 6, 13, 7, 8, 15, 16, 9
5       libpango-1.0.0.dylib -> 5, 6, 13, 7, 8
6       libgobject-2.0.0.dylib -> 6, 7, 11, 12, 8
7       libglib-2.0.0.dylib -> 7, 11, 8
8       libintl.8.dylib -> 8
9       libfreetype.6.dylib -> 9, 3
10      libjpeg.8.dylib -> 10
11      libpcre.1.dylib -> 11
12      libffi.6.dylib -> 12
13      libgthread-2.0.0.dylib -> 13, 7, 11, 8
14      libpangoft2-1.0.0.dylib -> 14, 5, 6, 13, 7, 8, 15, 16, 9
15      libharfbuzz.0.dylib -> 15, 7, 8, 9
16      libfontconfig.1.dylib -> 16, 9

canvas.node + 16 dependencies successfully patched
```

Everything that your executable uses should then be copied into the same folder
that your binary is. When your main binary is run next, it will look in the new
location you specified (default is `binary_dir/libs/<lib>`, see `-d` below).
Those dylibs will look in the same directory for dylibs they depend on, too, even
if your main binary does not use them.

You can then distribute the whole folder as one.

# Options

## -v, --verbose

Pass `-v` to get output from `otool` if it failed to patch or more information on 
which dependencies could not be loaded.

It will also print a more easy to read dependency tree, with the full names of
dependenies under each one

## -d, --destination

This is the destination folder **relative to the binary's containing folder** to
copy library dependencies to. For example, if you binary is `/a/b/program`,
and you pass `-d ../libraries`, they will copy to and load from
`/a/libraries/`. The default value is `../libs`.

If you want the executable and libraries to have absolute paths instead of loading
relative to the binary, you just need to specify an absolutep path for `-d`. In
that case the `@executable_path` will not be put into the binaries at all.

## -n, --dry-run

Just prints the dependency tree and doesn't do any patching. Use `-nv` to get a
slightly more user-friendly tree printed out.

# Background
It will parse out the executable's dependencies (using `otool -L`) and their
dependencies recursively, filtering out system libraries. When the tree
is built, it will copy the libraries to your program's folder and then patch
everything that it is aware of (using `install_name_tool`). It should be able
to handle different symbolic links and all that correctly

# Credits
Inspired by [macdylibbundler](https://github.com/auriamg/macdylibbundler), it does
the same basic thing except with less options (at the moment) and it builds a full
dependency tree

