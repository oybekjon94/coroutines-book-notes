## What Is Asynchronous Programming?
The `UI (user interface)` is a fundamental part of almost every app. It’s what users
see and interact with in order to do their tasks. More often than not, applications do
***complex work***, such as talking to external services or processing data from a
database. Then, when the work is done, they show a result, mostly in some form of
text or images.

The UI must be `responsive`. If the work at hand takes a lot of time to complete, it’s
necessary to provide `feedback` to the user so that they don’t feel like the application
has frozen, that they didn’t click a button properly — or perhaps that a feature
doesn’t work at all.

## Providing Feedback
Suppose you have a function that needs to upload an image. When the user selects
the Upload button, loading bars or spinners appear to indicate that something is
going on and the application hasn’t stopped working. This information is crucial for
a good user experience since no one likes unresponsive applications. But what does
providing feedback look like in code?
Consider the following task wherein you want to upload an image but must wait for
the application to complete the upload:
```
fun uploadImage(image: Image) {
 showLoadingSpinner()
 // Do some work
 uploadService.upload(image)
 // Work’s done, hide the spinner
 hideLoadingSpinner()
}
```
At first glance, the code gives you an idea of what’s happening:
- You start by showing a spinner.
- You then upload an image.
- When complete, you hide the spinner.
  
Unfortunately, it’s not exactly that simple because the spinner contains an
animation, and there must be code responsible for that. showLoadingSpinner must
then contain code such as this:
```
fun showLoadingSpinner() {
 showSpinnerView()
 while(running) {
 rotateSpinnerImage()
 delay()
 }
}
```
showSpinnerView displays the actual UI component, and the following cycle
manages the image rotation. But when does this function actually return?
In uploadImage, you assumed that the spinner animation was running even after the
completion of showLoadingSpinner, so that the uploading of the image could start.
Looking at the previous code, this is not possible.
If the spinner is animating, it means that showLoadingSpinner has not completed.
Once showLoadingSpinner completes, the upload will start. This means that the
spinner is not animating anymore. This is happening because when you invoke
showLoadingSpinner you’re making a blocking call.

## Blocking Calls
A `blocking call` is a function that only returns when it has completed. In the
example above, showLoadingSpinner prevents the upload of an image because it
keeps the `main thread` of execution busy until it returns. But when it returns
(because running becomes false), the spinner stops rotating.
So how can you solve this problem and animate the spinner while you’re running
upload?
Simply put, you need additional threads on which to execute your long-running
tasks.

The `main thread` is also known as the `UI thread`, because it’s responsible for
rendering everything on the screen, and this should be the only thing it does. This
means that it should manage the rotation of the spinner but not the upload of the
image — that has nothing to do with the UI. But if the main thread cannot do this
because that isn’t its job, what can execute the upload task? Well, quite simply, you
need a `new thread` on which to execute your long-running tasks!

Computers nowadays are far more advanced than they were 20, 15 or even 10 years
ago. Back then, computers could only have one thread of execution making them
freeze up if you tried to do multiple things at once. But because of technological
advancements, your applications support a mechanism known as `multi-threading`.
It’s the art of having multiple threads, where each can process a piece of work,
collectively finishing the needed tasks.

## Why Multithreading?
There’s always been a hardware limit on how fast computers could be — that’s not
really about to change. Moreover, the number of operations a single processor in a
computer can complete is reaching the law of diminishing returns, unless we
discover more advanced ways of computing.

Because of that, technology has steered in the direction of increasing the number of
cores each processor has and the number of threads each core can have running
`concurrently`. This way, you could logically divide any number of tasks between
different threads and the cores could prioritize their work by organizing them.
Simply put, multithreading has drastically improved how computer systems optimize
work and what the speed of execution is.

You can apply the same idea to modern applications. For example, rather than
spending large amounts of money on servers with better hardware, you can speed up
the entire system using `multithreading `and the smart application of `concurrency`.

## Comparing the Main and Worker Threads
The `main thread`, or the `UI thread`, is the thread responsible for managing the UI.
Every application can only have one main thread in order to avoid a classical
problem called `deadlock`. This can happen when at least two threads access the
same resources — in this case, UI components — in a different order. The other
threads, which are not responsible for rendering the UI, are called `worker threads` or
`background threads`. The ability to allow the execution of multiple threads of
control is called `multithreading` and the set of techniques used to control their
collaboration and synchronization is called `concurrency`.

