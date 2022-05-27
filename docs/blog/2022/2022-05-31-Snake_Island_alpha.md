---
layout: fable-blog-page
title: Announcing Snake Island (Fable 4) Alpha Release
author: ncave, Dag Brattli, Alfonso García-Caro
date: 2022-05-31
author_link: https://twitter.com/fablecompiler
author_image: https://github.com/fable-compiler.png
# external_link:
abstract: |
---

Hey there everybody! It's been some time without Fable news but we're bringing you something today that we hope will get you excited :) Today we're announcing the first **alpha** release of Snake Island, codename for Fable 4. If you've been following us on Twitter you'll probably know this the most ambitious Fable release to date, extending the compilation targets for F# beyond JS, and include languages like Python, Rust or Dart. The ultimate goal is to convert F# into a super-powerful DSL you can use to design your programs and algorithms and still have the freedom to choose the platform you want to run your code on.

We also believe that, same as it happened with JS, the interaction with other ecosystems and communities is tremendously beneficial for the F# language and F# developers. Although it makes for a good catch-phrase, Fable wasn't born to make F# "the One to rule them all", instead we'll be most happy if Fable can be a bridge for F# developers to explore other worlds beyond .NET. Some of them have fully crossed the bridge and become Typescript/JS experts. This is great too and we hope it can also happen in the future for Python, Rust or Dart!

Please see below for specific details about each of the new language targets and how you can try this one.

:::warning
Don't update your current Fable JS projects just yet. In principle, everything should work the same as we're aiming for no breaking changes. But there won't be many JS new features (except for JSX compilation, see below) and compiler plugins do need to be updated. So if you're using `ReactComponent` attribute in your project, it won't work yet with Snake Island.
:::

## Python

...

## Rust

...

