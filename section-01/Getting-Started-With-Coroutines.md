## Getting Started With Coroutines
In this chapter, you will:
- Learn about `routines` and how a program controls its execution flow.
- Learn about suspendable functions and suspension points in code.
- `Launch` your first Kotlin coroutine, creating `jobs` in the background.
- Practice what you’ve learned by creating a few typical tasks, including posting to
the `UI thread`.

## Executing Routines
Every time you start a process — launching an application, for instance — your
computer creates something called a `main routine`. This is the core part of every
program because it’s where you set up and run all the other components in your
code. As in the most basic learning samples, you often have a `main` function, which
prints Hello World. That main function is the entry point of your program and is
part of the main routine.

But as your programs gets bigger, so does the number of functions and the number of
calls to other functions. Whenever you call some other function in the `main` block,
you start something called a `subroutine`. A subroutine is just a routine, nested
within another routine. The computer places all of these routines on the `call stack`, a
construct that keeps track of what’s currently running and how the current routine
has been called. When a subroutine finishes running, it is `popped` off the stack, and
control is passed back to the caller routine. Lastly, if the stack is empty, and there’s
nothing else to run, the program finishes.

Invoking a subroutine is like doing a `blocking call`. A `coroutine` is then a subroutine
that you can invoke as a `non-blocking call`. Because of this, the main difference
between a standard subroutine and a coroutine is that the latter can run in parallel
with other code. You can start and forget them, moving on to the rest of the program.

## Launching a Coroutine
Coroutines have several concepts that you have to learn to understand their inner
works. Before we dive into those concepts, let’s try to launch a few coroutines so that
we can analyze these concepts with a code snippet at hand.

```
fun main() {
  (1..10000).forEach {
    GlobalScope.launch {
      val threadName = Thread.currentThread().name
      println("$it printed on thread $threadName")
     }
   }
   Thread.sleep(1000)
}
```
Since launching your first coroutine is not **that** fascinating, you’ll launch your first
ten thousand coroutines! Now, launching ten thousand threads is a bit tedious for a
computer and most programs would get an **OutOfMemoryException**. But since
coroutines are extremely lightweight, you’re able to launch a large number of them,
without any performance impact.
If you run the program, you should see a lot of text, each line saying which number it
is printing and on which thread.

There are a few important things to notice in the snippet above.
First, when launching coroutines, you have to provide a `CoroutineScope` because
they are background mechanisms which don’t really care about the `lifecycle` of their
starting point.

What would happen if the program ended before the completion of the coroutine
body? In this case, you use something called the GlobalScope, which makes explicit
the fact that the coroutine lifecycle is bound to the `lifecycle of the application`.
Because of this, you also need to put the current thread on hold, calling
Thread.sleep(1000) in the end of main.

Secondly, the coroutine body here is represented by the block of code passed as the
parameter to launch, which is called a `coroutine builder`. These special functions
let you build coroutines that will run your code, based on the configuration you give
them, which we’ll talk about in a moment.

This is the basic explanation of what you’re doing, but these concepts are more
complex than that. Let’s dive into coroutine builders before we analyze what a
CoroutineScope is and how it all comes together.

## Building Coroutines
You’ve heard the term `launching coroutines` quite a few times now. In truth, you
first have to use a `coroutine builder`. The Coroutines library has several coroutine
builder functions for you to use to start a new coroutine. In the previous example,
you used launch with the following signature:
```
public fun CoroutineScope.launch(
 context: CoroutineContext = EmptyCoroutineContext,
 start: CoroutineStart = CoroutineStart.DEFAULT,
 block: suspend CoroutineScope.() -> Unit
): Job
```
As you can see, launch has a few arguments that you can pass in: a
`CoroutineContext`, a `CoroutineStart` and a lambda function, which defines what’s
going to happen when you launch the coroutine. The first two are optional. This
function returns a `Job` class.

A CoroutineContext is a persistent dataset of contextual information about the
current coroutine. This can contain objects like a `Job` and `Dispatcher` of the
coroutine, both of which you will learn about later. Since you haven’t specified
anything in the snippet above, it will use the EmptyCoroutineContext, which points
to whatever context the specified CoroutineScope uses. You can create custom
contexts if you’d like, but for the most part, the existing ones are sufficient.

