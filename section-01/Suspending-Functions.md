## Suspending Functions

### Suspending vs. Non-Suspending
Up until now, you’ve learned that coroutines rely on the concept of `suspending
code` and `suspending functions`. Suspended code is based on the same concepts as
regular code, except the system has the ability to `pause` its execution and continue it
later on. But when you’re using two functions, a suspendable and a regular one, the
calls seem pretty much the same.

If you go a step further and duplicate a function you use, but add the suspend
modifier keyword at the start, you would have two functions with the same
parameters, but you’d have to wrap the suspendable function in a launch block. This
is how the Kotlin coroutines APIs are built, but the actual function call doesn’t
change.

The system differentiates these two functions by the suspend modifier at compile
time, but where and how do these functions work differently and how do both
functions work with respect to the suspension mechanism in Kotlin coroutines?
The answer can be found by analyzing the bytecode generated for each function ,
and by explaining how the call-stack works in both of these cases. You’ll start by
analyzing the non-suspendable variant first.

### Handling Happy and Unhappy Paths
When programming, you usually have something called a `happy path`. It’s the
course of action your program takes, when everything goes smoothly. Opposite of
that, you have an `unhappy path`, which is when things go wrong.

In the example above, if things went wrong, you wouldn’t have any way of handling
that case from within the callback. You’d either have to wrap the entire function call
in a `try/catch` block, or catch exceptions from within the thread function. The
former is a bit ugly, as you’d really want to have all possible paths handled at the
same place. The latter isn’t much better either, as all you can pass to the callback is a
value, so you’d have to either pass a nullable value, or an empty object, and go from
that.

To make this functionality available and a bit more clean, programmers define the
callback as a two-parameter lambda, with the first being the value, if there is any,
and the second being the error, if it occurred.
The signature of the function, and its callback, would be next, so replace
getUserFromNetworkCallback in Main.kt:
```
fun getUserFromNetworkCallback(
     userId: String,
     onUserResponse: (User?, Throwable?) -> Unit) {
 thread {
   try {
     Thread.sleep(1000)
     val user = User(userId, "Filip")
     onUserResponse(user, null)
 } catch (error: Throwable) {
    onUserResponse(null, error)
    }
  }
}
```
The callback can now accept either a value or an error. Whichever parameter or path
is taken, it should be valid, and non-null, while the remaining parameter will be
null, showing you that the path it governs hasn’t happened.

### Analyzing a Suspendable Function
The caveats found in the examples with callbacks are things which can be remedied
with the use of coroutines. Revise the changes you need to make to the example
above, to improve it even further:

- Remove the callback and implement the example with coroutines.
- Provide efficient error handling.
- Remove the new Thread allocation overhead.
  
To implement these improvements, you’ll learn another function from the
Coroutines API — suspendCoroutine. This function allows you to manually create a
coroutine and handle its control state and flow. This is unlike the launch block,
which just defined a way in which a coroutine was built, but took care of everything
behind the scenes.

### Living in the Stack
When a program first starts, its call-stack has only one entry — the initial function,
usually called `main`. This is because within it, no other functions have been called
yet. The initial function is important, because when the program reaches its end, it
calls back to the continuation of main, which completes the program, and notifies
the system to release it from memory

![image](https://github.com/oybekjon94/coroutines-book-notes/assets/91370134/72518c58-9371-43f1-9dc5-63bdf11d3ce6)


### Key Points
- Having callbacks as a means of providing result values can be pretty ugly and
cognitive-heavy.
- Coroutines and suspendable functions remove the need for callbacks and
excessive thread allocation.
- What separates a regular function from a suspendable one is the first-class
continuation support, which the Coroutine API uses internally.
- Continuations are already present in the system, and are used to handle function
lifecycle — returning the values, jumping to statements in code, and updating the
call-stack.
- You can think of continuations as low-level callbacks, which the system calls to
when it needs to navigate through the call-stack.
- Continuations always persist a batch of information about the context in which
the function is called — the parameters passed, call site and the return type.
- There are three main ways in which the continuation can resolve - in a happy path
returning a value the function is expected to return, throwing an exception in
case something goes bad, and blocking infinitely because of flawed business
logic.
- Utilizing the suspend modifier, and functions like launch and suspendCoroutine,
you can create your own API, which abstracts away the threading used for
executing code.
- withContext is a great way to bridge between the main and background thread
while still writing clean and sequential code.
