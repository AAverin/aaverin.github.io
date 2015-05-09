---
layout: post
comments: true
categories: [android, tutorials]
tags: [dagger2, di, dependency injection]
title: Dagger2 For Android - Part3 - Modules
excerpt: Part3 of Dagger For Android series of articles. Gives a pretty straighforward and simple explanation of how Modules work.
---

In the [last article]({% post_url 2014-05-06-dagger2-for-android-part2-basics %}) we have implemented 2 very simple examples that were injecting our custom classes. Now let's try doing something more elaborate and Android-related.

In Android development one of very common cases of Dependency Injection would be to inject something from Android SDK into our Activity or Fragment. Something like SharedPreferences, for example.

As you may have noticed from previous articles, to make an object become available for injection we need to annotate it's constructor with *@Inject* annotation. But we do not own the constructor of SharedPreferences, it's a System class.

In that case we will have to define a Module.

Main purpose of a Module is to provide dependencies that we:
- can not annotate with @Inject in constructor
- require some additional logic during instance creation
- are interfaces (when coding against interfaces, for example)

A Module that would provide SharedPreferences may look like this:

```java
@Module
public class SDKClassesModule {

    private final BaseApplication context;
    public SDKClassesModule(BaseApplication context) {
        this.context = context;
    }

    @Provides
    SharedPreferences provideSharedPreferences() {
        return context.getSharedPreferences("my_shared_preferences.pref", Activity.MODE_PRIVATE);
    }

}

```

We make a method that would return out object properly instantiated, and annotate it with **@Provides**.

*Modules* do not work alone - they require a *Component*. Our *Component* would look like this to make it work:

```java
@Component(modules = SDKClassesModule.class)
public interface SDKClassesComponent {

    void injectTo(MainActivity mainActivity);
}
```

Notice the *modules* parameter - it lists all modules that this *Component* will be using. So basically *Component* will be injecting:
- instances that are provided by the module
- instances that are available through constructor annotations
That way you can have some custom logic of instantiating an object in Modules, and custom clases with constructor available will be available for injection automatically.

One more important thing to notice is that now that you have a *Module* you need to explicitly set it in the Component during instantiation.

So *MainActivity* will now have these lines:

```java
SDKClassesComponent sdkClassesComponent = DaggerSDKClassesComponent.builder()
                .sDKClassesModule(new SDKClassesModule(baseApplication))
                .build();
        sdkClassesComponent.injectTo(this);
```

You can find the whole stage in [this branch](https://github.com/AAverin/dagger2_stepbystep/tree/step2_modules)