The CoroutineStart is the mode in which you can start a coroutine. Options are:

- `DEFAULT`: Immediately schedules a coroutine for execution according to its
context.
- `LAZY`: Starts coroutine lazily.
- `ATOMIC`: Same as DEFAULT but cannot be cancelled before it starts.
- `UNDISPATCHED`: Runs the coroutine until its first suspension point.
  
The lambda block you pass in is the code that the coroutine will execute. If you check
the previous definition of launch, you will notice that this lambda block has a
somewhat different signature than standard lambda blocks. Its signature is block:
suspend `CoroutineScope.() -> Unit`.

It’s a lambda with a receiver of type `CoroutineScope`. This allows you to have nested
Jobs, as you can launch more coroutines from another launch block. Another thing
that is specific is the suspend modifier.

As you’ve learned, coroutines build upon the concept of suspendable functions. You
can use the modifier at hand to mark a lambda or another function suspendable.
You’ll learn a bit more about suspendable functions in the next chapter.
Finally, once you successfully launch a coroutine, you receive a Job. As the name
states, the Job represents some work that the coroutine encapsulates and lets you
control.
Before we explain how Jobs work, we have to take a small detour and dive into what
a CoroutineScope does!

## Explaining Jobs
If you’ve noticed, launch and a lot of concepts in coroutines refer to a `Job` which
you create and run. What can you do with a Job?

When you launch a coroutine you basically ask the system to execute the code you
pass in using a lambda expression. That code is not executed immediately, instead
it’s inserted into a queue.

A Job is a `handle` to the coroutine in the queue. It only has a few fields and
functions, but it provides a lot of extensibility. For instance, it’s possible to introduce
a dependency relation between different Job instances using join. If Job `A` invokes
join on Job `B`, it means that the former won’t be executed until the latter has
completed. It’s also possible to set up a `parent-child` relation between Job instances
using specific `coroutine builders`. A Job cannot complete if all its children haven’t
completed. A Job must complete in order for its parent to complete.
The Job abstraction makes this possible through the definition of `states`, whose
transitions follow the workflow described by the following diagram:

