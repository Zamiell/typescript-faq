# Should I use JavaScript + JSDoc type annotations instead of TypeScript?

The point of TypeScript is that it provides type-safety on top of JavaScript. But it is possible to get most of the safety that TypeScript provides simply by using JSDoc type annotations. (In other words, writing the types as "normal" JavaScript comments.) Notably, the [Svelte](https://github.com/sveltejs/svelte/pull/8569) project does this, and [Rich Harris recommends it for libraries](https://twitter.com/Rich_Harris/status/1350436286948122625) (but not applications). However, most of the ecosystem does not agree with this. As of 2024, no other major projects have done this. So who is right? Let's take a closer look at some of the pros and cons.

## Pro - No Build Step

Far and away, the biggest reason to use JSDoc is to get rid of the build step.

In TypeScript, you write ".ts" files, but the compiler creates ".js" files, and those are what are executed at run-time. Thus, in general, if we could simply have JavaScript files that are directly served to users, that would remove a bunch of complexity!

But is removing the build step as big of a win as it seems?

### 1) Troubleshooting Run-Time Errors

Regardless of whether we are using TypeScript types or JSDoc types, having a fully-typed codebase is going to cut out the vast majority of the run-time errors. But in the real-world, we are sometimes faced with run-time errors, and have to examine the stack trace to try and figure out what is going on.

When we have compiled JavaScript files, the line of the file where the error occurred may not correspond to the same line in the TypeScript source code. This disconnect between the source code and the run-time code can cause some pain. But to be fair, there are some reasons why this is not a big deal.

First, since TypeScript is just JavaScript with types, the compiled code is often extremely similar to the original. For example, if a runtime error happens in a compiled function called "getUserAvatar", it is going to be extremely easy to hone in on the relevant TypeScript code.

Second, [source maps](https://www.typescriptlang.org/tsconfig#sourceMap) exist. When compiling with the TypeScript compiler (or other tools such as [esbuild](https://esbuild.github.io/)), you can turn on source maps, which create ".js.map" files next to the ".js" files. This automatically makes stack traces show the names of the "true" TypeScript functions.

### 2) You Probably Need to Build Your Code Anyway

[If you are writing your JavaScript for Node.js, skip this section. If you are writing your JavaScript for a browser, read on.]

Even in a world where we use JSDoc types instead of TypeScript, we probably need to have a build step anyway. This is mostly for three reasons.

First, we want to convert modern JavaScript to legacy JavaScript for maximum backwards compatibility. Modern JavaScript keeps getting [new features](https://en.wikipedia.org/wiki/ECMAScript_version_history) every year that we probably want to use, like the `class` keyword (from 2015) or the `Array.toSorted` method (from 2023). So, if we are writing a web-application, we probably want to support older browsers. The TypeScript compiler can take care of this automatically, but if we were not using TypeScript, then we would have to use a separate build tool like [Babel](https://babeljs.io/).

Second, if we are serving JavaScript over the web, then we generally want to [minify](<https://en.wikipedia.org/wiki/Minification_(programming)>) the code. This can reduce the size of the bundle by a lot, making the website feel a lot snappier in places of the world with slower internet. In other words, removing TypeScript to "get rid of the build step" is not really a win when we need to run the JavaScript code through [Terser](https://terser.org/) anyway.

Third, if we are developing a library, then we need to provide TypeScript declaration files (".d.ts" files) for end-users so that they can have a good developer experience. If the library is natively written in TypeScript, then these files are automatically created (as long as the [`declaration`](https://www.typescriptlang.org/tsconfig#declaration) compiler option is set). And if our library is written in JavaScript with JSDoc type-annotations, we can also automatically create these files by pointing the TypeScript compiler at our JavaScript code. But the point is that if we have to run the TypeScript compiler in either case before we publish our code, we are not really gaining much by "skipping" the build step.

### 3) [`tsx`](https://github.com/privatenumber/tsx) Exists

[If you are writing your JavaScript for a browser, skip this section. If you are writing your JavaScript for Node.js, read on.]

Node.js cannot natively run TypeScript, so for Node.js projects, we have historically needed to convert the TypeScript to JavaScript. But in 2024 this is not relevant anymore because [`tsx`](https://github.com/privatenumber/tsx) allows us to seamlessly run TypeScript code. It is as easy as typing `npx tsx foo.ts`. There are also other solutions to run TypeScript directly, including [Deno](https://deno.com/) and [Bun](https://bun.sh/).

### 4) Building is Basically Instantaneous

Most people use the official TypeScript compiler for the task of converting ".ts" files to ".js" files. ([K.I.S.S.](https://en.wikipedia.org/wiki/KISS_principle)) In general, this works great, and small projects will compile within a second or two.

However, for larger projects, the compilation time can start to get annoying. The root of the issue is twofold. First, the `tsc` tool is doing both compilation (i.e. generating ".js" files) and type-checking (i.e. checking for errors) at the same time. Second, the TypeScript compiler is itself written in TypeScript, which is not the most performant language.

Thankfully, in 2024 we have a solution for this. Large projects can use tools like [esbuild](https://esbuild.github.io/) (written in Golang) or [swc](https://swc.rs/) (written in Rust) to only compile the code (and skip type-checking it). Both of these perform orders of magnitude faster than the official TypeScript compiler.

Thus, "TypeScript compilation is slow" is not a reason anymore to not use TypeScript, and this is not a motivating factor to want to remove the build step more generally.

### 5) Declaration Maps

A lot of libraries that are built with TypeScript will only ship the JavaScript source code, because that's the code that is actually needed in order to make the library work. However, if a consumer imports a function from the library and then presses F12 on the function (for i.e. "Go To Definition"), the IDE would warp to the compiled, messy JavaScript implementation, instead of the "real" TypeScript source code.

But there is a solution. If you are shipping a library and care about this kind of thing, you can simply turn on the [`declarationMap`](https://www.typescriptlang.org/tsconfig#declarationMap) compiler option and then make sure to include the `src` folder inside the published npm package. Thus, this is not considered a win for removing the build step.

## Neutral - Same Dependencies + Same Maintenance of Config Files

Unfortunately, `tsc` (the TypeScript command-line tool) does not have very good defaults. If you want to make it as strict and as safe as possible, you need to set up a configuration file that extends from [`@tsconfig/strictest`](https://github.com/tsconfig/bases?tab=readme-ov-file#strictest-tsconfigjson) as well as do some other minor tweaks.

But note that even if we "remove" TypeScript from our codebase, we will still need to type-check our JSDoc-annotated code in CI to make sure that no bugs slip in to the codebase. Thus, we still need to run the TypeScript type-checker, and this requires a configuration file that has the [`allowJs`](https://www.typescriptlang.org/tsconfig#allowJs) option and so forth.

So, we would still need to have "typescript" as a `devDependency` in the project, and we would still have to maintain "tsconfig.json" files and CI type-checking jobs either way.

## Con - Non-Perfect Type Support

JSDoc functionality is pretty good, but it is not perfect.

Let's start with the good. A basic TypeScript function like this:

```ts
function foo(arg1: string, arg2: number) {}
```

Would be equivalent to:

```js
/**
 * @param {string} arg1
 * @returns {number} arg2
 */
function foo(arg1, arg2) {}
```

Additionally, you can convert most TypeScript `interface` and `type` to JSDoc using `@typedef`. For example:

```ts
interface Foo {
  someField: string;
  someOtherField: number;
}
```

Would be equivalent to:

```js
/**
 * @typedef {{
 *  someField: string;
 *  someOtherField: number;
 * }} Foo
 */
```

And this:

```ts
type Bar = Exclude<Foo, "someOtherField">;
```

Would be equivalent to:

```js
/** @typedef {Exclude<Foo, 'someOtherField'>} Bar */
```

JSDoc also has [some support for generics](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html). You can even put JSDoc types in dedicated ".js" files and then import them elsewhere in the codebase.

Now let's go over the bad:

- You [cannot do wildcard imports](https://github.com/microsoft/TypeScript/issues/41825).
- There are problems with [importing generic types](https://github.com/microsoft/TypeScript/issues/41825#issuecomment-1855755546).

With projects of sufficient complexity, the time you spend battling JSDoc-related issues might exceed whatever other penalties you would have incurred by simply writing your code in TypeScript from the get-go.

## Con - Your Code is Ugly

The beauty of code is subjective, but JSDoc is undeniably more verbose than TypeScript is. Instead of neatly annotating the type directly next to the variable like we do in other languages like Rust or C#, the type is separated by some distance, making it harder to locate at a glance.

Making a codebase more verbose and harder to read is not a cost that should be easily shrugged off.

## Con - Some Lint Rules Do Not Work

Rules that rely on TypeScript-specific keywords will not work with their JSDoc counterparts, such as [`@typescript-eslint/explicit-member-accessibility`](https://typescript-eslint.io/rules/explicit-member-accessibility/) and [`@typescript-eslint/explicit-function-return-type`](https://typescript-eslint.io/rules/explicit-function-return-type/).

Brad Zacher from the core team explains why this is:

> [Adding JSDoc support] comes at a high maintenance cost because we need to manually build out all support. i.e. We need to figure out predictable comment attachment that matches how TypeScript does it and then we need to parse and extract the relevant information from the comments. It's also not included in the AST so we can't traverse it with the standard tooling - meaning we'd need to manually traverse and extract things.
>
> It's very expensive to build and maintain and it's costly at runtime - JSDoc parsing is slow.
>
> On the other hand, with type annotations we need to do nothing extra - we just analyze the AST and that's it. Zero runtime cost and its low maintenance.
>
> I fully get why people want this and I think there is a small portion of the ecosystem which would benefit - but the massive cost of building, maintaining, and running it makes it a non-starter. The cost just far outweighs the small benefit.

## Conclusion

- Pro - No Build Step
- Neutral - Same Dependencies + Same Maintenance of Config Files
- Con - Non-Perfect Type Support
- Con - Your Code is Ugly
- Con - Some Lint Rules Do Not Work

For tiny projects (like one-off scripts), using JSDoc types makes a lot of sense. It satisfies the [Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle) (i.e. the 80-20 rule). For these kinds of projects, you probably do not need to run `tsc` as a type-checker in CI, and you probably do not need a full-blown ESLint setup.

But as soon as your project grows larger than a single file, you should probably start to think about converting your code to TypeScript.

Part of the problem here is that setting up TypeScript + ESLint by hand can be time-intensive. One possible solution is to simply bootstrap all of your new projects, big or small, through a TypeScript + ESLint project bootstrapper such as [`create-typescript-app`](https://github.com/JoshuaKGoldberg/create-typescript-app) or my personal tool [`isaacscript init-ts`](https://isaacscript.github.io/main/isaacscript-in-typescript).
