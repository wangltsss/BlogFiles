---
title: Java多线程的学习笔记(Multithreading in Java)(未完)
date: 2022-01-22 21:16:08
categories: Programming & Algorithms
tags:
- Java
- Multithreading
---

# 写在前边
前一阵子想用Java从头开始写一个自己的网站，于是找了一个教程。教程的第一步就是入门servlet
，自己了解了一下发现需要熟悉多线程。正好这学期有一门课是以C语言为基础讲并发系统的，干脆自己在研究一下Java多线程。

<!--more-->

# 多线程基础
在计算机中，通常运行的每个应用被叫做一个进程(Process)。而每个进程可以包含多个子进程，叫做线程(Thread)。例如在音乐播放器中，播放音乐需要一个线程，显示歌词又需要一个线程。一个进程至少包含一个线程，也可以包含多个线程。

# Java的多线程

## 多线程流程

在创建Thread实例后，线程会等待```start()```的执行以进入就绪状态(Ready)。</br>
进入就绪状态后，该线程会处于就绪队列中，等待被调度。 </br>
当该就绪状态的线程获取CPU资源后，就可以调用```run()```方法，此时该线程处于运行状态(Running)。</br>
当一个线程占用资源时间耗尽，或者主动执行```wait()```等方法后，就会进入阻塞状态(Blocked)。处于阻塞状态的线程再重新获取CPU资源之前，会一直处于阻塞状态。

## 新建```Thread```实例

可以通过

```java
public class ThreadRunnable extends Thread{
	/* override run() method */
	private void run(){
		/* */
	}
}
```
或者

```java
public class ThreadRunnable implements Runnable{
	/* override run() method */
	private void run(){
		/* */
	}
}
```

## 线程的同步
### 原子(atomic)操作
语句如</br>

```java
n = n + 1;
```

看似为单个语句，但其实其中至少包含了三步:

```
// 不标准的Assembly.
ld sp(0), x5		// suppose stack pointer is pointing to n and load it into x5.
addi x5, x5, 1		// n = n + 1.
sd x5, sp(0)		// save n = n + 1 back.
```

原子操作为不可被分割的指令，如:

```java
int n;   //---  addi sp sp -8		make room on the stack for one int.
```

### 多线程中共享变量(shared variable)的问题
如果将```n```初始化为0，同时创建两个线程，一个线程（线程1）负责执行10000次```n += 1```，另一个（线程2）负责执行10000次```n -= 1```，最后```n```的值每次都会不同。</br>
这是因为，非常可能地，在线程1读取到```n = 0```后，线程2立刻抢占了CPU资源，并且同样读取到了```n = 0```（因为线程1还来不及对```n```做修改），并且执行了```n -= 1```并存储回```n```。此时线程1重新获得资源，但此时线程1已经进行过读取操作（```n = 0```），所以最后经过```n += 1```和```n -= 1```后，```n```此时的值为1.

### synchronized关键字

```java
synchronized(lock object){
	// clauses
}
```
```{```会自动为```synchronized```内部的语句加锁，而```}```会自动解锁。</br>
注意，```lock```实例与```synchronized```内部的语句无关，该lock实例可以为任何对象的实例。
在```synchronized```结构中归属于同一个```lock```实例的语句不能被同时执行。



































