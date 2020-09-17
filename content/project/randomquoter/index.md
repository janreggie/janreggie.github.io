---
title: "randomquoter: Random Quote Machine"
type: project
date: 2020-09-17T17:34:33+08:00
publishdate: 2020-09-17
lastmod: 2020-09-17
draft: false
description: "Creating a random quote machine for freeCodeCamp's Front End Libraries certification using create-react-app and TypeScript"
tags:
    - react
    - javascript
    - freecodecamp
categories:
    - "project"
link: "https://janreggie.github.io/randomquoter/"
---

In this post, I talk about randomquoter, a random quote machine,
which is the first project of five for the "Front End Libraries" certification in freeCodeCamp,
wherein I used `create-react-app` to bootstrap my React application
as well as TypeScript to greatly improve type safety.

<!--more-->

## Bootstraping using create-react-app

I have never built a React app prior to this one.
On freeCodeCamp I was concentrated on writing individual code snippets
and seeing how they build together to form the final product,
but not writing the more internal guts of the project.
In addition, while I could write the application in [CodePen](https://codepen.io/),
the application that I will be making may be too complex for it,
and running a development environment on a browser is a bit clunky.

Fortunately, [create-react-app](https://create-react-app.dev/) exists
which generates a React app with a simple command.
I had to modify `package.json` to change the URL to [the Github Pages site](https://janreggie.github.io/randomquoter/),
and replace a few default parameters such as the app name and description.
From there, it's just a matter of modifying `App.js` to my liking.

## Dabbling with TypeScript

When I finished the lessons for the certification,
I wanted to expose myself to [TypeScript](https://www.typescriptlang.org/),
so I decided to incorporate its use in the project.
It's a really interesting technology,
and as someone who's used to statically typed languages such as C++ and Go,
having this strict typing is beneficial for me.

There are however some quirks which I didn't know about initially,
such as using constructs like `React.Component<props, state>`.
It took me a while to determine why the compiler is acting up
but I got everything running at least.

I initially intended to justuse vanilla JavaScript for this project
but I decided after writing a bit of code that I wanted to try out TypeScript.
I added TypeScript by manually invoking `yarn add` and changing files manually.
This led to a somewhat awkward mix of JavaScript and TypeScript files.
Next time I'll use `--template typescript` from the start.

Since I am used to typing in JavaScript from freeCodeCamp,
this repository contains code that doesn't seem to be idiomatic in TypeScript.
Apologies for that but I am learning along the way.

## Program structure and layout

The overall structure is simple.
The app contains static HTML+CSS `Header` and `Footer`, with the `QuoteContainer` in between.
To keep the Footer at the bottom of the screen layout,
I used MDN's [Sticky footers](https://developer.mozilla.org/en-US/docs/Web/CSS/Layout_cookbook/Sticky_footers) guide.

```tsx
function App() {
  return (
    <div id="App">
      <Header />
      <QuoteContainer />
      <Footer />
    </div>
  );
}
```

{{< optfigure
  src="always-on-top-footer"
  alt="An earlier version of the program, the footer is seen occupying most of the screen when viewed on mobile"
  caption="This is why we check on Mobile. Thanks to a close friend for bringing this to my attention." >}}

### Generating quotes

Before I talk about `QuoteContainer`, I want to elaborate how I generate my quotes.

I have a class `Quote` which describes a quote.
I'll let the code speak for itself:

```ts
// Quote is an interface that has an Author and a Text, but may have an optional Context.
// The Text is the text of the quote.
// The Author is the person who said the Quote.
// The Context contains details where the Quote may be from;
// for example, as an excerpt from a book.
export interface Quote {
  readonly Text: string;
  readonly Author: string;
  readonly Context?: string;
}
```

I then made a standard pool of quotes that's available on default.
I just took a few from a few pages from [Wikiquote](https://en.wikiquote.org/wiki/Main_Page):

```ts
// DefaultQuotes is a default array of Quotes
export const DefaultQuotes: Array<Quote> = [
  {
    Text: `Mona Lisa, Mona Lisa, men have named you
You're so like the lady with the mystic smile
Is it only 'cause you're lonely they have blamed you?
For that Mona Lisa strangeness in your smile?`,
    Author: `Nat King Cole`,
    Context: `from his song Mona Lisa`
  },
  {
    Text: `A learning experience is one of those things that say, "You know that thing you just did? Don't do that."`,
    Author: `Douglas Adams`,
    Context: `Interview in The Daily Nexus (5 April 2000), reprinted in The Salmon of Doubt`
  },
  // this list continues on
]
```

An issue I have though is in encoding newlines, and the first quote appears like it's one line.
[There is a solution which uses simple CSS](https://stackoverflow.com/a/44413780/14020202).
In addition there are just too few quotes, and I should add more.
However, my current system is just too slow to program in React,
so those [issues](https://github.com/janreggie/randomquoter/issues) will be waiting for me some time in the future.

I then made a QuoteMachine which takes in an array of Quotes
and has a generate method:

```ts
// QuoteMachine is an object that can generate quotes from a pool of quotes.
// The quotes must be an array of Quote items.
export class QuoteMachine {
  quotes: Array<Quote>

  constructor(pool: Array<Quote>) { this.quotes = [...pool] }

  generate(): Quote {
    // rng generates a random number from 0 to len(quotes)
    const rng = () => Math.floor(Math.random() * this.quotes.length)
    return this.quotes[rng()]
  }
}
```

Fairly simple. Now for the frontend that takes in these quotes...

### Using QuoteContainer

QuoteContainer is a way to contain the box containing the quote,
a New Quote button, and a Tweet Quote button.
It uses a QuoteMachine which takes in DefaultQuotes,
and stores a state containg the current quote.

```tsx
class QuoteContainer extends React.Component {
  quoteMachine: QuoteMachine
  state: { quote: Quote }

  constructor(props: object) {
    super(props)
    this.quoteMachine = new QuoteMachine(DefaultQuotes)
    this.state = { quote: this.quoteMachine.generate() }
    this.generateQuote = this.generateQuote.bind(this)
  }

  generateQuote(): void {
    this.setState({ quote: this.quoteMachine.generate() })
    console.log(this.state.quote)
  }

  render(): JSX.Element {
    const newQuoteArgs = {generator: this.generateQuote}
    return (
      <div id="quote-box-conatiner">
        <div className="card border-primary mx-auto bg-light" id="quote-box">
          <div className="card-body">
            <QuoteBox {...this.state.quote} />
            <div className="list-inline">
              <NewQuote {...newQuoteArgs} />
              <TweetQuote {...this.state.quote} />
            </div>
          </div>
        </div>
      </div>
    )
  }
}
```

Notice that `this.state.quote` is passed onto `QuoteBox` for it to be displayed
and to `TweetQuote` for a proper Tweet to be generated.
In addition, `generateQuote` is passed to NewQuote which alters `this.state.quote`
and thus alters the contents of `QuoteBox` and `TweetQuote`.

```tsx
class NewQuote extends React.Component {
  generator: () => {}

  constructor(props: { generator: () => {} }) {
    super(props)
    this.generator = props.generator
  }

  render(): JSX.Element {
    return (
      <li className="list-inline-item">
        <button id="new-quote" className="list-inline-item btn btn-primary" onClick={this.generator}>New Quote</button>
      </li>
    )
  }
}

class TweetQuote extends React.Component<Quote> {
  render(): JSX.Element {
    let text = encodeURIComponent(`"${this.props.Text}" - ${this.props.Author}`)
    return (
      <li className="list-inline-item">
        <a id="tweet-quote" className="btn btn-primary" href={`https://twitter.com/intent/tweet?text=${text}`}>Tweet Quote</a>
      </li>
    )
  }
}
```

`NewQuote` isn't too interesting, since it's just a button that has an `onClick` property.
`TweetQuote` though has this `encodeURIComponent` going on it.
This is done in order to escape characters that would otherwise make the URL malformed.

## Testing using freeCodeCamp's test bundle

freeCodeCamp mandates that the random quote machine fulfill [these user stories](https://www.freecodecamp.org/learn/front-end-libraries/front-end-libraries-projects/build-a-random-quote-machine).
To check if the tests go through, they included a [link to some JavaScript code](https://cdn.freecodecamp.org/testable-projects-fcc/v1/bundle.js)
that creates a little hamburger on the upper-leftmost part of the code,
where a developer can check if it fulfills all tests for a certain project.

{{< optfigure
  src="testsuite"
  alt="Screenshot of the FCC test suite, showing that it passes all 12 tests"
  caption="One project down. Four to go." >}}

## Hosting to Github Pages

Since this is a static website that doesn't need to interact with any server,
Github Pages seems like a good service to use,
and my code is already in Github so it just needs a simple workflow YAML file
in order to compile and publish to gh-pages.

I used this Medium post,
[*Deploy create-react-app to GitHub Pages using GitHub Actions*](https://medium.com/swlh/deploy-create-react-app-to-github-pages-using-github-actions-4e95ae7fd65f),
to aid myself in the process.
It uses the action [peaceiris/actions-gh-pages@v3](https://github.com/peaceiris/actions-gh-pages)
to publish the resources compiled by npm.
I however used `GITHUB_TOKEN` instead of `ACTIONS_DEPLOY_KEY`
since it frees me from doing more configuration
