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
- In other words, interfaces check for incompatible properties, producing a TypeScript compiler error, which is good. But intersection types may or may not produce an error, which is bad!
  - Of course, there are some edge-cases when we deliberately want to create a `never` type, or when [using `interface` is awkward](https://www.typescriptlang.org/play?#code/C4TwDgpgBAggjFAvFA3gWAFBW1MAnAezAC5UoBDUgZ2DwEsA7AcygF9N2NNRIoAhBMnhQAZKkw5chEmQBGpBgFcAtrIh42HTJgAmEAMYAbcnmj6CDGlFlxSAzLIB0+ItoyNg6gGbl90GABM4lg4LjIoFNS0jCycnJge3r7QfEEQAB6eDDpUsEHoIdhhpIEA2gDkYeUAuqJyCipqGnFuekYmZhZW8vwBDs7SmEA). So we can't use `extends` in _every_ situation. But in the general case, it is probably better to have more robust error handling.
- Other concerns about `extends` vs intersection types generalize to the greater discussion of `interface` vs `type`, so we also need to take the other differences below into account.

### 2) Declaration Merging - Nobody wins (but it is debatable)

- `interface` can use declaration merging, which allows you to add new fields to an existing declared interface. For example, this is useful for augmenting `globals.Window`.
- `type` can not be declaration merged.
- On the one hand, a feature that `interface` has and `type` does not have should make `interface` win.
- On the other hand, declaration merging is considered to be confusing and dangerous. Some people advocate using `type` just so that you can avoid it altogether.
- While I think that avoiding `interface` altogether just because declaration merging exists is a little too extreme (more on that later), we can probably say that there is no clear winner here.

### 3) Unions - ❌ `type` wins

- `interface` is for declaring the shape of objects.
- `type` is for computing types based on other information.
- It is idiomatic in TypeScript to represent an object as a [discriminated union](https://basarat.gitbook.io/typescript/type-system/discriminated-unions). Thus, if you have a type that is a composition of other types, you cannot use `interface` and must use `type`.

### 4) Primitives - ❌ `type` wins

- If you want to create a type that is based off of a primitive type (like `number`), then you cannot use `interface` and must use `type`.
- This kind of thing is common when branding primitives for better type-safety. For example:

```ts
export type UserID = number & { readonly __userIDBrand: unique symbol };
```

### 5) Opaque Naming - Nobody wins

- Historically, `type` was bugged such that TypeScript would sometimes forget the name and just refer to the type as its anonymous object shape. However, in 2024, these bugs have been fixed. I am not aware of any current situations in which `type` will "bleed through" like it used to. (If you are, please submit a pull request!)
- Opaque naming is extremely useful in order to not be blasted with information when inspecting complex types. But since both `interface` and `type` should always result in a named type, then this historic win for `interface` no longer applies.
  - If a type uses [`Expand`](https://stackoverflow.com/a/57683652/1062714), it can still bleed through, but such a thing would be done intentionally.

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

- `extends` in particular is a useful feature for its flattening functionality. Many codebases will have interfaces that use `extends`.
- Thus, even though we are trying to purge `interface` from our codebase, we might end up having a mix of both `type` and `interface` anyway.

### Argument: Use `type` Because Accidental Declaration Merging Sucks

- Some people argue that declaration merging is dangerous in a similar way to having global variables in your program is dangerous. Thus, declaration merging is an anti-feature, and having it exist actually makes `interface` bad. Because declaration merging exists, you should try to use `type` over `interface` whenever possible so that you can avoid shooting yourself in the foot.
- But how much of a footgun is declaration merging really? It turns out that as long as you use ESM, all things are module scoped - meaning that there is no chance of accidental cross-file declaration merging from name clashes. When using ESM, [you cannot even accidentally merge with a global type](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgHIHsAmKDeBYAKGWQBt0E4SAFKdABwC5kBnMKUAcwG5CBfQwtgQk4UFAnQhWySKyYZsPArLAA6MhWq06SlaoCCAFUMAlAJIAhAKqGAogH1UAeQAitpYQgAPOuihhkHF4uZAB6UOQJAFsoiHAZAAtgZmR0AFcAuBBMRJQ4ACN0ADcUaFooZExkuDo6CFFmQiA)!
- In other words, the footgun from declaration merging only applies in very specific cases, like when writing `.d.ts` files with other interfaces in the global scope. This is not something that we need to do in modern applications that are written in TypeScript from the get-go.

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
- So, if you writing a library, you might feel compelled to keep track of any `interface` that you export, in order to code defensively around the fact that some user might extend them. But that's extra work for not much other gain. So this is a pretty good reason to prefer `type` over `interface`! (This is covered in [Matt Pocock's video on `type` versus `interface`](https://www.youtube.com/watch?v=zM9UPcIyyhQ).)
- But let's step back for a moment. Just because it is _technically possible_ to mutate an existing `interface`, does not mean that it is something worth worrying about.
- The previous analogy to `let` is a bit flawed in that it is trivial to mutate a `let` variable. But we have to deliberately go out of our way to declaration merge, such that it would be virtually impossible to do it by accident. In other words, the danger of immutability scales proportionally with how easy it is to mutate.
- To illustrate this point, I think a good analogy is [the intended TypeScript escape hatch](https://github.com/microsoft/TypeScript/issues/19335) for accessing private class fields. TypeScript rightly disallows accessing class fields marked as `private` from outside the class, as you would expect. But the linked issue showcases that TypeScript actually allows access to private fields when using index notation. This is very surprising for people who have not seen it before! (But the escape hatch is very useful, as it allows classes to be easily tested.)
- One could argue: "Since it is technically possible for people who consume your library's class to use the escape hatch to access your private variables, you should carefully code your class with safeguards that mitigate the damage they could do." One could go even farther: "You should not use `private` fields at all since they are technically unsafe". If you are like me, this probably strikes you as a weird argument: if someone is deliberately mutating your private variables, all bets are off and the warranty on the class is void.
- In other words, even though `private` variables are not technically private, we should treat them as such. And subsequently, even though `interface` is not technically constant, we should also treat them as such. Because when someone mutates private variables or mutates exported interfaces, they are breaking the implicit contract of the library, and this should not be our problem.
- The conclusion here is that `type` is unambiguously safer, but the safety does not matter that much.

### Argument: Use `type` Because `interface` Can Explicitly Denote Declaration Merging

- First, see the argument in the previous section, which is similar.
- Some library authors write interfaces that are intended to be declaration merged by the consumers. These special interfaces are usually denoted with a JSDoc comment.
- But if a library exports some interfaces that should be declaration merged and some that should not, that's confusing. Why are we delineating them with a JSDoc comment if we could instead delineate them with official TypeScript keywords? The obvious advantage of the latter is that the immutability would be enforced by the language itself!
- This pattern makes a lot of sense. But notice that the base assumption here is that "sometimes, we want to declaration merge, and other times we don't". Is that assumption true?
- In libraries that are natively written in TypeScript, there is probably no need for the consumers to use declaration merging at all. It is safer and more understandable for consumers if the mutation is passed to the library explicitly - either as either the input to a function or the input to a generic type. Then, the modified type can be passed back to the consumer as an output.
- In conclusion, this pattern makes sense for existing libraries that already have declaration merging. But in a world where we are writing new code, we can ignore that declaration merging exists, and we subsequently do not need to use `interface` to explicitly denote it.

### Argument: Use `interface` Because The Ecosystem Has Already Chosen `interface`

- We previously explored some practical reasons why one would want to prefer `interface` over `type` or vice versa. I think the practical arguments point slightly towards using `type`. But to be honest, the practical reasons are not super powerful for one side or the other.
- For this reason, whether to choose `interface` or `type` might mostly just come down to a matter of style. But when choosing the style for your TypeScript code, it makes a lot of sense to use the conventions that already prevail in the ecosystem. The idea is that we want our TypeScript code to look like everyone else's TypeScript code, since it makes it easier for other people to read and understand. (You have probably heard before that [code is read more often than it is written](https://skeptics.stackexchange.com/questions/48560/is-code-read-more-often-than-its-written).)
- So what is the prevailing style in the ecosystem? The answer is `interface`. As mentioned previously, this is mostly a historical artifact of `type` having some buggy behavior.
- This convention is codified inside of the [`@typescript-eslint/consistent-type-definitions`](https://typescript-eslint.io/rules/consistent-type-definitions/) ESLint rule. The rule ensures a consistent style for type definitions throughout a codebase, and the default option is `interface`.

## Conclusion

- `interface` and `type` have subtle differences. Sometimes, choosing one over the other will make more sense:
  - It could be that one of them is more semantically appropriate.
  - Or it could be that one of them has a specific feature that you need.
- Thus, in many codebases, we will have a mix of both `interface` and `type`, and it is impossible and/or not practical to remove one or the other entirely.
- Subsequently, the debate mostly centers around which one to use for the "default" case of a basic object.
- `type` has some small practical advantages over `interface`, but they matter much less in "modern" code (where we can ignore that declaration merging exists).
- This means that whether to use `interface` or `type` for the default case is mostly a matter of style, in the same way that using tabs versus spaces is mostly a matter of style.
- For the [same reason that would you want to use Prettier](./prettier-argument.md) to decide the debate between tabs versus spaces, it makes sense to follow the existing ecosystem convention of preferring `interface`.
