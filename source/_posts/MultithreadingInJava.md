---
title: Multithreading in Java
tags:
  - Java
  - Multithreading
categories: Programming & Algorithms
date: 2022-01-22 21:16:08
---


# Preface
I wanted to build a website of my own from scratch using Java recently, and so I found myself a hands-on tutorial. The very first step was to write small components called ***servlet***, which
required multithreading.
<!--more-->

# Multithreading Basics
In an OS, every application executing is usually called a ***process***, while each ***process*** can contain multiple ***threads***.
Take a typical music player as an example: a music player usually is capable of playing the music while showing the lyrics, and maybe show the album cover as well. Here, one thread will take care of playing the music, while another will take care of synchronizing the lyrics.
One process contains at least one thread, but it can contain multiple threads.

# Multithreding in Java

## Multithreading procedure

After creating a ***Thread*** instance, a new thread will wait for the```start()``` method to be called so that it will enter ***Ready*** state.</br>
While being in ***Ready*** state, a thread will stay in the ready queue, waiting to be called. </br>
When a thread in ***Ready*** state acquire CPU resources, it can be put to use by calling the ```run()``` method. In this procedure, the thread is in ***Running*** state.</br>
When a thread runs out of CPU time, or the ```wait()``` method is called, the thread will enter ***Blocked*** state. While in ***Blocked*** state, the thread will remain in this state unless it regains CPU resources.

## Creating a new ***Thread*** Instance


```java
public class ThreadRunnable extends Thread{
	/* override run() method */
	private void run(){
		/* */
	}
}
```
or

```java
public class ThreadRunnable implements Runnable{
	/* override run() method */
	private void run(){
		/* */
	}
}
```

## Synchronization between threads

### Atomic operation

Assignment like</br>
```java
n = n + 1;
```

seems to look like a single instrution, however, it consists of at least three instructions.

```
// erroneous Assembly.
ld sp(0), x5		// suppose stack pointer is pointing to n and load it into x5.
addi x5, x5, 1		// n = n + 1.
sd x5, sp(0)		// save n = n + 1 back.
```

Atomic operations are indivisible:

```java
int n;   //---  addi sp sp -8		make room on the stack for one int.
```

### Shared variable in Multithreading
Consider this: initialize ```n``` to 0, and create two threads. One thread, say thread1 will execute ```n += 1``` for 10000 times and the other thread, thread2 will execute ```n -= 1``` for 10000 times. Would you expect ```n == 0```? </br>
The fact is, the result might differ every time this snippet of codes is executed. This is because, very possibly, after thread1 add 1 to ```n``` 10000 times, and before it could store the result, thread2 takes over CPU resources and found that ```n``` is still 0.

### synchronized key word

```java
synchronized(lock object){
	// clauses
}
```
All clauses in side of these ```{```, ```}``` will be locked.</br>