Given this, you can rethink how uploadImage should work. showLoadingSpinner
starts a new thread that is responsible for the rotation of the spinner image, which
interacts with the main thread just to notify a refresh in the UI. Starting a new
thread, the function is now a `non-blocking call` and can return immediately,
allowing the image upload to start its own worker thread. When completed, this
background thread will notify the main thread to hide the spinner.

Once the program launches a background thread, it can either forget about it or
expect some result. You will see how background threads process the result and
communicate with the main thread in the following section.

## Sharing Data
In order to communicate, different threads need to share data. For instance, the
thread responsible for the rotation of the spinner image needs to notify the main
thread that a new image is ready to be displayed. Sharing data is not simple, and it
needs some sort of synchronization, which is one of the main benefits of well written
concurrency code.

What happens, for instance, if the main thread receives a notification that a new
image is available and, before displaying it, the image is replaced? In this case, the
application would skip a frame and a `race condition`would happen. You then need
some sort of a `thread safe` data structure. This means that the data structure should
work correctly even if accessed by multiple threads at the same time.

Accessing the same data from multiple threads, maintaining the correct behavior
and good performance, is the real challenge of `concurrent programming`.

There are special cases, however. What if the data is only accessed and never
updated? In this case, multiple threads can read the same data without any race
condition, and your data structure is referred to as `immutable`. Immutable objects
are always thread safe.

As a practical example, take a coffee machine in an office. If two people shared it,
and it wasn’t thread safe, they could easily make bad coffee or spill it and make a
mess. As one person started making a mocha latte and another wanted a black
coffee, they would ultimately ruin the machine — or worse, the coffee.

