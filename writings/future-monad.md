## Intuition of Monad from Future

This is yet another short article attempting to provide some
insights on `Monad`. It assumes the knowledge of `Future` in scala and
basic knowledge of Scala's syntax.

A famous definition of Monad looks like this

> Monad is just a monoid in the category of endofunctors

Unfortunately, even though this definition is succinct and technically correct,
it is not very useful to many of us.

Today I will show some special properties of Monad using `Future` type as
an example, which hopefully will be familiar to most of you.

### What is `Future` ?

`Future` type in scala standard library is a common way for programmer to express
asynchronous computation. One can think it is a wrapper that wrap some computation
so that it does not block the current execution path.

Example

```scala
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global
import java.lang.Thread

val slow = Future { Thread.sleep(5000); println("slow task completed") }
val fast = Future { println("fast task completed") }

// > fast task completed
// > slow task completed
```

As you can see, `fast` task started and completed before slow task completed,
it is not blocked by `slow`

This property is very useful, and is a big improvement compared to `callback` which
provides similar feature, which you will see why shortly.

So now that we know what `Future` does, how does it relates to `Monad`?

It turns out `Future` is a `Monad` type (Not really, but we will discuss that later)

### What is `Monad` ?

Turn out we still need to define `Monad` :(

Let's make use of what we already know about `Future` to understand `Monad`

I mentioned `Future` provides similar feature provided by `callback`, so what's the differences ?

**Callback Example**

We are using an example that does

1. Predict lottery number using our awesome prophecy AI
2. Buy the number predicted
3. Celebrate

```scala

// prediction is hard, so this will be a slow computation
def predictLotteryNumber[T](whenDone: String => T): Unit = { // .... }
def buyLotteryNumber[T](no: String, whenDone: Unit => T): Unit = { //... }

def celebrate(): Unit = println("Open champagne !!")

predictLotteryNumber(number => buyLotteryNumber(number, celebrate))
```

Note that async computation takes a `whenDone` callback
argument, which will be called when the computation finishes, to compose them,
we need to have nested functions call, like this

`predictLotteryNumber(number => buyLotteryNumber(number, celebrate))`

we call `buyLotteryNumber` when prediction is done,
and then call `celebrate` when `buyLotteryNumber` finished.

This is not great, imagine you have 10 async computation that need to
be chained together, the number of nesting will be terrible.

Another downside is that callback mandates programmer to handle all control
flow in one place, as it is not a value that can be passed around.

**Future Example**
`Future` have better composability compared to `Callback`,
thanks to 2 important method available on `Future[A]` :

* `map(fn: A => B): Future[B]`
* `flatMap(fn: A => Future[B]): Future[B]`

Notice that both methods takes in a `function` as an argument, both methods allows you to
use the results of the asynchrounous computation easily.

eg.
```scala

// prediction is hard, so this will be a slow computation
def predictLotteryNumber(): Future[String] = { //... }

def buyLotteryNumber(no: String): Future[Unit] = { //... }

def celebrate(): Unit = println("Open champagne !!")

val buyLotteryTask: Future[Unit] = predictLotteryNumber()
    .flatMap(buyLotteryNumber)

val celebrateTask = buyLotteryTask.map(celebrate)

```

This is pretty neat as we are able to define `predictLotteryNumber` and `buyLotteryNumber`
separately, and chain them up later, it's important that the chaining can be separated,
unlike callback. This means it can be separate file, this gives programmer better control
on the program flow.

