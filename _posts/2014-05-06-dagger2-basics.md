---
layout: post
title: Dagger2 basics
---

# Basics

To proceed into the most simplified example of Dagger2 usage let's first go through some terms and explain them.

Here are some Dagger2 annotations:

@Inject - Usually it's said that Inject requests dependencies. But it actually has 2 purposes:
- requests a dependency to be injected
- makes class to be available for injection (if used on constructor)

**@Module** - Provides dependencies. By using @Provides annotation on a method you can have some specific logic for instantiating a dependency.

**@Component** - Ties together Models and Injects. But I don't really like this description of Components because it doesn't really explain anything, at least for me=) I will make a more detailed explanation later, but for now - this class has 2 main puposes:
- it can add injection support to system (eg. Android SDK) or any class that you don't own. It will allow you to inject something into other classes
- it lists which dependencies you want to be available for injection

I undestand that can be a bit too vague, but let's try to figure things out in example.

## Simpliest injection with Dagger2 on Android

Being an Android developer I will be using Android envoronment for examples. You can find plain Java examples in other articles, including Dagger2 docuemntation with *CoffeeMaker* sample.

Let's make a basic Android project with Dagger2 dependency. On how to setup Dagger2 please see [1st part](http://aaverin.github.io/2014/04/30/understanding-dagger2-for-android/).
You can follow my description in this github repository: [Dagger2 Step By Step repo](https://github.com/AAverin/dagger2_stepbystep). *master* brach has the basic project setup.

Now that we have a dummy activity, let's do some simple injection.

This class would be our parent class, that has a dependency.

```java
public class TestClassParent {

    private final TestClassDependency testClassDependency;

    @Inject
    public TestClassParent(TestClassDependency testClassDependency) {
        this.testClassDependency = testClassDependency;
    }

    public void call() {
        Log.d("TestClassParent", "call");
        if (testClassDependency != null) {
            testClassDependency.call();
        }
    }
}
```

This is our dependency:

```java
public class TestClassDependency {

    @Inject
    public TestClassDependency() {

    }

    public void call() {
        Log.e("TestClassDependency", "call");
    }
}
```

I also would like to inject our parent class into main activity

```java
public class MainActivity extends BaseActivity {

    @Inject TestClassParent testClassParent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        TestClassComponent testComponent = DaggerTestClassComponent.builder().build();
        testComponent.injectTo(this);

        testClassParent.call();
    }
}
```

And this small interface is what really makes things possible:

```java
@Singleton
@Component
public interface TestClassComponent {
    void injectTo(MainActivity mainActivity);
}
```

Check [this branch](https://github.com/AAverin/dagger2_stepbystep/commits/step1_basics) for details.
The diff of changes in my repo: [Simpliest injection](https://github.com/AAverin/dagger2_stepbystep/commit/834ced4304f05498d75f144fc8060de2575235d1)

Now, let's walk through changes quickly.
*TestClassParent* constructor has @Inject annotation. For Dagger2 that means that this class is now can be injected. Also, our consturctor has a parameter. That means that Dagger2 should inject this parameter into constructor when TestClassParent will be injected somewhere.

*TestClassDependency* also has @Inject in constructor, but no parameters. That means that it can be injected somewhere. Into our *TestClassParent*, for example.

*MainActivity* has some more significant changes. First - we have added ```@Inject TestClassParent testClassParent;``` - meaning we would like to inject a variable.
We have also added these 2 lines:

```java
TestClassComponent testComponent = DaggerTestClassComponent.builder().build();
testComponent.injectTo(this);
```

To explain their meaning let's see our *TestDaggerComponent*.
By definition, Component can either expose some class to be injected (more on this later), or, like in our case, make some other class available for injections.
In @Component interface, when you define a method with signature like:
```void myInjectMethod(MyClass myclass)```
it means that Dagger2 should make MyClass be available for injections.

Dagger does that by generating a new class for you. It will have a name of your Component prefixed with 'Dagger'. See *DaggerTestClassComponent*?
That newly generated class will have a builder() available, that you will need to use to instantiate a component.
And after that you will have to call ```myComponent.myInjectMethod(this)``` within the class that you make available for injection.

Sounds pretty complex at first. But if you run the code - you'll see that now both depdndencies were injected.

## Another approach

Let's re-work our simple example to show how can we get a dependency explicitly.
Here is the [diff](https://github.com/AAverin/dagger2_stepbystep/commit/7b8fcaecf3dc73d40054f1a54669280400a97d21). 
Or you can checkout [this branch](https://github.com/AAverin/dagger2_stepbystep/tree/step1_basics_2)

Here is how our TestClassComponent looks now:

```java
@Singleton
@Component
public interface TestClassComponent {
    TestClassDependency testClassDependency();
}
```

The idea is that we no longer make MainActivity available for injection, but instead we say that TestClassDependency is now available for injections in other places. And after instantiating the component we can call our method to get the actual dependency.

This is the simpliest possible usage of Dagger2, and I will explain some additional features of Components, Scopes and Modules in the next article.