---
title: "Just One More Bool Bro"
published: false
---

# The Problem

```cpp
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
        bool cache_results = true,
        bool try_new_method = false);
private:
    // member variables
    // more functions
    // et cetera 
};
```

## How Did It Get Like This?

A long time ago, `Widget` was simple. It did one job and did it well. It had a class declaration that looked like this:

```cpp
class Widget{
public:
    Widget();
private:
    // ...
};
```

Then, we wanted to see what would happen if `Widget` did one thing differently, say, stopped applying paste. 
We didn't want to break existing code, so we added a default argument.

```cpp
class Widget{
public:
    Widget(bool apply_paste = true);
private:
    // ...
};
```

That way, all the existing code can continue to call `const auto widget = Widget()` without error, and our new code could call `const auto dryWidget Widget(false)`.
Allowing old code to continue working is a **GOOD THING**.

### Expressing Multiple Options

```cpp
class Widget{
public:
    Widget(
        bool apply_paste = true,
        bool new_calculation = false);
private:
    // ...
};
```

```cpp
class Widget{
public:
    enum class PasteApplication{Absent, Present};
    enum class CalculationMethod{Old, New};
    Widget(PasteApplication paste, CalculationMethod method);
    Widget(PasteApplication paste);
    Widget(CalculationMethod method);
    Widget();
private:
    // ...
};
```



# The Solution

Use [designated initializers](https://en.cppreference.com/w/cpp/language/aggregate_initialization.html#Designated_initializers)!


```cpp
class Widget{
public:
    ConstructionFlags{
        bool apply_paste = true;
        bool new_calculation = false;
        bool defeat_evil = false;
        bool invert = true;
        bool laminate = false;
        bool cache_results = true;
        bool try_new_method = false;
    }
    Widget(const ConstructionFlags& flags={});
private:
    // ...
};

// The oldest code can still call the default constructor
const auto oldWidget = Widget();

// The first addition would construct a Widget like this
const auto dryWidget = Widget({.apply_past=false});

// The most-recent experimenter can construct with try_new_method
// and they don't have to know what all the other flags do!
const auto newWidget = Widget({.try_new_method=false});

// And the user can construct with any combination of flags, old and new!
const auto spicyWidget = Widget({.defeat_evil=false, .laminate=true});

```