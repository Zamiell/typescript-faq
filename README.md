# Introduction

I have been heavily using TypeScript since 2019. This is a collectible of my opinions and interpretations of current best-practices in the ecosystem.

<br>

# Should I use `noUnusedLocals`? Should I use `noUnusedParameters`?

- TLDR; no.
- [`noUnusedLocals`](https://www.typescriptlang.org/tsconfig#noUnusedLocals) is a TypeScript compiler option that checks for unused local variables. By default, it is set to false.
- [`noUnusedParameters`](https://www.typescriptlang.org/tsconfig#noUnusedParameters) is a TypeScript compiler option that checks for unused function parameters. By default, it is set to false.
- TypeScript can be configured to be very lax or very strict. By default, it is very lax. My TypeScript philosphy is that you should configure it to be **as strict as possible**. That way, it can catch as many bugs as possible. From this philosphy, it follows that we would want to enable both of these options. And indeed, they are both enabled in the [`@tsconfig/strictest`](https://github.com/tsconfig/bases/blob/main/bases/strictest.json) configuration, which everyone should extend from in their project "tsconfig.json" files.
- If our goal is to catch as many bugs as possible, we should also be running a linter on our TypeScript code in addition to running the TypeScript compiler. [ESLint](https://eslint.org/) is the best JavaScript/TypeScript linter in the world. So, we should be using ESLint along with a configuration that enables [as many good lint rules as possible](https://isaacscript.github.io/eslint-config-isaacscript), including [most of the rules from the `typescript-eslint` project](https://typescript-eslint.io/rules/).
- The `typescript-eslint` project provides a rule called [`@typescript-eslint/no-unused-vars`](https://typescript-eslint.io/rules/no-unused-vars) that finds unused variables. If you have this rule turned on, then you do not need to turn on `noUnusedLocals` or `noUnusedParameters`. Otherwise, you would get double messages for unused variables inside of your IDE instead of just one.
- So, if both `noUnusedLocals`/`noUnusedParameters` and `@typescript-eslint/no-unused-vars` can be used to catch unused variables, which should be used? Which is better?
- The `@typescript-eslint/no-unused-vars` rule is better for several reasons:
  - The [official page for the `@typescript-eslint/no-unused-vars` rule gives a great explanation](https://typescript-eslint.io/rules/no-unused-vars#benefits-over-typescript) as to why it is better than `noUnusedLocals`/`noUnusedParameters`. (Disclaimer: I helped write it!) In short, the lint rule is more customizable and less likely to block builds.
  - `noUnusedLocals`/`noUnusedParameters` were added back in 2016, but they have always been kind of an ad-hoc feature, since finding unused variables does not really have to do with types. It makes sense to let the TypeScript compiler handle the actual type-checking of your code and leave the linting tasks to ESLint. This results in a clear seperation of concerns.
- Thus, if the ESLint rule is better, then we should explicitly disable `noUnusedLocals` or `noUnusedParameters` in our "tsconfig.json" files and then make sure we enable the `@typescript-eslint/no-unused-vars` lint rule. Personally, I enable it like this:

```js
  /**
   * The `args` option is set to `all` make the rule stricter. Additionally, we ignore things that
   * begin with an underscore, since this matches the behavior of the `--noUnusedLocals` TypeScript
   * compiler flag.
   */
  "@typescript-eslint/no-unused-vars": [
    "error",
    {
      args: "all", // "after-used" is the default.
      argsIgnorePattern: "^_",
      varsIgnorePattern: "^_",
    },
  ],
```

- If you don't want to manually enable individual ESLint rules one by one like this, you can use [`eslint-config-isaacscript`](https://isaacscript.github.io/eslint-config-isaacscript), which is a sharable configuration for ESLint that you can use in any TypeScript project.

<br>

# Why should I use Prettier?

- It is extremely common for TypeScript projects to use the [Prettier](https://prettier.io/) code formatter, which makes sure that all the code has the same style.
- Prettier has taken over the TypeScript ecosystem for many reasons. My favorite reason is that it saves an enourmous amount of time, allowing you to code twice as fast.
- Prettier has changed my life and I cannot recommend it enough. For more details, please read [this excellent rant from Brad Zacher](https://github.com/typescript-eslint/typescript-eslint/issues/4907#issuecomment-1118145339). (Brad is a core maintainer of `typescript-eslint`.)

<br>

# Why should I run Prettier on save?

- See the previous question for some introductory details about Prettier.
- It is extremely common for TypeScript projects to use [Visual Studio Code](https://code.visualstudio.com/) (or VSCode, for short) as the code editor or [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment). VSCode is fast, light-weight, customizable, and free, which makes it more popular than other paid-for alternatives (like [WebStorm](https://www.jetbrains.com/webstorm/).)
- ✔️ Many people set the VSCode configuration for their TypeScript projects to automatically run Prettier every time a file is saved. This is considered to be the "best" configuration, since it seamlessly ensures that all code is formatted by Prettier without the developer having to do anything extra over what they would normally do. It also makes it obvious to the developer how the formatting will look in the final upstream repository.
- ❌ Some people do not like formatting on save and instead prefer to explicitly format the code with the Ctrl + Alt + F hotkey (which is the default hotkey for "Format Document" in VSCode). But this pattern has a failure mode: what if you forget to format the file after you save it? Now formatting errors can sneak into the code base. We can run `prettier --check` in CI to guard against this, but this just makes for a worse developer experience, because we want to know about formatting changes immediately, not 15 minutes later after we notice a red X in CI (which causes additional work to be performed).
- ❌ Some people do not like formatting on save and instead prefer the formatting to happen automatically on commit via [a pre-commit hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) (with e.g. [Husky](https://typicode.github.io/husky/)). But this introduces some painful overhead. There is a little overhead in maintaining the pre-commit hook itself (i.e. adding Husky as a development dependency to the project, having the pre-commit hook automatically installed when installing dependencies, and maintaining the pre-commit hook files). But this isn't too bad. The more important problem is that every time a commit is made to the repository (or even a working branch), lag is incurred while the pre-commit hook does its thing. Git commits are best when they contain small, atomic changes. By encumbering the developer with lag every time they make a commit, it discourages small commits and makes for a more painful development experience in general.

<br>

# How do I run Prettier on save? Why isn't Prettier running on save when it is supposed to?

- See the previous question for some introductory details about Prettier and VSCode.
- If you want to officially support VSCode in your TypeScript project, then you can add a `./.vscode/settings.json` file which contains something like the following:

```jsonc
// These are Visual Studio Code settings that should apply to this particular repository.
{
  // Automatically run the formatter when certain files are saved.
  "[javascript][typescript][javascriptreact][typescriptreact]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit",
    },
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.tabSize": 2,
  },
  "[css][json][jsonc][html][markdown][postcss][yaml]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.tabSize": 2,
  },
}
```

(This config also runs `eslint --fix` on save, which is a good idea for the same reason that you want to run Prettier on save.)

- If a project has this configuration, then once anyone clones the repository, installs the npm dependencies (with e.g. `npm install`), and then opens the cloned directory in VSCode, then Prettier should automatically format the file upon saving.
- In this situation, if Prettier is not working automatically, then you should check the following things:
  - Are the dependencies properly installed? In other words, did you run `npm install`? (If a different package manager is used for the project, you might need to run `yarn install` or `pnpm install` instead.) The presence of a "node_modules" direcotry at the root of the project usually indicates that the dependencies for the project has been properly installed (although it might not be there with non-standard package managers).
  - Are you using VSCode to open individual files or are you opening the project directory itself? For VSCode to work properly, you need to select "File" --> "Open Folder" and then point it at the cloned repository.

<br>

# Should I use JavaScript + JSDoc type annotations instead of TypeScript?

The point of TypeScript is that it provides type-safety on top of JavaScript. But it is possible to get most of the safety that TypeScript provides simply by using JSDoc type annotations. (In other words, writing the types as "normal" JavaScript comments.) Notably, the [Svelte](https://github.com/sveltejs/svelte/pull/8569) project does this, and [Rich Harris recommends it for libraries](https://twitter.com/Rich_Harris/status/1350436286948122625) (but not applications). However, most of the ecosystem does not agree with this. As of 2024, no other major projects have done this. So who is right? Let's take a closer look at some of the pros and cons.

## Pro - No Build Step

Far and away, the biggest reason to use JSDoc is to get rid of the build step.

In TypeScript, you write ".ts" files, but the compiler creates ".js" files, and those are what are executed at run-time. Thus, in general, if we could simply have JavaScript files that are directly served to users, that would remove a bunch of complexity!

But is removing the build step as big of a win as it seems?

### 1) Troubleshooting Run-Time Errors

Regardless of whether we are using TypeScript types or JSDoc types, having a fully-typed codebase is going to cut out the vast majority of the run-time errors. But in the real-world, we are sometimes faced with run-time errors, and have to examine the stack trace to try and figure out what is going on.

When we have compiled JavaScript files, the line of the file where the error occurred may not correpond to the same line in the TypeScript source code. This disconnect between the source code and the run-time code can cause some pain. But to be fair, there are some reasons why this is not a big deal.

First, since TypeScript is just JavaScript with types, the compiled code is often extremely similar to the original. For example, if a runtime error happens in a compiled function called "getUserAvatar", it is going to be extremely easy to hone in on the relevant TypeScript code.

Second, [source maps](https://www.typescriptlang.org/tsconfig#sourceMap) exist. When compiling with the TypeScript compiler (or other tools such as [esbuild](https://esbuild.github.io/)), you can turn on source maps, which create ".js.map" files next to the ".js" files. This automatically makes stack traces show the names of the "true" TypeScript functions.

### 2) You Probably Need to Build Your Code Anyway

[If you are writing your JavaScript for Node.js, skip this section. If you are writing your JavaScript for a browser, read on.]

Even in a world where we use JSDoc types instead of TypeScript, we probably need to have a build step anyway. This is mostly for three reasons.

First, we want to convert modern JavaScript to legacy JavaScript for maximum backwards compatibility. Modern JavaScript keeps getting [new features](https://en.wikipedia.org/wiki/ECMAScript_version_history) every year that we probably want to use, like the `class` keyword (from 2015) or the `Array.toSorted` method (from 2023). So, if we are writing a web-application, we probably want to support older browsers. The TypeScript compiler can take care of this automatically, but if we were not using TypeScript, then we would have to use a separate build tool like [Babel](https://babeljs.io/).

Second, if we are serving JavaScript over the web, then we generally want to [minify](https://en.wikipedia.org/wiki/Minification_(programming)) the code. This can reduce the size of the bundle by a lot, making the website feel a lot snappier in places of the world with slower internet. In other words, removing TypeScript to "get rid of the build step" is not really a win when we need to run the JavaScript code through [Terser](https://terser.org/) anyway.

Third, if we are developing a library, then we need to provde TypeScript declaration files (".d.ts" files) for end-users so that they can have a good developer experience. If the library is natively written in TypeScript, then these files are automatically created (as long as the [`declaration`](https://www.typescriptlang.org/tsconfig#declaration) compiler option is set). And if our library is written in JavaScript with JSDoc type-annotations, we can also automatically create these files by pointing the TypeScript compiler at our JavaScript code. But the point is that if we have to run the TypeScript compiler in either case before we publish our code, we are not really gaining much by "skipping" the build step.

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

Additionally, you can convert most TypeScript `interface` and `type` to JSDoc using  `@typedef`. For example:

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

Now let's go over the bad. You [cannot do wildcard imports](https://github.com/microsoft/TypeScript/issues/41825). This in turn makes [dealing with some generics pretty painful](https://github.com/microsoft/TypeScript/issues/41825#issuecomment-1855755546).

With projects of sufficient complexity, the time you spend battling JSDoc-related issues might exceed whatever other penalties you would have incurred by simply writing your code in TypeScript from the get-go.

## Con - Your Code is Ugly

"Uglyness" is subjective, but JSDoc is undeniably more verbose than TypeScript is. Instead of neatly annotating the type directly next to the variable like we do in other languages like Rust or C#, the type is separated by some distance, making it harder to locate at a glance.

Making a codebase more verbose and harder to read is not a cost that should be easily shrugged off.

## Con - Type-Aware Lint Rules Do Not Work

The overall assumption hanging above this discussion is that types are good, because they help us catch bugs at write-time. But if our goal is to catch as many bugs as possible, we should also be running a linter on our code in addition to running the TypeScript type-checker. [ESLint](https://eslint.org/) is the best JavaScript/TypeScript linter in the world. So, we should be using ESLint along with a configuration that enables [as many good lint rules as possible](https://isaacscript.github.io/eslint-config-isaacscript), including [most of the rules from the `typescript-eslint` project](https://typescript-eslint.io/rules/).

However, [around half of the rules](https://typescript-eslint.io/rules/) from the `typescript-eslint` project (and rules from other projects such as [`eslint-plugin-isaacscript`](https://github.com/IsaacScript/isaacscript/tree/main/packages/eslint-plugin-isaacscript)) use type information in order to work properly. And this "type information" is not currently extracted from JSDoc comments. Meaning that tons of ESLint rules that would catch bugs on a TypeScript codebase are not going to catch those bugs in a JavaScript codebase with JSDoc type annotations!

Brad Zacher from the core team explains why this is:

> [Adding JSDoc support] comes at a high maintenance cost because we need to manually build out all support. i.e. We need to figure out predictable comment attachment that matches how TypeScript does it and then we need to parse and extract the relevant information from the comments. It's also not included in the AST so we can't traverse it with the standard tooling - meaning we'd need to manually traverse and extract things.
>
> It's very expensive to build and maintain and it's costly at runtime - JSDoc parsing is slow.
>
> On the other hand, with type annotations we need to do nothing extra - we just analyse the AST and that's it. Zero runtime cost and its low maintenance.
>
> I fully get why people want this and I think there is a small portion of the ecosystem which would benefit - but the massive cost of building, maintaining, and running it makes it a non-starter. The cost just far outweighs the small benefit.

That is a big coverage gap and is probably the biggest con to using JSDoc, especially for people who are used to having ESLint around as a helpful friend who always has their back.

## Conclusion

- Pro - No Build Step
- Neutral - Same Dependencies + Same Maintenance of Config Files
- Con - Non-Perfect Type Support
- Con - Your Code is Ugly
- Con - Type-Aware Lint Rules Do Not Work

For tiny projects (like one-off scripts), using JSDoc types makes a lot of sense. It satisfies the [Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle) (i.e. the 80-20 rule). For these kinds of projects, you probably do not need to run `tsc` as a type-checker in CI, and you probably do not need a full-blown ESLint setup.

But as soon as your project grows larger than a single file, you should probably start to think about converting your code to TypeScript. Troubleshooting bugs is not fun: any time that you would spend troubleshooting bugs that would be automatically caught by ESLint has to be weighed against the upfront, one-time cost of setting up TypeScript + ESlint.

Part of the problem here is that setting up TypeScript + ESlint can be annoying. One possible solution is to simply bootstrap all of your new projects, big or small, through a TypeScript + ESLint project bootstrapper such as [`create-typescript-app`](https://github.com/JoshuaKGoldberg/create-typescript-app) or my personal tool [`isaacscript init-ts`](https://isaacscript.github.io/main/isaacscript-in-typescript).
