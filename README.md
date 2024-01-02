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

# Should I use [`tsx`](https://github.com/privatenumber/tsx) or [`ts-node`](https://github.com/TypeStrong/ts-node)?

Both `tsx` and `ts-node` are programs that allow you to run TypeScript files directly without compiling them to JavaScript first. This is incredibly useful for testing out code in development, running project scripts, and more. But which of the two is better?

`tsx` has out-of-the-box support for ESM and tsconfig.json paths, so it will "just work" in many situations where `ts-node` will require additional configuration and/or not work at all. Furthermore, `tsx` is more actively developed/maintained as of the time of this writing (January 2024). Thus, I recommend always using `tsx` and never using `ts-node`.

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
- ❌ Some people do not like formatting on save and instead prefer the formatting to happen automatically on commit via [a pre-commit hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) (with e.g. [Husky](https://typicode.github.io/husky/)). But this introduces [some painful overhead](https://www.youtube.com/watch?v=RAelLqnnOp0). There is a little overhead in maintaining the pre-commit hook itself (i.e. adding Husky as a development dependency to the project, making sure that [people explicitly add the pre-commit hooks after cloning the repository](https://blog.typicode.com/posts/husky-git-hooks-autoinstall/), and maintaining the pre-commit hook files). The overhead is annoying, but is not too bad. The more important problem is that every time a commit is made to the repository (or even a working branch), lag is incurred while the pre-commit hook does its thing. Git commits are best when they contain small, atomic changes. By encumbering the developer with lag every time they make a commit, it discourages small commits and makes for a more painful development experience in general.

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

Now let's go over the bad:

- You [cannot do wildcard imports](https://github.com/microsoft/TypeScript/issues/41825).
- There are problems with [importing generic types](https://github.com/microsoft/TypeScript/issues/41825#issuecomment-1855755546).

With projects of sufficient complexity, the time you spend battling JSDoc-related issues might exceed whatever other penalties you would have incurred by simply writing your code in TypeScript from the get-go.

## Con - Your Code is Ugly

"Uglyness" is subjective, but JSDoc is undeniably more verbose than TypeScript is. Instead of neatly annotating the type directly next to the variable like we do in other languages like Rust or C#, the type is separated by some distance, making it harder to locate at a glance.

Making a codebase more verbose and harder to read is not a cost that should be easily shrugged off.

## Con - Type-Aware Lint Rules Do Not Work

The overall assumption hanging above this discussion is that types are good, because they help us catch bugs at write-time. But if our goal is to catch as many bugs as possible, we should also be running a linter on our code in addition to running the TypeScript type-checker. [ESLint](https://eslint.org/) is the best JavaScript/TypeScript linter in the world. So, we should be using ESLint along with a configuration that enables [as many good lint rules as possible](https://isaacscript.github.io/eslint-config-isaacscript), including [most of the rules from the `typescript-eslint` project](https://typescript-eslint.io/rules/).

However, [around half of the rules](https://typescript-eslint.io/rules/) from the `typescript-eslint` project (and rules from other projects such as [`eslint-plugin-isaacscript`](https://github.com/IsaacScript/isaacscript/tree/main/packages/eslint-plugin-isaacscript)) use type information in order to work properly. And this "type information" is not currently extracted from JSDoc comments. Meaning that tons of ESLint rules that would catch bugs on a TypeScript codebase are not going to work with JavaScript JSDoc type annotations!

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

Part of the problem here is that setting up TypeScript + ESLint by hand can be time-intensive. One possible solution is to simply bootstrap all of your new projects, big or small, through a TypeScript + ESLint project bootstrapper such as [`create-typescript-app`](https://github.com/JoshuaKGoldberg/create-typescript-app) or my personal tool [`isaacscript init-ts`](https://isaacscript.github.io/main/isaacscript-in-typescript).

<br>

# Should I use `interface` or `type`?

In TypeScript, you can use both `interface` and `type` to model a basic object:

```ts
interface Foo {
  bar: number;
  baz: string;
}

type Foo = {
  bar: number;
  baz: string;
};
```

So which should you use? Which is better? First, let's go over the differences.

## The Differences Between `interface` and `type`

### 1) `extends` - ✔️ `interface` wins (but it is debatable)

- `interface` can use `extends`, which allows you to create types without repeating yourself. (i.e. `interface B extends A {}`)
- `type` cannot use `extends`, but it can use `&` to intersect, which is similar. (i.e. `type B = {} & A`)
- However, using `&` to create an intersection is not exactly the same as extends. [According to Daniel Rosenwasser](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections) (the TypeScript program manager): "Interfaces create a single flat object type that detects property conflicts, which are usually important to resolve! Intersections on the other hand just recursively merge properties, and in some cases produce never."
- In other words, interfaces check for incompatible properties, producing a TypeScript compiler error, which is good. But intersection types may or may not produce a downstream error, which is bad!
  - Of course, there are some edge-cases when we deliberately want to create a `never` type, or when [using `interface` is awkward](https://www.typescriptlang.org/play?#code/C4TwDgpgBAggjFAvFA3gWAFBW1MAnAezAC5UoBDUgZ2DwEsA7AcygF9N2NNRIoAhBMnhQAZKkw5chEmQBGpBgFcAtrIh42HTJgAmEAMYAbcnmj6CDGlFlxSAzLIB0+ItoyNg6gGbl90GABM4lg4LjIoFNS0jCycnJge3r7QfEEQAB6eDDpUsEHoIdhhpIEA2gDkYeUAuqJyCipqGnFuekYmZhZW8vwBDs7SmEA). So we can't use `extends` in _every_ situation. But in the general case, it is probably better to have more robust error handling.
- Other concerns about `extends` vs intersection types generalize to the greater discussion of `interface` vs `type`, so we also need to take the other differences below into account.

### 2) Declaration Merging - ✔️ `interface` wins (but it is debatable)

- `interface` can use declaration merging, which allows you to add new fields to an existing declared interface. For example, this is useful for augmenting `globals.Window`.
- `type` can not be declaration merged.
- Some people argue that declaration merging is actually dangerous, which makes it an anti-feature and automatically makes `interface` lose. More on that later on though.

### 3) Opaque Naming - ✔️ `interface` wins (but it is debatable)

- `interface` creates a concrete, unique, named type.
- `type` creates a type alias, which means that TypeScript sometimes forgets the name and just refers to the type as its anonymous object shape.
- In general, the ecosystem seems to agree that opaque naming is superior, but we discuss that later on in more detail.

### 4) Unions - ❌ `type` wins

- `interface` is for declaring the shape of objects.
- `type` is for computing types based on other information.
- It is idiomatic in TypeScript to represent an object as a [discriminated union](https://basarat.gitbook.io/typescript/type-system/discriminated-unions). Thus, if you have a type that is a composition of other types, you cannot use `interface` and must use `type`.

### 5) Primitives - ❌ `type` wins

- If you want to create a type that is based off of a primitive type (like `number`), then you cannot use `interface` and must use `type`.
- This kind of thing is common when branding primitives for better type-safety. For example:

```ts
export type UserID = number & { readonly __userIDBrand: unique symbol };
```

### 6) Performance - Nobody wins

- [Matt Pocock](https://www.youtube.com/@mattpocockuk) mentions that [the TypeScript team says that there should be no performance difference between the two as of 2023](https://www.youtube.com/watch?v=zM9UPcIyyhQ). (He does not show any receipts for the claim, but it appears plausible.)

## Arguments

Now that we have a firm grasp of the concrete differences between `interface` and `type`, we can start to look at the arguments for using one over the other.

### Argument: Use `type` Since It Is More Generic

[Web Dev Simplified](https://www.youtube.com/c/WebDevSimplified) puts forth the argument that [we should use `type` over `interface` because it can make our codebases more consistent](https://www.youtube.com/watch?v=jJGzYdS4ZfY). The argument goes like this:

- Most types can be represented by both `interface` and `type`.
- However, overall, many more types can be represented by `type` than by `interface` (e.g. unions, primitives).
- If we always use `type` for as much stuff as we can, then we can mostly purge `interface` from our codebase entirely (except in the special case where we need to use `extends` or declaration merging).
- Thus, by having almost everything be `type`, it is easier to read the codebase, because everything is consistently declared as a `type`.

This argument has some merit, but it does not strike me as being very convincing:

- `extends` in particular is an extremely useful feature; many codebases will have interfaces that use `extends`.
- Thus, even though we are trying to purge `interface` from our codebase, we might end up having a mix of both `type` and `interface` anyway.

### Argument: Use `type` Because Accidental Declaration Merging Sucks

- As mentioned above, declaration merging is a feature that `interface` has but `type` does not. Subsequently, you might be tempted to immediately mark this as a win for `interface`. But not so fast.
- Some people argue that declaration merging is dangerous in a similar way to having global variables in your program is dangerous. Thus, declaration merging is an anti-feature, and having it exist actually makes `interface` bad. Because declaration merging exists, you should try to use `type` over `interface` whenever possible so that you can avoid shooting yourself in the foot.
- But how much of a footgun is declaration merging really? It turns out that as long as you use ESM, all things are module scoped - meaning that there is no chance of accidental cross-file declaration merging from name clashes. When using ESM, [you cannot even accidentally merge with a global type](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgHIHsAmKDeBYAKGWQBt0E4SAFKdABwC5kBnMKUAcwG5CBfQwtgQk4UFAnQhWySKyYZsPArLAA6MhWq06SlaoCCAFUMAlAJIAhAKqGAogH1UAeQAitpYQgAPOuihhkHF4uZAB6UOQJAFsoiHAZAAtgZmR0AFcAuBBMRJQ4ACN0ADcUaFooZExkuDo6CFFmQiA)!
- In other words, the footgun from declaration merging only really applies in very specific cases, like when writing `.d.ts` files with other interfaces in the global scope. This is not something that we need to do in modern applications that are written in TypeScript from the get-go.

### Argument: Use `type` Because Declaration Merging Allows Mutability

- Most TypeScript programmers would agree that you should always use `const` instead of `let`, if possible. Programs are much easier to reason about without having to keep track of variable mutations. `let` is the root cause of many bugs!
- This same concept applies to exported types. Even if an `interface` is properly exported, and both sides are using ESM, it is still possible for a consumer to mutate a library's type like this:

```ts
declare module "my-library" {
  interface Foo {
    sneakyExtraProperty: string;
  }
}
```

- It is not possible to mutate an exported `type`. In other words, `interface` is like `let`, and `type` is like `const`.
- So, if you writing a library, you might feel compelled to keep track of any `interface` that you export, in order to code defensively around the fact that some user might extend them. But that's extra work for not much other gain. So this is a pretty good reason to prefer `type` over `interface`! This is covered in [Matt Pocock's video on `type` versus `interface`]](https://www.youtube.com/watch?v=zM9UPcIyyhQ).
- But let's step back for a moment. Just because it is _technically possible_ to mutate an existing `interface`, does not mean that it is something worth worrying about.
- The previous analogy to `let` is a bit flawed in that it is trivial to mutate a `let` variable. But we have to deliberately go out of our way to declaration merge, such that it would be virtually impossible to do it by accident. In other words, the danger of immutability scales proportionally with how easy it is to mutate.
- To illustrate this point, I think a good analogy is [the intended TypeScript escape hatch](https://github.com/microsoft/TypeScript/issues/19335) for accessing private fields. TypeScript rightly disallows accessing class fields marked as `private` from outside the class, as you would expect. But the linked issue showcases that TypeScript actually allows access to private fields when using index notation. This is very surprising for people who have not seen it before! (But the escape hatch is very useful, as it allows classes to be better tested.)
- One could argue: "Since it is technically possible for people who consume your library's class to use the escape hatch to access your private variables, you should carefully code your class with safeguards that mitigate the damage they could do." One could go even farther: "You should not use `private` fields at all since they are technically unsafe". If you are like me, this probably strikes you as a weird argument: if someone is deliberately mutating your private variables, all bets are off and the warranty on the class is void.
- In other words, even though `private` variables are not technically private, we should treat them as such. And subsequently, even though `interface` is not technically constant, we should also treat them as such. Because when someone mutates private variables or mutates exported interfaces, they are breaking the implicit contract of the library, and this should not be our problem.
- The conlusion here is that `type` is unambiguously safer, but the safety does not matter that much, and the safety might not be more important than the other advantages that `interface` has.

### Argument: Use `type` Because `interface` Can Explicitly Denote Declaration Merging

- First, see the argument in the previous section.
- Some library authors write interfaces that are intended to be declaration merged by the consumers. These special interfaces are usually denoted with a JSDoc comment.
- But if a library exports some interfaces that should be declaration merged and some that should not, that's confusing. Why are we delineating them with a JSDoc comment if we could instead delineate them with official TypeScript keywords? The obvious advantage of the latter is that the immutability is enforced by the language itself!
- This pattern makes a lot of sense. But notice that the base assumption here is that "sometimes, we want to declaration merge, and other times we don't". Is that assumption true?
- In libraries that are natively written in TypeScript, there is probably no need for the consumers to use declaration merging at all. It is safer and more understandable for consumers if the mutation is passed to the library explicitly - either as either the input to a function or the input to a generic type. Then, the modified type is passed back to the consumer as an output.
- In conclusion, in a world where we ignore that declaration merging exists, we don't need to use `interface` to explicitly denote it.

### Argument: Use `interface` Because Names Are Awesome

- In general, we want our types to be named, since it provides a better developer experience. For example, consider the following code that uses the [Zod validator library](https://zod.dev/):

```ts
import { z } from "zod";

const user = z.object({
  userID: z.number(),
  username: z.string(),
  counters: z.number(),
})

type User = z.infer<typeof user>;

function getUser(): User | undefined {
  // TODO
}

const someUser = getUser();
```

- When we mouse over `someUser` to examine the type, we see the following type:

```ts
{
    userID: number;
    username: string;
    counters: number;
} | undefined
```
 
- This is quite verbose and hard to parse. Even though we manually specified the return type of the `getUser` function to be `User | undefined`, TypeScript still spits out the uglified type back at us. What we really want is the mouseover to show `User | undefined`.
- This can be accomplished by using an interface, since an interface is always a named type:

```ts
- type User = z.infer<typeof user>;
+ interface User extends z.infer<typeof user> {}
```

- So in general, we want to prefer `interface` over `type` since it provides named types in all circumstances!
- In fact, this is the reason that the TypeScript ecosystem as a whole has converged on `interface` over `type` (more on that later). Let's do a quick history lesson. Consider the following code:

```ts
interface InterfaceUser {
    userID: number;
    username: string;
    counters: number;
}

declare const interfaceUser: InterfaceUser;

type TypeUser = {
    userID: number;
    username: string;
    counters: number;
}

declare const typeUser: TypeUser;
```

- In this example, mousing over `interfaceUser` would always show the type correctly as `InterfaceUser`. But historically, mousing over the `typeUser` variable would not work properly! (It used to show the uglified type.) But in 2024, both of these now display correctly, because TypeScript has made improvements to `type`. But as we saw from the Zod example earlier on in this section, `type` does not _always_ work properly. So even though `type` has been improved, it still is not on par with interfaces.

### Argument: Use `interface` Because The Ecosystem Has Already Chosen `interface`

- We previously explored some practical reasons why one would want to prefer `interface` over `type` or vice versa. I think the practical arguments sway more towards using `interface`. But to be completely honest, the practical reasons are not super powerful for one side or the other.
- For this reason, whether to choose `interface` or `type` might mostly just come down to a matter of style. But when choosing the style for your TypeScript code, it makes a lot of sense to use the conventions that already prevail in the ecosystem. The idea is that we want our TypeScript code to look like everyone else's TypeScript code, since it makes it much easier for other people to read and understand. A great man once said that [code is read more often than it is written](https://skeptics.stackexchange.com/questions/48560/is-code-read-more-often-than-its-written).
- So what is the prevailing style in the ecosystem? The answer is `interface`. As mentioned in the previous section, this is mostly a historical artifact of `type` having some buggy behavior.
- This convention is codified inside of the [`@typescript-eslint/consistent-type-definitions`](https://typescript-eslint.io/rules/consistent-type-definitions/) ESLint rule. The rule ensures a consistent style for type definitions throughout a codebase, and the default option is `interface`.
  - If you are not already using most of the rules in `typescript-eslint`, you definately should! The aforementioned rule is contained within the [`stylistic`](https://typescript-eslint.io/linting/configs/#stylistic) config, so that is the easiest way to turn it on. Alternatively, the rule is also included in the comprehensive [`eslint-config-isaacscript`](https://isaacscript.github.io/eslint-config-isaacscript).

## Conclusion

- You will have to use both `interface` and `type` no matter what.
- When you have a choice, you should use `interface` for practical reasons:
  - You don't have to be scared of declaration merging in most cases.
  - The unambiguous names provide a better development experience (for e.g. mouseover types and TypeScript error messages).
- If you care about following ecosystem standards and conventions, then using `interface` is a no-brainer.

<br>
