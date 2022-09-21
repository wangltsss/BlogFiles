---
title: Concurrent System Study Notes(Continuing...)
tags:
  - Concurrent System
  - C
  - Operating System
categories: Programming & Algorithms
date: 2022-01-26 21:37:00
---


# Signals

## Signal Lifespan

A **signal** is **generated** when a relevant event happens, and is **delivered** when the target process receves it. The lifespan of a signal is from its generation to its delivery.

<!--more-->

## Signal Block vs. Signal Ignore

When a process **blocks** a signal, the signal is pending to be unblocked. It is **not delivered** while being blocked.</br>

Whereas when a process **ignores** a signal, the signal **is delivered**, but the process does not respond to the signal.

## Signal Mask

**Signal mask** is a set of signals that will be blocked.</br>

A signal mask is of type `sigset_t`.</br>

To add/remove signals to/from a signal mask:

```c
sigset_t mymask;

sigemptyset(&mymask);			\\ Set the custom mask to contain no signals.
sigaddset(&mymask, SIGINT);		\\ Set the custom mask to contain SIGINT.
sigdelset(&mymask, SIGINT);		\\ Set the custom mask to delete SIGINT.
sigfillset(&mymask);			\\ Set the custom mask to contain all the signals.
```

Then, to make a signal mask effective, call the `sigprocmask()` function:</br>
(To learn details of `sigprocmask()`, refer to its [man page](https://man7.org/linux/man-pages/man2/sigprocmask.2.html#top_of_page).)

```c
sigprocmask(SIG_BLOCK, &mymask, NULL);		\\ Add signals in "mymask" to block list. 
```

Here, `mymask` is like a list that contains all the signals you want to block, it is not yet effective by simply add them to `mymask`. </br>
To make them effective, you have to call the `sigprocmask()` function to add them to an "invisible, unaccessible" list that actually controls the blockage of signals.

To remove a list of signals you do not wish to block anymore, put them into a mask, and call `sigprocmask()` with the first parameter being `SIG_UNBLOCK`:

```c
sigprocmask(SIG_UNBLOCK, &mymask, NULL);	\\ remove signals in "mymask" from block list.
```
Note that to remove a signal that is not in to block list is permissable.

The last value of `sigprocmask` is `SIG_SETMASK`, which functions like overwriting what was originally in the block list with the signals in your mask.

## Signal Disposition

Signal disposition is the behavior of the process that received the signal.

To create a custom signal disposition, use [`sigaction`](https://man7.org/linux/man-pages/man2/sigaction.2.html) struct.

```c
struct sigaction my_siga;

void siga_handler(){
	// ...
}

my_siga.sa_handler = &siga_handler();
sigfillset(&my_siga.sa_mask);
```

A `sigaction` struct looks something like this:

```c
struct sigaction {
  void     (*sa_handler)(int);  // assign to only one of 'sa_handler' and 'sa_sigaction'.
  void     (*sa_sigaction)(int, siginfo_t *, void *); // assign to only one of 'sa_handler' and 'sa_sigaction'. see man page.
  sigset_t   sa_mask;   // signals to be blocked while executing signal handler.
  int        sa_flags;    // modifies behavior of signals. see man page.
  void     (*sa_restorer)(void);  // see man page.
};
```

After customizing your signal disposition, let's make it the disposition of ***SIGALARM*** by calling [`sigaction()`](https://man7.org/linux/man-pages/man2/sigaction.2.html).

```c
sig_action(SIGALARM, &my_siga, NULL);
```
















