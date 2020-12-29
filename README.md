# cpc-dev

Designed to be used as a starting point for developing complex programs for
CraftOS-PC, this repository contains everything you need to transpile TypeScript
to Lua, including the typings (as a submodule).

Great editor support, solid types and autocomplete make it a breeze to hammer
out long complex programs that perform without a hitch. It's no substitute for
skill, but it makes it so much easier to use that skill without having to
memorize all the function signatures, or keep a reference open 24/7.

## Getting Started

Clone the repository recursively:

    git clone --recursive https://github.com/LoganDark/cpc-dev

If you did not clone it recursively, run this command to update the submodules:

    git submodule update --init

Next, run this command to download the Node modules you need (typings, TS, and
the transpiler):

    npm install

Then, use your favorite text editor or IDE to edit the source files inside the
`src` folder. Once you're ready to transpile to Lua, run this command:

    npm run build

This will run the transpiler and generate two files: `build/main.lua`, and
`build/main.lua.map`. In almost all cases, you won't need the second file. For
the curious, [source maps](https://en.wikipedia.org/wiki/Minification_(programming)#Source_mapping)
simply describe which parts of your TS code ended up as which parts of the
generated Lua code. They do not contain code themselves.

You can run your program within CraftOS-PC by simply mounting the file like so:

    /path/to/craftos --mount-ro /main.lua=$HOME/cpc-dev/build/main.lua

Substitute the executable path and mount point appropriately.

## Caveats

TypeScript and Lua differ greatly. For one, arrays in Lua are one-indexed, where
in TypeScript they are zero-indexed. This means the transpiler will attempt to
translate one to the other when generating Lua code. This works most of the
time, but when working with array-like types that are not instances of `Array`,
the transform will not be applied, which can lead to confusing occurrences of
`undefined` (or `nil` in Lua-land).

You should definitely have deep knowledge of Lua and how your TypeScript code
will translate before attempting a serious project using this base.

## Tips

- Most common functions like `string.substring` are polyfilled, but some aren't.
  The most notable thing that isn't polyfilled is `process.argv`, which does NOT
  transpile to `{...}`. The officially endorsed solution is this:
  
      /** @vararg */
      interface Vararg<T> extends Array<T> {}
      declare const _args: Vararg<string>
      const args = _args
  
  I don't personally like how hacky it feels, and I feel like it could stop
  working at any time, but it'll probably be fine.

- CraftOS-PC's debugger is broken until 2.5.1. If breakpoints aren't working for
  you, that could be why. You can compile CPC from source like I have, or wait
  for 2.5.1 to release.

- You're still going to have to use coroutines. You should be okay, because
  `craftos.d.ts` includes typings for (soon) all Lua APIs included in CPC,
  including the coroutine library. However, the debug library is not typed yet,
  as it is hidden behind a config flag. (also bit32 is not typed yet)

- Since TypeScript functions have an implicit `this`, TSTL will always add an
  explicit `self` unless you say otherwise. Sometimes this is good, like in the
  case of classes, but it's not really needed when writing other functions,
  especially functions you expect someone from the Lua side to call into.
  
  To remove this explicit `self`, you should write your function declarations
  like this:
  
      function doSomething(this: void, arg1: number, ...) {
          // stuff...
      }
  
  It's a type-safe way to declare that the function does not have a `this`, and
  TSTL will pick up on it and not add a `self`.

- In order to use some Lua features from TS, you might need to use annotations.
  For example, if your function needs to return multiple values to the Lua side
  (mandatory if you're creating a redirect object for example), you will need an
  `@tupleReturn` annotation. Check out the [TypeScriptToLua documentation](
  https://typescripttolua.github.io/docs/advanced/compiler-annotations) for more
  information about the different annotations you can use.

- Maybe more? Submit a pull request or even just an issue :)

## Getting Help

Most likely, the people in the ComputerCraft Discord won't know much about
transpilation from TypeScript to Lua. Since this repository does not contain
anything custom, you should be able to look up help pertaining to
TypeScriptToLua specifically, and that should apply fine here (since that is the
transpiler we're using, after all).

GitHub Discussions is also enabled on this repository if you need more help.
