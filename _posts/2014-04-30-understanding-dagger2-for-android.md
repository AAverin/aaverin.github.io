---
layout: post
title: Undersanding Dagger2 for Android
---

**Why**
I've been working on this new architecture for large-scaled Android applications, and the obvious choice to make sure everthing is modular and fully testable was to pick some Dependncy Injection framework.
I've got no prior experience with them, and with Dagger2 being just released to the public I decided to just get the latest popular framework available and use it.
But here's the thing. Even though there are several really good articles over the net about Dagger2, and there are talks by *Jake Wharthon* and *Greg Kick* - I took me quite some time to wrap my head around the whole concept and understand which part goes where and how it works with Dagger2. And I decided to write everything down so I wouldn't forget it myself and maybe it will help to understand it better to someone else.

**Dependency Injection and Dagger2**

Basic idea of DI is to be able to separate different classes in the architecture - it let's you to write modular code, re-use components, solve multiple common problmes and, most importantly, - makes your architecture testable.
Everything is quite straightforward. Behind the fancy name of "Dependency Injection" hides simple idea to pass everything that your class depends on via a constructor. Simple. But things become very difficult when you have many dependencies, and when you have to manage all of the factories that call your constructors in the particular order to satisfy every dependency.
This is where DI frameworks help - Spring, Guice, RoboGuice, Dagger, Dagger2.
Dagger2 at the moment is most advanced and fast.

**Setup**

To make Dagger2 running for your project you need to do following steps:
* add ```classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'``` to depenencies in your main build.gradle
```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.2'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}
```
* add ```apply plugin: 'com.neenbedankt.android-apt'``` to your module build.gradle
* add Dagger2 and Java Annotations dependencies to your module:
```
apt         "com.google.dagger:dagger-compiler:2.0"
compile     "com.google.dagger:dagger:2.0"
provided    "org.glassfish:javax.annotation:10.0-b28"
```

[2nd part - Dagger2 Basics](http://aaverin.github.io/2014/05/06/dagger2-basics/)

**Links**
 * [Jake Wharthon's talk on Devoxx2013 about Dagger1](https://www.parleys.com/tutorial/architecting-android-applications-dagger)
 * [Jake Wharthon's talk on Devoxx2014 about Dagger2](https://www.parleys.com/tutorial/the-future-dependency-injection-dagger-2)
 * [Greg Kick's initial introduction to Dagger2](https://www.youtube.com/watch?v=oK_XtfXPkqw)
 * [Fernando Cejas - Tasting Dagger2 on Android](http://fernandocejas.com/2015/04/11/tasting-dagger-2-on-android/)
 * [Konstantin Mikheev - Snorkeling with Dagger2](http://konmik.github.io/snorkeling-with-dagger-2.html)

**Useful Github repos**
* [Android CleanArchitecture @android10](https://github.com/android10/Android-CleanArchitecture)
* [U2020 port to MVP + Dagger2 @LiveTyping](https://github.com/LiveTyping/u2020-mvp)
* [Dagger2 Example @mgrzechocinsky](https://github.com/mgrzechocinski/dagger2-example)
