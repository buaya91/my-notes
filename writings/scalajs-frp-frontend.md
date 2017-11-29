This blog post will discuss what Scala.js is suitable for.
I will first introduce current state of web development,
and discuss why scala.js might a sensible choice for web development

## State of Web Development
As of early 2017, ReactJS remain the most popular frontend framework/ecosystem.

From the point of application architecture, two notable improvements are
 - composable UI components
 - Flux Architecture (aka Uni-directional data flow)

both which made popular by React.

In this blog post I'll talk about how Flux and React get it's fame and how does Scalajs come into play.

## What's the big deal about React and Flux?
Below are some important improvements that made popular by React.

### 1. Embrace UI component as code
React creates a fair amount of knee jerk reaction on how ugly JSX is when they
first introduce JSX to encode Virtual DOM.

If you dont't know what is JSX, basically it allows developer to write HTML-DOM-like tags in javascript,
for example

```javascript
class ShoppingList extends React.Component {
  render() {
    return (
      <div className="shopping-list">
        <h1>Shopping List for {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}
// shamelessly copied from https://facebook.github.io/react/tutorial/tutorial.html
```

I personally think [Pete Hunt](https://github.com/petehunt) sums it up well

> Template separate technologies, not concerns ~ Pete Hunt

When your UI component is a reflection of data structure, they really are not separate concerns,
and forcing a separation causes people come out with hacky workaround (yes, I am talking about 2-way data binding).

This idea is so profound once you realize all component are describing it's relation to data,
which are essentially functions `(data) => viewComponent`, and functions compose (to a certain extent).

This simple idea made frontend development much simpler, developer can reason about one component at a time,
and compose them at the end, [React Storybook](https://getstorybook.io/) is a nice tool to help developer visualize
component in isolation. (I am not aware of Angular2 equivalent, please leave a comment if you know one)

### 2. Flux aka Uni-directional data flow
I can understand why Facebook want to name it Flux, because `Uni-directional data flow` is too unwieldy LOL.

Back to the topic, the essence of Flux architecture is all data changes should go through the same orchestration flow,
no more `anyone can talk to anyone` data flow.

![complex-models](https://raw.githubusercontent.com/buaya91/image-storage/master/models.png)

instead we delibrately limits the way data can be change, by enforcing 2 rules
- view only read from model directly
- view never modify data directly, instead it emits events to an intermediate orchestration layer

you can visualize it as below

![flux](https://raw.githubusercontent.com/buaya91/image-storage/master/flix.png)

By adding such restriction, it gives developer a better overview on how data flow from one end to the other
, and better control on how view could affect model, by reading dispatcher code, you can know what changes
might happen and how it will affect the application state.

### 3. Reactive UI component
By expressing UI component in relationship with application state, React freed programmer
from manually updating the DOM, this make developer experience less painful compared to JQuery era,
where code like `$('#poor-div').text("New text")` is all over the place.

### 4. Functional Reactive Programming as a Paradigm
All of these features allow a new paradigm shift in frontend development, Functional Reactive Programming (FRP).

If you're using `React + Redux`, you should be familiar with Reactive Programming Paradigm, where React Component will
listen to changes from Redux store, and re-render whenever changes happens.

From a programmer perspective, Functional Reactive Programming is Reactive Programming with higher level
declarative abstraction, allowing programmer to abstract over a stream of value over time, and perform all kind
of operations between such data abstractions.

To give a better sense of what FRP is about, I will use [Pacman](http://macek.github.io/google_pacman/) as an example.

In pacman, there's only 1 external data source, which is user input for direction.

In conventional javascript, you will model the input this way. (All js code assuming ES6)

```javascript
var lastUserInput = null

document.addEventListener('keyup', ev => lastUserInput = ev.key)
```

then somewhere in your main loop, you will read the `lastUserInput` and change PACMAN's direction accordingly.

now, imagine you want to add a feature, when user press `down arrow` and `a` at the same time,
PACMAN will become invisible, and use can only use it once per 10 seconds, how would you implement it.

```javascript

var lastUserDirectionInput     = []
var lastUserAInput             = false

document.addEventListener('keyup', ev => {
  if (ev.keyCode == 65) {   // keycode of 'a'

    lastUserAInput = [true, Date.now()]

  } else if (ev.keyCode >=37 && ev.keyCode <= 40) {   // key code of arrow key

    lastUserDirectionInput = [ev.key, Date.now()]
  }
})

// main loop
while(true) {
  if (Math.abs(lastUserDirectionInput[1] - lastUserAInput[1]) < 300) {
      // make PACMAN invisible
  }
  ...
}

```

The above implementation simply store the time a key is pressed as variable, and read them to compare and make decision
whether the 2 key is pressed at the same time or not.

Here's an implementation using FRP concepts

```
var keyStream = Observable.fromEvent(document, 'keyup')
  .filter(ev => {
    const code = ev.keyCode
    return code == 65 || (code >= 37 && code <= 40)
  })
  .map(ev => ev.keyCode)

var mainLoop = keyStream
  .bufferTime(1000)
  .scan((acc, input) => {
    // game logic
    // extract input and apply
    if (!acc)
      return applyInputToState(initGameState, input)
    ele
      return applyInputToState(acc, input)
  })
```

The FRP example might look a bit alien to you, here's a few things you might not be familiar
* Observable - an abstraction to allow us describe values over time, you can think of it as a stream of value
* bufferTime - group values by time
* scan       - it's like reduce/fold, applying certain operation on value emitted by last event, repeatedly

More reading: [RxJS by example](https://gist.github.com/btroncone/d6cf141d6f2c00dc6b35)

The FRP approach make it clear on what it is trying to do,
and more important, Observables as an abstraction is composable, you can build your application as
dozens of data pipeline and combine them as you wish. Each pipeline will

 * easily testable
 * simple
 * immutable

Note: FRP is nothing new, FRP library like `rxjs` and `bacon.js` existed for years in javascript world, but it
is much popular nowadays as it fits well with React and Flux.

## What Scala.js has to offer
For those haven't tried, [Scala.js](https://www.scala-js.org) is a Scala to js compiler.

It comes with following key features

### 1. Static, expressive type system
I think the internet have enough discussion about what static type system gives you,
the major complain of
type definition obscuring

### 2. Immutable by default
Immutable data structure is very useful in context of efficient rendering, React decide whether to render or not by comparing data of last render, if the data did not change, we dont have to re-render, as the view would be the same.

In most common case where data is mutable, we can only tell if data is equal or not by doing deep comparison, which is more often than not, slower than simply re-render, this is the main reason why [ImmutableJs](https://facebook.github.io/immutable-js/) is developed. But due to mutable nature of JS world, something it is clunky to use and requires alot of `mutable <--> immutable` conversion.

In scala, immutable data is first class citizen, and most of the time, the default.

### 3. Server-client in same programming language
By having backend and frontend all using scala, we can effectively share code and more importantly keep client and server
code in sync, if your backend change protocol without updating client code, it just won't compile.

Another big advantage is when server/client needs to run the same business logic, complex business logic is error prone
and rely on test to ensure correctness, and now you only need to write it once instead of twice.

A good example is networked fast paced game, where client side often need to simulate server behaviour
to overcome latency.

### 4. Simple interopability with Javascript
Javascript have been the programming language for web for almost 20 years, meaning there's a whole lot of existing library/frameworks written in javascript, and building an ecosystem is hard.

Scala.js decided to embrace whatever is available in javascript ecosystem, by providing easy JS-interop, one can call javascript easily in scala.js code, there's way of interoperation.

** Dynamic typed interop**

where everything from the JS world is given `Dynamic` as the type, which does not type check
eg.
```scala
    val jq = js.Dynamic.global.JQuery
    val playground = jq('#playground')

    val newP = document.createElement("p")
    newP.innerHTML = "Hello world!"
    playground.appendChild(newP)
```

The code above looks similar to javascript, except the usage of `val` instead of `var` and the `js.Dynamic.global.JQuery`
which means calling JQuery from global.
This means most javascript code can easily reused in scalajs project.

** Static typed interop**

While the first way of interop is all easy, but it means things might blows up in runtime if methods is missing
or type does not match, which is common in javascript world, but less appealing to scala developer.

Type facade to the rescue, we can write type-facade for any javascript library so that we can call them like normal
Scala code.

eg.
```
@js.native
trait JQueryAjaxSettings extends js.Object {
  var accepts: js.Any = js.native
  var async: Boolean = js.native
  var cache: Boolean = js.native
  var contentType: js.Any = js.native
  var data: js.Any = js.native
}
```

Above is a snippet of ScalaJS Type Facade for Jquery, taken from [JQuery facade](https://github.com/scala-js/scala-js-jquery/blob/master/src/main/scala/org/scalajs/jquery/JQuery.scala), it specify that JqueryAjaxSettings have 5 properties,
so in your caller code, if you do

```
val request = JqueryAjaxSetting.send()
```

this will not compile as `send()` is not defined in the trait.

As you can see, this is more involved, but provides strong type safety, and the type facade can be published and shared,
just like the typings of Typescript.


### 5. Safe and powerful Functional Reactive Programming library