You can track the current progress for Rust compilation [here](https://github.com/fable-compiler/Fable/issues/2703).

## Dart

Cross-platform has been the holy grail of app development for a long time. The .NET team is also be very active in this space with [the recent release of MAUI](https://devblogs.microsoft.com/dotnet/introducing-dotnet-maui-one-codebase-many-platforms/), but there's another project that has gained a lot of traction over the years thanks to its high-quality and great development experience: [Flutter](https://flutter.dev/). And with React-inspired [declarative UIs](https://docs.flutter.dev/development/ui/widgets-intro), Flutter's model is very familiar to most Fable developers.

Dart, the language used for Flutter, is a typed language, with null-safety and very similar to C# so it's a perfect for F#. There's still work to do, but thanks to the analysis tools in Dart we're polishing the code generation to get a very clean output. We expect to have even better F#-Dart interoperability than we have with JS!

You can clone [this repo](https://github.com/alfonsogarciacaro/fable-flutterapp) to try a simple Elmish Todo app running in Dart. Note the app translates the original Elmish code (with small edits) to Dart so it's already quite powerful. Follow the README instructions to start the app, the commands in `build.sh` will start Fable and then you can use a Flutter-compatible IDE (or the [Flutter CLI](https://docs.flutter.dev/get-started/install)) to run and debug Flutter.

When writing your own code for Dart compilation please note the following:

- Dart has null safety, so the compiler may complain, e.g. when assigning null to a string, even if this is possible in F#. For this reason, `Unchecked.default<_>` won't work in many cases, although you can use it to initialize a mutable variable (Fable translates this is an empty `late` var in Dart).
- For now, we are compiling F# options as Dart nullables. It's working well and plays nicely with Dart native code, but it has a limitation: you cannot have nested options (be careful also with generic options). We may change the F# option representation if this proves to be too limiting.
- If Dart compiler complains about generics inferred by F#, try adding type annotations to your F# code.
- At the time of writing, print formatting with `(s)printf` is not supported. Use string interpolation and `Fable.Core.Dart.print` instead.

Besides the missing code and library features, the biggest challenge for writing Flutter apps with F# is to have proper bindings for the enormous Flutter widget library. We're working to find a way to do it in a (semi)automated way as we have with ts2fable. [Any help is appreciated!](https://github.com/fable-compiler/Fable/issues/2878)

You can track the current progress for Dart compilation [here](https://github.com/fable-compiler/Fable/issues/2877).

## JSX

Fable JS wasn't supposed to get any new feature with Snake Island, but at the last minute we've implemented support for yet another language or, more exactly, a language-in-a-language. [JSX](https://reactjs.org/docs/introducing-jsx.html) was introduced by the React team as a way to declare HTML-like UIs directly in JavaScript without the need of a template language. Although Fable has supported React from the start, it never used JSX. Instead it directly output the JS code generated by JSX templates. This made sense because for React JSX was only syntax sugar and this way we could save a compilation step.

However, given the popularity gained by JSX, there are now tools that take advantage of this compilation step to perform code analysis or other operations. Like [SolidJS](https://www.solidjs.com/) which uses JSX to analyze the dependency tree in your code and translate the declarative render functions into surgical imperative updates that don't need a Virtual DOM and are smaller and faster than equivalent React apps (while maintaining the same developer experience overall). We have already implemented a [Feliz](https://zaid-ajaj.github.io/Feliz/)-like API to create Solid apps with good results. For example, the following code:

```fsharp
open Browser
open Feliz.JSX
open Fable.Core

[<JSX.Component>]
let Counter() =
    let count, setCount = Solid.createSignal(0)
    let doubled() = count() * 2
    let quadrupled() = doubled() * 2

    Html.fragment [
        Html.p $"{count()} * 2 = {doubled()}"
        Html.p $"{doubled()} * 2 = {quadrupled()}"
        Html.br []
        Html.button [
            Attr.className "button"
            Ev.onClick(fun _ -> count() + 1 |> setCount)
            Html.children [
                Html.text $"Click me!"
            ]
        ]
    ]

Solid.render(Counter, document.getElementById("app-container"))
```

Which looks like much of the current existing Feliz apps, will be translated by Fable to:

```js
import { createSignal } from "solid-js";
import { render } from "solid-js/web";

export function Counter() {
    const patternInput = createSignal(0);
    const setCount = patternInput[1];
    const count = patternInput[0];
    const doubled = () => (count() * 2);
    const quadrupled = () => (doubled() * 2);
    return <>
        <p>
            {`${count()} * 2 = ${doubled()}`}
        </p>
        <p>
            {`${doubled()} * 2 = ${quadrupled()}`}
        </p>
        <br></br>
        <button class="button"
            onClick={(_arg1) => {
                setCount(count() + 1);
            }}>
            Click me!
        </button>
    </>;
}

render(() => <Counter></Counter>, document.getElementById("app-container"));
```

Which in turn gets translated by Solid into something similar to:

```js
import { render, createComponent, delegateEvents, insert, template } from 'solid-js/web';
import { createSignal } from 'solid-js';

const _tmpl$ = /*#__PURE__*/template(`<p></p>`, 2),
      _tmpl$2 = /*#__PURE__*/template(`<br>`, 1),
      _tmpl$3 = /*#__PURE__*/template(`<button class="button">Click me!</button>`, 2);

function Counter() {
  const patternInput = createSignal(0);
  const setCount = patternInput[1];
  const count = patternInput[0];

  const doubled = () => count() * 2;

  const quadrupled = () => doubled() * 2;

  return [(() => {
    const _el$ = _tmpl$.cloneNode(true);

    insert(_el$, () => `${count()} * 2 = ${doubled()}`);

    return _el$;
  })(), (() => {
    const _el$2 = _tmpl$.cloneNode(true);

    insert(_el$2, () => `${doubled()} * 2 = ${quadrupled()}`);

    return _el$2;
  })(), _tmpl$2.cloneNode(true), (() => {
    const _el$4 = _tmpl$3.cloneNode(true);

    _el$4.$$click = _arg1 => {
      setCount(count() + 1);
    };

    return _el$4;
  })()];
}

render(() => createComponent(Counter, {}), document.getElementById("app"));

delegateEvents(["click"]);
```

Your nice, declarative code has been magically transformed into performant imperative instructions :)

Soon JSX will be available to Fable React apps too! Though as commented above, it won't make a big difference in React (and in fact it can be more limiting as JSX needs to be interpreted statically, so list comprehensions for example can be trickier) but we are also introducing JSX templates which can be particularly helpful when bringing external HTML code to your app. And with the [F# template highlighting VS Code extension](https://marketplace.visualstudio.com/items?itemName=alfonsogarciacaro.vscode-template-fsharp-highlight) you can achieve a very similar experience to [Fable.Lit](https://fable.io/Fable.Lit/).

```fsharp
let gradient, setGradient = React.useState(5)

Html.fragment [
    Html.div [
        Html.input [
            Attr.typeRange
            Attr.min 1
            Attr.max 100
            Attr.value $"{gradient}"
            Ev.onTextInput (fun (value: string) -> value |> int |> setGradient)
        ]
    ]

    JSX.html $"""
    <svg height="150" width="300">
        <defs>
            <linearGradient id="gr2" x1="0%%" y1="60%%" x2="100%%" y2="0%%">
            <stop offset={ length.perc gradient } style="stop-color:rgb(52, 235, 82);stop-opacity:1" />
            <stop offset="100%%" style="stop-color:rgb(52, 150, 235);stop-opacity:1" />
            </linearGradient>
        </defs>
        <ellipse cx="125" cy="75" rx="100" ry="60" fill="url(#gr2)" />
        Sorry but this browser does not support inline SVG.
    </svg>
    """
```

Basically this means you'll be able to write JSX code directly into your F# app! And you can try it out already with Solid and Snake Island by cloning [this repo](https://github.com/alfonsogarciacaro/Feliz.Solid).

<br />

Hopefully you're now as excited as we are! You can give Snake Island a go right now by following the instructions for each language above or installing it locally specifying the `snake-island` version range:

```
dotnet tool install fable --local --version 4.0.0-snake-island-*
```

Now you can test the different targets by calling the tool as with Fable 3 and passing the `--lang` option. For example if you want to compile a script to Python, run:

```
dotnet fable MyScript.fsx --lang python
```

:::info
Just remember that supporting the whole .NET Base Class Library is not one of Fable's goal, so calls to `System.IO` and friends won't work. Please check the current [BCL Compatibility Guide](https://fable.io/docs/dotnet/compatibility.html) for JS as reference, though the new language targets still lag behind.
:::

As mentioned above this a very ambitious project, and Fable is still an Open Source project maintained by contributors in their free time. We don't have strict roadmaps or deadlines, Snake Island has emerged because each of the contributors had a personal interest in using Fable to bring F# to an ecosystem they love (if you want Fable to target a new language, contribute!). We're still in the alpha release cycle and there are many challenges left until having a stable version. We can only overcome these challenges with the help of the community, you don't need to dive into the compiler code to fix bugs and add features, but it's very important for contributors to know their work is helping others, get feedback and receive words of encouragement (in Github or Twitter). Community contributions have been crucial for previous Fable releases, let's also bring this one to life together!

Best and Slava Ukraini!