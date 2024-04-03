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

## 