![image](https://github.com/oybekjon94/coroutines-book-notes/assets/91370134/4b45e887-59eb-4703-bb0f-d3202fa132dd)

When you launch a coroutine, you create a Job, which is always in the `New` state. It
then goes directly into the `Active` state by default unless you’ve supplied the LAZY
CoroutineStart parameter in the coroutine builder you’ve used. You can also move
a Job from the New to the Active state using start or join. A running coroutine is
always in the Active state. As you can see in the state diagram, the Job can complete
or can be canceled.

It’s very important to note how completion and cancellation work for dependent Job
instances. In particular, you can see that a Job remains in the `Completing` state until
all of its children complete. It’s important to say that the Completing state is
internal and, if queried from outside, the Job will result in the Active state.
States are fundamental because they give you information about what’s going on
with the coroutines and what you can do with them. You can also query the state of a
Job or simply iterate over the children and do something with them.

## Canceling Jobs
When you launch a coroutine and the builder creates a Job, many things can happen.
An exception can occur or you might need to cancel the Job because of some new
conditions in the application. Consider a list of images that you download from the
network. Every time you need to display an image in a list item you start a coroutine
for the download. This download might fail because there’s no connection and you
have to handle the related exception. Or the download might be canceled because
the user scrolls the list and the image goes out of the screen before it’s available. It’s
very important to understand how you can manage these use cases when using
coroutines.

Usually, an `uncaught exception` would cause the entire program to crash. However,
since coroutines have suspending behavior, if an exception occurs, it can also be
suspended and managed later.

There is a much easier way to handle a `cancellation`. You can do it by invoking
cancel on the related Job. The system is then smart enough to understand the
dependencies between Jobs. If you cancel a Job you automatically cancel all its
children. If it has a parent, the parent is canceled. A parent of a Job is also canceled
if one of its children fails.

> [!NOTE]
> It’s possible to use a special parent job, which doesn’t require all it’s
children to complete happily - they can be cancelled or can fail independently.
This version of a Job is called the SupervisorJob.

As mentioned before, even though you cancel a Job , your code might not be cooperative with cancellation events. You can check this by using its isActive
property. If your code does computational work, without checking the isActive flag,
it isn’t listening to cancellation events. So running while loops with the isActive
flag is safer than with your own conditions. Or you should at least try to depend on
isActive, on top of your conditions.

## Digging Deeper Into Coroutines
So far you’ve launched a large amount of coroutines, and with it you’ve created
multiple coroutine jobs. But there are other things you can do when launching a
coroutine. For example, if you have some work that you have to first delay for a
period of time, before running, you can do so with delay. Open up Main.kt again
and replace the code with the following snippet and import delay:

```
fun main() {
 GlobalScope.launch {
 println("Hello coroutine!")
 delay(500)
 println("Right back at ya!")
 }
 Thread.sleep(1000)
}
```
If you run the code above, you should see “Hello coroutine,” in the console and
briefly after that, “Right back at ya.” delay is really useful because you can
effectively wait for the given amount of time and then run work when everything is
ready. And most importantly it does not sleep on a thread or block, it just suspends
the coroutine.

## Dependent Jobs in Action
So far, you’ve learned that, every time you launch a coroutine, you can get a Job
reference. You can also create dependencies between different Jobs — but how? Just
replace the previous code with this:

```
fun main() {
 val job1 = GlobalScope.launch(start = CoroutineStart.LAZY) {
   delay(200)
   println("Pong")
   delay(200)
 }
 GlobalScope.launch {
   delay(200)
   println("Ping")
   job1.join()
   println("Ping")
   delay(200)
 }
 Thread.sleep(1000)
}
```

and import CoroutineStart. Going through the code above:
- You first launch a coroutine that contains some delays and prints the Pong word,
saving the created Job into the job1 reference.
- Then you launch a second coroutine that contains a couple of printlns but also
invokes the join function on job1.

What is the expected output? If you follow the code you would expect to see Pong
and then Ping twice, but this is not the case. As you can see, you used the
CoroutineStart.LAZY value as CoroutineStart for the first Job and this means
that the related code is going to be executed only when you actually need it.
This happens when the second coroutine invokes the join function on job1. This is
why the result of the previous code is Ping, then Pong and finally Ping again.

## Using Standard Functions With Coroutines
Another thing you can do with coroutines is build retry-logic mechanisms. Using
repeat from the standard library, paired up with delay you learned above, you can
create code that attempts to run work in delayed periods of time. Once again, replace
the Main.kt code with the next snippet:

```
fun main() {
  var isDoorOpen = false
  println("Unlocking the door... please wait.\n")
  GlobalScope.launch {
    delay(3000)
    isDoorOpen = true
 }
 GlobalScope.launch {
   repeat(4) {
     println("Trying to open the door...\n")
     delay(800)
     if (isDoorOpen) {
     println("Opened the door!\n")
     } else {
     println("The door is still locked\n")
     }
   }
 }
 Thread.sleep(5000)
}
```
Try running the code. You should see that someone’s trying to open the door a few
times before ultimately succeeding. So using delay, and repeat from Kotlin’s
standard library, you managed to build a mechanism that tries to run some code
multiple times before you meet a time or logic condition. You can use the same flow
to build networking back-off and retry logic. And once you learn how to return
values from coroutines later in this book, you’ll see how powerful this can be.

## Key Points
- You can build coroutines using `coroutine builders`.
- The main coroutine builder is the `launch` function.
- Whenever you launch a coroutine, you get a Job object back.
- `Jobs` can be canceled or combined together using the join function.
- You can nest jobs and cancel them all at once.
- Try to make your code `cooperative` — check for the state of the job when doing
computational work.
- Coroutines need a `scope` they’ll run in.
- You can mark your functions with @DelicateCoroutinesApi in case you’re using
the GlobalScope for a long running operation or something that lives with your
app lifecycle.
- Posting to the UI thread in advanced applications is as easy as passing in the
Dispatchers.Main instance as the context.
- Coroutines can be postponed, using the delay function.
