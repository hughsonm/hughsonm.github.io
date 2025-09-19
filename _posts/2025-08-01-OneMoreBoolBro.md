---
title: "Just One More Bool Bro"
published: false
---

# TL;DR

If your class has a contructor that takes a bunch of defaultable flags and settings and you have access to C++20, then you should put those flags in a struct.
Your code will be easier to read and easier to write.


```cpp
// In Widget.h
class Widget{
public:
    Widget(
        bool apply_paste = true,
        bool new_calculation = false,
        bool defeat_evil = false,
        bool invert = true,
        bool laminate = false,
        bool cache_results = true,
        bool try_new_method = false);
private:
    // ...
};
```

The class `Widget` is better expressed like this:

```cpp
// In Widget.h
class Widget{
public:
    struct Flags{
        bool apply_paste = true;
        bool new_calculation = false;
        bool defeat_evil = false;
        bool invert = true;
        bool laminate = false;
        bool cache_results = true;
    }
    Widget(const Flags& options);
private:
    // ...
};
```

# The Setup

Suppose I want to add a feature to `Widget`. I read some code that uses `Widget`, and you read `Widget`'s header, and they look something like this: 

```cpp
// In SomeOtherModule.cpp
const auto myWidget = Widget(false, true, true, false, true, false);
```

```cpp
// In Widget.h
class Widget{
public:
    Widget(
        bool apply_paste = true,
        bool new_calculation = false,
        bool defeat_evil = false,
        bool invert = true,
        bool laminate = false,
        bool cache_results = true);
private:
    // ...
};
```

I want to make build `Widget`s that use the new feature, so I add an argument to that list of arguments, and I set the default to `false`, so I don't break any existing code.

```cpp
// In Widget.h
class Widget{
public:
    Widget(
        bool apply_paste = true,
        bool new_calculation = false,
        bool defeat_evil = false,
        bool invert = true,
        bool laminate = false,
        bool cache_results = true,
        bool use_new_feature = false); // <-- See the new argument
private:
    // ...
};
```

Then, I write some test code that uses the feature.

```cpp
// In TestNewFeature.cpp
const auto myWidgetWithNewFeature = Widget(false, true, true, false, true, false, true);
//                                                                                  ^
//                                                                                  |
//                Setting use_new_feature = true, instead of the default (false). --+ 
```

Notice a few things at this point:

1. ⚠️ I had to pass in values for the first six constructor arguments just so I could set the seventh to `true`. Are those really the values I want to test, or would I be better off just using the defaults? Are those the defaults? Did I copy-paste the defaults correctly?
1. ⚠️ Can I remember what the first argument means just by looking at the constructor call? If I'm **really** familiar with this code, then I might remember. If I'm new to this code, then I probably don't remember. If only we had named arguments in C++, a la Python! 🤔

# The Solution

I should use [designated initializers](https://en.cppreference.com/w/cpp/language/aggregate_initialization.html#Designated_initializers)!


```cpp
// In Widget.h
class Widget{
public:
    struct Flags{
        bool apply_paste = true;
        bool new_calculation = false;
        bool defeat_evil = false;
        bool invert = true;
        bool laminate = false;
        bool cache_results = true;
        bool use_new_feature = false;
    }
    Widget(const Flags& options);
private:
    // ...
};
```

Then, I use `Widget` like this:

```cpp
// In TestNewFeature.cpp
const auto myWidgetWithNewFeature = Widget(Widget::Flags{.use_new_feature=true});
```

This solves both of the problems mentioned above:

1. I can specify a value for just the flag I care about, without copy-pasting all the defaults for the other flags.
1. I can clearly see that `true` applies to `use_new_feature`, without jumping back to the header.

# How To Not Break Existing Code

1. Write the new struct-based constructor, and leave the old constructor(s) in place so that old code can still use it.
1. Deprecate the old constructor(s), with a helpful message that makes it easy to upgrade.

```cpp
// In Widget.h
class Widget{
public:
    struct Flags{
        bool apply_paste = true;
        bool new_calculation = false;
        bool defeat_evil = false;
        bool invert = true;
        bool laminate = false;
        bool cache_results = true;
        bool use_new_feature = false;
    }
    
    // This constructor allows setting flags indepently of each other
    // Example: set the value of defeat_evil, while leaving the other flags as defaults
    //  Widget(Widget::Flags{.defeat_evil=true});
    Widget(const Flags& options);
    
    [[deprecated("This constructor is confusing because it uses too many flags. Use the other constructor")]]
    Widget(
        bool apply_paste = true,
        bool new_calculation = false,
        bool defeat_evil = false,
        bool invert = true,
        bool laminate = false,
        bool cache_results = true,
        bool try_new_method = false);
private:
    // ...
};
```