![image](https://github.com/oybekjon94/coroutines-book-notes/assets/91370134/612116fb-67cb-43c0-9065-6a4fe4c055c9)

What are the data structures that you can use in order to safely share data in a
thread? The most important data structures are `queues` and, as a special case,
`pipelines`.

## Queues
Threads usually communicate using `queues`, and they can act on them as `producers`
or `consumers`. A producer is a thread that puts information into the queue, and the
consumer is the one that reads and uses it. You can think of a queue as a list in which
producers append data to the end, and then consumers read data from the top,
following a logic called FIFO (First In First Out). Threads usually put data into the
queue as objects called `messages`, which encapsulate the information to share.

A queue is not just a `container` as it also provides synchronization in order to allow
a thread to consume a message only if it is available. Otherwise, it waits if the
message is not available. If the queue is a `blocking queue`, the consumer can block
and wait for a new message.

## Handling Work Completion Using Callbacks
Out of all the asynchronous programming mechanisms, `callbacks` are the most often
used. This consists of the creation of objects that encapsulate code that somebody
else can execute later, like when a specific task completes. This approach can also be
used in real life when you ask somebody to push a button when they have completed
some task you have assigned to them. When using `callbacks`, the button is
analogous to code for them to execute; the person executing the task is a `nonblocking function`.

How can you put some code into an object to pass around? One way is by using
interfaces. You can create the interface in this way:
```
interface OnUploadCallback {
 fun onUploadCompleted()
}
```
With this, you are passing an implementation of the interface to the function that is
executing the long-running task. At completion, this function will invoke
onUploadCompleted on the object. The function doesn’t know what that
implementations does, and it’s not supposed to know.

In modern programming languages like Kotlin, which support functional
programming features, you can do the same with a `lambda` expression. In the
previous example, you could pass the lambda to the upload function as a callback.
The lambda would then contain the code to execute when the upload task
completes:
```
fun uploadImage(image: Image) {
 showLoadingSpinner()
 uploadService.upload(image) { hideLoadingSpinner() }
}
```
Looking back at the very first snippet, not much has changed. You still show a
loading spinner, call upload and hide the spinner when the upload is done. The core
difference though is that you’re not calling hideLoadingSpinner right after the
upload. That function is now part of the `lambda block`, passed as a parameter to
upload, which you will execute at completion. Doing so, you can call the wrapped
function anytime you’re done with the connected task. And the lambda block can do
pretty much anything, not just hide a loading spinner.

In case some value is returned, it is passed down into the lambda block, so that you
can use it from within. Of course, the inner implementation of the uploadService
depends on the service and the library that you’re using. Generally, each library has
its own types of callbacks. However, even though callbacks are one of the most
popular ways to deal with asynchronicity, they have become notorious over the
years. You’ll see how in the next section.

## Indentation Hell
Callbacks are simpler than building your own mechanisms for thread
communication. Their syntax is also fairly readable, when it comes to simple
functions. However, it’s often the case that you have multiple function calls, which
need to be `connected or combined` somehow, mapping the results into more
complex objects.

In these cases, the code becomes extremely difficult to write, maintain and reason
about. Since you can’t return a value from a callback, but have to pass it down the
lambda block itself, you have to `nest callbacks`. It’s similar to nesting forEach or
map statements on collections, where each operation has its own lambda parameter.
When nesting callbacks, or lambdas, you get a large number of braces ’{}’, each
forming a `local scope`. This creates a structure called `indentation hell` — or
`callback hell` (when it’s specific to callbacks). A good example would be the fetching,
resizing and uploading images:
```
fun uploadImage(imagePath: String) {
 showLoadingSpinner()
 loadImage(imagePath) { image ->
   resizeImage(image) { resizedImage ->
     uploadImage(resizedImage) {
       hideLoadingSpinner()
      }
    }
  }
}
```

## A Blast From the Past
This is a book about `coroutines`. They’re a mechanism dating back to the 1960’s,
depicting a unique way of handling asynchronous programming. The concept
revolves around the use of `suspension points, suspendable functions` and
`continuations` as first-class citizens in a language.

They’re a bit abstract, so it’s better to show an example:
```
fun fetchUser(userId: String) {
 val user = userService.getUser(userId) // 1

 print("Fetching user") // 2
 print(user.name) // 3
 print("Fetched user") // 4
}
```
Using the above code snippet, and revisiting what you learned about blocking calls,
you’d say that the execution order was 1, 2, 3 and 4. If you carefully look at the
code, you realize that this is not the only possible logical sequence. For instance, the
order between 1 and 2 is not important, nor is the order between 3 and 4. What is
important is that the user data is fetched before it is displayed; 1 must happen before
3. You can also delay the fetching of the user data to a convenient time before the
user data is actually displayed. Managing these issues in a transparent way is the
black magic of `coroutines`!

They’re a part-thread, part-callback mechanism, which use the system’s power of
scheduling and suspending work. This way, you can immediately return a result from
a call *without* using callbacks, threads or streams. Think of it this way, once you start
a coroutine, or call a suspendable function, it gets nicely wrapped up and prepared
like a taco. But, until you want to eat the taco, the code inside might not get
executed.

## Explaining Coroutines: The Inner Work
It’s not really black magic — only a smart way of using low-level processing. getUser
is marked as a `suspendable function`, meaning the system prepares the call in the
background, and you get an unfinished, wrapped taco. But it might not execute the
function yet. The system moves it to a `thread pool`, where it waits for further
commands. Once you’re ready to eat the taco and you request the result, the program
can block until you get a ready-to-go snack, or suspend and wait for it within the
coroutine.

Knowing this, the program can skip over the rest of the function code, until it
reaches the first line of code on which it uses the user. This is called awaiting the
result. At that point, it executes getUser() and if it hasn’t already, suspends the
program.

This means you can do as much processing as you want, in between the call itself
and using its result. Because the compiler knows suspension points and suspendable
functions are asynchronous and treats their execution sequentially, you can write
understandable and clean code. This makes your code very extensible and easy to
maintain.

Since writing asynchronous code is so simple with coroutines, you can easily
combine multiple requests or transformations of data. No more staircases, strange
stream mapping to pass the data around, or complex operators to combine or
transform the result. All you need to do is mark functions as suspendable, and call
them in a coroutine block.

Another, extremely important thing to note about coroutines is that they’re not
threads. They are a low-level mechanism that utilizes `thread pools` to shuffle work
between multiple, existing threads. This allows you to create millions of coroutines,
without overflowing memory. A million threads would take so much memory, even
today’s state-of-the-art computers would crash.

## Key Points
- `Multithreading` allows you to run multiple tasks concurrently.
- `Asynchronous programming` is a common pattern for thread communication.
- There are different `mechanisms` for sharing data between threads, some of which
are queues and pipelines.
- Most mechanisms rely on a `push-pull`tactic, blocking threads when there is too
much, or not enough data, to process.
- `Callbacks` are a complex, hard-to-maintain and cognitive-load-heavy mechanism.
- It’s easy to reach `callback hell` when doing complex operations using callbacks.
- `Reactive extensions` provide clean solutions for data transformation,
combination and error handling.
- Rx can be too complex, and doesn’t fit all applications.
- `Coroutines` are an established, and reliable concept, based on low-level
scheduling.
- Too many `threads` can take up a lot of memory, ultimately crashing your program
or computer.
- `Coroutines` don’t always create new threads, they can reuse existing ones from
thread pools.
- It’s possible to have asynchronous code, written in a clean, `sequential` style, using
coroutines.
