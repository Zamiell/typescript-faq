# TypeScript FAQ

<!-- markdownlint-disable MD033 -->

## Introduction

I have been heavily using [TypeScript](https://www.typescriptlang.org/) since 2019. This is a collectible of my opinions and interpretations of current best-practices in the ecosystem.

<br>

## Should I use `noUnusedLocals`? Should I use `noUnusedParameters`?

- tldr; no.
- [`noUnusedLocals`](https://www.typescriptlang.org/tsconfig#noUnusedLocals) is a TypeScript compiler option that checks for unused local variables. By default, it is set to false.
- [`noUnusedParameters`](https://www.typescriptlang.org/tsconfig#noUnusedParameters) is a TypeScript compiler option that checks for unused function parameters. By default, it is set to false.
- TypeScript can be configured to be very lax or very strict. By default, it is very lax. My TypeScript philosophy is that you should configure it to be **as strict as possible**. That way, it can catch as many bugs as possible. From this philosophy, it follows that we would want to enable both of these options. And indeed, they are both enabled in the [`@tsconfig/strictest`](https://github.com/tsconfig/bases/blob/main/bases/strictest.json) configuration, which everyone should extend from in their project "tsconfig.json" files.
- If our goal is to catch as many bugs as possible, we should also be running a linter on our TypeScript code in addition to running the TypeScript compiler. [ESLint](https://eslint.org/) is the best JavaScript/TypeScript linter in the world. So, we should be using ESLint along with a configuration that enables [as many good lint rules as possible](https://isaacscript.github.io/eslint-config-isaacscript), including [most of the rules from the `typescript-eslint` project](https://typescript-eslint.io/rules/).
- The `typescript-eslint` project provides a rule called [`@typescript-eslint/no-unused-vars`](https://typescript-eslint.io/rules/no-unused-vars) that finds unused variables. If you have this rule turned on, then you do not need to turn on `noUnusedLocals` or `noUnusedParameters`. Otherwise, you would get double messages for unused variables inside of your IDE instead of just one.
- So, if both `noUnusedLocals`/`noUnusedParameters` and `@typescript-eslint/no-unused-vars` can be used to catch unused variables, which should be used? Which is better?
- The `@typescript-eslint/no-unused-vars` rule is better for several reasons:
  - The [official page for the `@typescript-eslint/no-unused-vars` rule gives a great explanation](https://typescript-eslint.io/rules/no-unused-vars#benefits-over-typescript) as to why it is better than `noUnusedLocals`/`noUnusedParameters`. (Disclaimer: I helped write it!) In short, the lint rule is more customizable and less likely to block builds.
  - `noUnusedLocals`/`noUnusedParameters` were added back in 2016, but they have always been kind of an ad-hoc feature, since finding unused variables does not really have to do with types. It makes sense to let the TypeScript compiler handle the actual type-checking of your code and leave the linting tasks to ESLint. This results in a clear separation of concerns.
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

## Should I use [`tsx`](https://github.com/privatenumber/tsx) or [`ts-node`](https://github.com/TypeStrong/ts-node)?

Both `tsx` and `ts-node` are programs that allow you to run TypeScript files directly without compiling them to JavaScript first. This is incredibly useful for testing out code in development, running project scripts, and more. But which of the two is better?

`tsx` has out-of-the-box support for ESM and tsconfig.json paths, so it will "just work" in many situations where `ts-node` will require additional configuration and/or not work at all. Furthermore, `tsx` is more actively developed/maintained as of the time of this writing (January 2024). Thus, I recommend always using `tsx` and never using `ts-node`.

<br>

## Why should I use Prettier?

- It is extremely common for TypeScript projects to use the [Prettier](https://prettier.io/) code formatter, which makes sure that all the code has the same style.
- Prettier has taken over the TypeScript ecosystem for many reasons. My favorite reason is that it saves an enormous amount of time, allowing you to code twice as fast.
- Prettier has changed my life and I cannot recommend it enough. For more details, please read [this excellent rant from Brad Zacher](https://github.com/typescript-eslint/typescript-eslint/issues/4907#issuecomment-1118145339). (Brad is a core maintainer of `typescript-eslint`.)

<br>

## Why should I run Prettier on save?

- See the previous question for some introductory details about Prettier.
- It is extremely common for TypeScript projects to use [Visual Studio Code](https://code.visualstudio.com/) (or VSCode, for short) as the code editor or [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment). VSCode is fast, light-weight, customizable, and free, which makes it more popular than other paid-for alternatives (like [WebStorm](https://www.jetbrains.com/webstorm/).)
- ✔️ Many people set the VSCode configuration for their TypeScript projects to automatically run Prettier every time a file is saved. This is considered to be the "best" configuration, since it seamlessly ensures that all code is formatted by Prettier without the developer having to do anything extra over what they would normally do. It also makes it obvious to the developer how the formatting will look in the final upstream repository.
- ❌ Some people do not like formatting on save and instead prefer to explicitly format the code with the Ctrl + Alt + F hotkey (which is the default hotkey for "Format Document" in VSCode). But this pattern has a failure mode: what if you forget to format the file after you save it? Now formatting errors can sneak into the code base. We can run `prettier --check` in CI to guard against this, but this just makes for a worse developer experience, because we want to know about formatting changes immediately, not 15 minutes later after we notice a red X in CI (which causes additional work to be performed).
- ❌ Some people do not like formatting on save and instead prefer the formatting to happen automatically on commit via [a pre-commit hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) (with e.g. [Husky](https://typicode.github.io/husky/)). But this introduces [some painful overhead](https://www.youtube.com/watch?v=RAelLqnnOp0). There is a little overhead in maintaining the pre-commit hook itself (i.e. adding Husky as a development dependency to the project, making sure that [people explicitly add the pre-commit hooks after cloning the repository](https://blog.typicode.com/posts/husky-git-hooks-autoinstall/), and maintaining the pre-commit hook files). The overhead is annoying, but is not too bad. The more important problem is that every time a commit is made to the repository (or even a working branch), lag is incurred while the pre-commit hook does its thing. Git commits are best when they contain small, atomic changes. By encumbering the developer with lag every time they make a commit, it discourages small commits and makes for a more painful development experience in general.

<br>

## How do I run Prettier on save? Why isn't Prettier running on save when it is supposed to?

- See the previous question for some introductory details about Prettier and VSCode.
- If you want to officially support VSCode in your TypeScript project, then you can add a `./.vscode/settings.json` file which contains something like the following:

```jsonc
// These are Visual Studio Code settings that should apply to this particular repository.
{
  // Automatically run the formatter when certain files are saved.
  "[javascript][typescript][javascriptreact][typescriptreact]": {
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    },
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.tabSize": 2
  },
  "[css][json][jsonc][html][markdown][postcss][yaml]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.tabSize": 2
  }
}
```

(This config also runs `eslint --fix` on save, which is a good idea for the same reason that you want to run Prettier on save.)

- If a project has this configuration, then once anyone clones the repository, installs the npm dependencies (with e.g. `npm install`), and then opens the cloned directory in VSCode, then Prettier should automatically format the file upon saving.
- In this situation, if Prettier is not working automatically, then you should check the following things:
  - Are the dependencies properly installed? In other words, did you run `npm install`? (If a different package manager is used for the project, you might need to run `yarn install` or `pnpm install` instead.) The presence of a "node_modules" directory at the root of the project usually indicates that the dependencies for the project has been properly installed (although it might not be there with non-standard package managers).
  - Are you using VSCode to open individual files or are you opening the project directory itself? For VSCode to work properly, you need to select "File" --> "Open Folder" and then point it at the cloned repository.

<br>

## Should I use JavaScript + JSDoc type annotations instead of TypeScript?

See [this page](./questions/jsdoc-vs-typescript.md).

<br>

## Should I use `interface` or `type`?

See [this page](./questions/interface-vs-type.md).

<br>

## Should I use TypeScript enums?

Many TypeScript programmers [do like like using enums](https://www.youtube.com/watch?v=jjMbPt_H3RQ) for a variety of reasons:

- Enums are not a native JavaScript feature and they are compiled to an object with an arbitrary format, which breaks the typical TypeScript contract of only "adding types".
- Enums are both a type and a container at the same time, which can be confusing.
- Enum values are nominal/branded, which can be confusing (since TypeScript is normally structurally typed).
- Number enums automatically generate a reverse mapping, while string enums do not, which is confusing.
- The `const enum` variant can be confusing since it makes the behavior different from the other types of enums. There is [an entire section in the TypeScript docs](https://www.typescriptlang.org/docs/handbook/enums.html#const-enum-pitfalls) that goes over the pitfalls of `const enum`. (However, arguments against `const enum` do not necessarily apply to normal enums.)

The anti-enum folk suggest that you use "normal" objects instead of enums, like this:

```ts
export type ObjectValues<T> = T[keyof T];

export const FRUIT = {
  Apple: "Apple",
  Banana: "Banana",
} as const;

export type Fruit = ObjectValues<typeof FRUIT>;
```

This breaks up the enum into the run-time container (i.e. `FRUIT`) and the type (i.e. `Fruit`). However, there is a very dangerous downfall with this approach, which is that the values are no longer branded. In other words, we lose type-safety:

```ts
export type ObjectValues<T> = T[keyof T];

const FOO = {
  Value1: 1,
} as const;

export type Foo = ObjectValues<typeof FOO>;

const BAR = {
  Value1: 1,
} as const;

export type Bar = ObjectValues<typeof BAR>;

function useFoo(foo: Foo) {}

useFoo(Bar.Value1); // BUG! But the TypeScript compiler does not care.
```

There is also another major reason to use enums: code clarity. An object declaration can mean many different kind of things, but an `enum` declaration means exactly one thing. Thus, it is extremely clear what the original programmer intends and it is extremely clear what the code does. Enums are also usually very concise and easy to read.

For these reasons, I recommend using TypeScript's official (string) enums over manually-built creations that duplicate the same functionality. I believe that the benefits outweigh the other potentially confusing functionality. But you should always make sure to use the [`@typescript-eslint/no-unsafe-enum-comparison`](https://typescript-eslint.io/rules/no-unsafe-enum-comparison/) lint rule, which makes working with them safe.

<br>

## Should I use number enums or string enums?

In general, you should use string enums, but using number enums is okay in certain situations. See [this explanation](https://github.com/IsaacScript/isaacscript/blob/main/packages/eslint-plugin-isaacscript/docs/rules/strict-enums.md#number-enums-vs-string-enums).

<br>
