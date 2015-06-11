---
layout: post
comments: true
categories: [android, tutorials]
tags: [dagger2, di, dependency injection]
title: Dagger2 For Android - Part4 - Scopes
excerpt: Part4 of the article series focuses on the Scope concept, which took me quite some time to understand. Article will explain how Scopes are tied to the lifecycles and how you can use Scopes to build subgraphs and maintain dependencies in the project.
---

I took me a while to come to some understanding of the Scope concept in Dagger2. I couldn't really understand how @Activity scopes is ever related to real Activity lifecycle, and what's the whole thing about Components not being able to access another Scope.

But, finally, I got it. So, let's come up with an example if see if I can explain it to you too.

##Scopes in Dagger2

**@Scope** annotation is basically something that will help you define where your particular set of dependencies will be accessible.

I find the best explanation of **Scope** being written in the [Dagger2 Initial Proposal](https://docs.google.com/document/d/1fwg-NsMKYtYxeEWe82rISIHjNrtdqonfiHgp8-PQ7m8/edit#heading=h.pk0mr6enxl9x), which isn't a thing you find quickly while reading documentation.

As many things in Dagger2, Scopes surve multiple puprposes:
- they allow to make sure that dependencies are not accessible between the scopes, thus giving a better separation of dependencies in the project
- they provide sematical understanding for the developer and are used in internal static analysis during graph building process by Dagger 2
- they tie to the lifecycle of an object allowing to differentiate between the scopes of runtime

To make it easier to understand, let's first see an example of **@Singleton** scope definition and usage.

I already covered some injection basics in [part 2]({% post_url 2014-05-06-dagger2-for-android-part2-basics %}), and module usage cases in [part 3]({% post_url 2014-05-09-dagger2-for-android-Part3-Modules %})

Having all that, say we have an *ApplicationModule* that provides some complex dependencies, and an *ApplicationComponent* class that ties them together, and use a **@Singleton** annotation

```java
@Module
public class ApplicationModule {

    private final BaseContext baseContext;
    public ApplicationModule(BaseContext baseContext) {
        this.baseContext = baseContext;
    }

    @Provides @Singleton
    BaseContext baseContext() {
        return baseContext;
    }

    @Provides @Singleton
    Repository repository(StubRepositoryImpl repository) {
        return repository;
    }

    @Provides @Singleton
    AlarmManager alarmManager() {
        return (AlarmManager) baseContext.getSystemService(Context.ALARM_SERVICE);
    }
}

@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
    void injectTo(BaseActivity baseActivity);
}
```

Notice that now all methods in Module also have @Singleton annotation. If you will not do that you'll get a compilation exception, because Dagger2 controls that if Component has a **@Singleton** scope he should always inject only dependencies of **@Singleton** scope.

Also, when we will invoke **applicationComponent.injectTo(<BaseActivity instance>)**, Dagger2 will know that now our BaseActivity uses **@Singleton** scope, and all attempts to use any other scope except **@Singleton** inside our BaseActivity will also result in a compilation error.

That is how Dagger2 ties scope to the object.

Now, let's define a custom scope, name it **@ActivityScope**, and define Components and Modules for that scope.

```java
@Scope
@Retention(RUNTIME)
public @interface ActivityScope {}

...

@Module
public class ActivityModule {
    private final BaseActivity baseActivity;

    public ActivityModule(BaseActivity activity) {
        this.baseActivity = activity;
    }

    @Provides @ActivityScope
    BaseActivity activity() {
        return this.baseActivity;
    }
}

...

@ActivityScope
@Component(modules = ActivityModule.class)
public interface ActivityComponent extends ActivityInjectsTo {
    BaseActivity baseActivity();
}

public interface ActivityInjectsTo {
    void injectTo(LoginActivity authenticatorActivity);
}

...

public class LoginActivity extends BaseActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    
        getComponent().injectTo(this);
    }
}

...

public class BaseActivity {
    private ActivityComponent activityComponent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        baseContext = (BaseContext) getApplicationContext();
        baseContext.getApplicationComponent().injectTo(this);

        activityComponent = DaggerActivityComponent.builder()
                .applicationComponent(baseContext.getApplicationComponent())
                .activityModule(new ActivityModule(this))
                .build();
    }

    @Override
    public ActivityComponent getComponent() {
        return activityComponent;
    }
}

```

There's a lot of code so let's start from the bottom.

BaseActivity is parent for our LoginActivity. in *OnCreate* we inject ApplicationComponent dependencies into BaseActivity and create ActivityComponent for future reference.

In *LoginActivity* we inject *ActivityComponent* dependencies into our *LoginActivity*, and this also makes our *LoginActivity* of **@ActivityScope**, and will not be able to inject any variables of a **@Singleton** scope into it anymore.

Also, the moment we inject **@ActivityScope** into *LoginActivity* we say that now all objects of **@ActivitScope** are considered alive while *LoginActivity* is alive. Meaning that when our *LoginActivity* will get destroyed, all dependencies marked with @ActivityScope should be grabage collected by the GC.

All that is kinda cool, we now have separated our dependencies. But here's a small problem. Things like system classes and our helpers basically shouldn't be re-created every time when *LoginActivity* is created - it doesn't make much sense, it's better to have them on the @Singleton scope and keep them alive while our application is running. But this scope separation doesn't let us access **@Singleton** scope from **@ActivityScope**. How can we fix that?

Easily. We just need to say which of the **@Singleton** dependencies we would like our **@ActivityScope** to have access to. Let's update our code a little.

```java

@ActivityComponent
@Component(dependencies = ApplicationComponent.class,
        modules = ActivityModule.class)
public interface ActivityComponent extends ActivityInjectsTo {
    BaseActivity baseActivity();
}

...

@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
    void injectTo(BaseActivity baseActivity);

    AlarmManager alarmManager();
    Repository repository();
}

```

So, here's what we do. We say that our *ActivityComponent* now depends on *ApplicationComponent*. And in our *ApplicationComponent* was explicitly say which of the dependencies should available in other scopes.

```java
    AlarmManager alarmManager();
    Repository repository();
```

That's it, basically.

Unfortunately, I don't have a repository branch for this article, so please shoot any questions in the comments if you have any.


