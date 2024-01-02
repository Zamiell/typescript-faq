# The Prettier Argument

The "Prettier Argument" is the proposition that "it is a very good for code to adhere to the same conventions and style as other people's code". But this is not particularly intuitive. You might think to yourself: "Why does it matter what other people's code looks like? I just want my code to be the best that it can possibly be."

The answer lies in autoformatters. After using [Prettier](https://prettier.io/) or [`rustfmt`](https://github.com/rust-lang/rustfmt) or [`gofmt`](https://pkg.go.dev/cmd/gofmt) for the first time, your experience might go something like this:

- "I hate the way everything looks."
- "I am getting used to the way everything looks."
- "I am now used to the way everything looks. And the fact that everything reorganizes itself into this style automatically saves me _so much time_. As I code, I constantly save the file to summon the autoformatter, which makes me able to code twice as fast as I used to."
- "The entire codebase is much easier to read now, because everything has a consistent style."

But the mind-bending part starts to happen once you start working in and looking at other codebases.

- "The new project that I am working on uses the same formatting tool. I immediately feel at home, because all of the code looks identical to the last project."
- "Even though the projects were written by different people, it all looks like it was written by one person, which is great."

But the rabbit hole goes deeper.

- "Blogs on the internet show code that also uses the same formatting tool. It is now effortless to read other random people's code."
- "Most code on StackOverflow also uses the same formatting tool. I can copy paste other people's code into my own codebase without having to change anything!"
- "I have become so used to this that it actually becomes annoying when I see code that is not formatted by the tool. It would be great if the _entire_ ecosystem would agree on the style from this tool!"

This was basically my experience with writing [Golang](https://go.dev/) and using `gofmt`, which is distributed as part of the language. Because it was part of the language itself, `gofmt` got immediate and widespread adoption. Even most random StackOverflow questions have "correct" formatting!

Why isn't every programming language like Golang? I think that part of the problem is that when people try out a new language, they often use the same formatting and conventions that they used in their previous language. This fractures the ecosystem and makes everyone's code inconsistent and hard to read. The lesson of Go is that whenever you code in a new language, you should use the standard style that everyone else uses for that language. In this way, every language can have the superpower that Go has.

Maybe your experience with Prettier wasn't quite as earth-shattering as mine was with `gofmt`. If not, that's okay. Hopefully you can at least see the value in an ecosystem converging around a basic style. Based on this philosophy, we might come up with the following axiom:

- For matters of style (e.g. tabs vs spaces), you should always defer to what the most popular opinion is. In other words, you are not allowed to have your own opinions.
- For matters of mostly-style that actually have differences, it gets more tricky. Sometimes, the most popular choice in the ecosystem is suboptimal for one reason or another. So now we have to weigh the pros of getting the optimal thing in our personal codebase, versus making our code slightly different from the rest of the ecosystem. This means that _sometimes_, we get to have our own opinions. But we have to pick our battles, because every time we deviate from the ecosystem we get further and further away from our goal.

This is the _Prettier Argument_, which is just a rehashed version of the older argument [code is read more often than it is written](https://skeptics.stackexchange.com/questions/48560/is-code-read-more-often-than-its-written). A lot of people don't care about the _Prettier Argument_. "It's my personal codebase, I'll do what exactly how I please!" "But don't you want your code to be similar to others?" This is kind of a weak-sounding retort, since making your code easier to read for others who might not even be on your team is a pretty abstract benefit - it isn't immediately obvious why you would even want this.
