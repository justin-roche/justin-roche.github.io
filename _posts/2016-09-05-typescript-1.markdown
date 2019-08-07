---
layout: post
title:  "Monads part III"
date:   2016-09-10 19:45:31 +0530
categories: Javascript
---
The Excellence of Typescript
=
You thought monads were over? You were wrong. Rule number one of monads is that monads are never done. This post will look at how Typescript, the Microsoft-created superset of Javascript, is not only completely awesome but can allow you to write statically-typed code that compiles into regular Javascript. This pre-compiled code is perfect for anything that requires us to carefully look at types. 

To get a better grip on monads, we'll be taking a look at the tsMonad library, available [here](https://github.com/cbowdon/TsMonad). It's a monad library written in Typescript, which will allow for a comparison between the final javascript and the algebraic representation of type conversions represented in Typescript. Particularly, we'll look at its implementation of the maybe monad, which was investigated in the last post. Hopefully we'll get an abstract understanding of what monads are, so that we can finally give the box model the old heave-ho. 

Getting Started with Typescript
=
A good guide for setting up typescript on a mac can be found [here](http://amanvirk.me/setting-up-typescript-for-sublime-text-on-mac/). Following this guide, you'll be able to build from typescript files to javascript files in sublime text, without ever touching the console. Best of all, this build includes ES6 transpilation. 

Next, download the tsMonad library, npm install, and you'll see a number of tests have just passed. That's the monads working. First, let's see how the monad system implemented in part II would be typed using Typescript. 

Typing Unit/Lift/Bind
=

The unit function from part II might look like this in typescript: 

{% highlight javascript %}
    function unit<U>(t: U): Nothing|Just<U>;
{% endhighlight %}

What we have here is a generic function. The generic function takes an argument, of a type called U. This U binds a type (not the value of the variable). So if at compile time the U binds to a string type, all other references to the U must bind to a string type. The argument to the unit function is marked with parenthesis: it is a variable named t (this naming only holds for the type declaration of the function, not for the function definition itself), of type U.

The return type is represented after a colon. It has two possibilities, Nothing and Just, seperated by a pipe. The type of the Just must match the type that was input (both have U in angle brackets). 

All of this type checking is done at transpile time. 

Lift and bind are similar, except that they return functions:

{% highlight javascript %}
function lift<U,T>(f: (t: T) => U): ((t: T)=>Nothing|Just<U>);
function bind<U,T>(f: (t: T) => Nothing|Just<U>): (t: Nothing|Just<U>) => Nothing|Just<U>;
{% endhighlight  %}

So with lift, a function with an argument of type T (which produces a result of U) is the argument, and the result is a function taking an argument of type T and returning an argument of Nothing or Just<U>.

The Unit Function
=
tsMonad is little more complicated than our simple implementation, because they are implementing a number of different monads. So the first thing they did was build a common monad interface. In the file lib/src/maybe.d.ts we can now find a type definition for what makes a monad interface. 

What is an interface you ask? It's like a class. An interface is a "structural type". For example, if there are two classes that can be enumerated with an "of" statement, these implement the same enumerable interface. A monad class that implements the monad interface must include the functions defined on the interface. The monad interface, for example, defines a unit function, which every monad instance must have: 

{% highlight javascript %}

export interface Monad<T> {
    unit<U>(t: U): Monad<U>;
}

{% endhighlight %}

Every class or instance that implements the monad interface must also implement a unit function. And that unit function has some type constraints. It takes some parameter t, of type U. It also returns an object which also has the Monad interface, of type U. 

Note that all this is what we would expect from what is already known about the unit function: it takes an element and wraps it in some new context. However, the difference between this and the implementation of unit above is that in this case, the nothingness or justness of the monad must be stored in some state on the monad, rather than marking them just in different classes.  

To see how Maybe implements this interface we can look in the maybe.d.ts file. Simplifying from the code there, here's the key snippet:

{% highlight javascript %}

export declare class Maybe<T> implements Monad<T> {
  unit<U>(u: U): Maybe<U>;
}

{% endhighlight %}

This shows that the Maybe class is a class which implements Monad. If for some reason we forgot our obligatory unit function in the Maybe class, the typescript transpiler would remind us, because to implement a Monad interface a class needs a unit function. 

With unit as it is, we see that it takes a value of type U and returns a Maybe of type U. Since Maybe implements the monad interface, this more specific type declaration for the unit function passes the more general type declaration of the unit function found in Monad. 

How does this look in the final javascript? 

{% highlight javascript %}

Maybe.prototype.unit = function (u) {
    return Maybe.maybe(u);
};

 Maybe.maybe = function (t) {
        return t === null || t === undefined ?
            new Maybe(MaybeType.Nothing) :
            new Maybe(MaybeType.Just, t);
};

  function Maybe(type, value) {
        this.type = type;
        this.value = value;
}
       
{% endhighlight %}

This implementation of the maybe constructor is a lot like what we saw before. Again, here the type is purely marked with state on the object. So a maybe with "just" as its type holds a value, and a maybe with "nothing" as its type holds nothing, and these types (which are implemented elsewhere as objects) are passed as arguments to the constructor. 

It's noteable that what is being done here-- with the type as argument or, in our previous implementation, as tested against instance of, is dynamic type checking done at runtime, and has nothing to do with typescript's type checking. 

The Lift Function
=

From the previous discussion of lift, we know it takes some function in one type and makes it return that value in the monad wrapper. What does that process look like as a type declaration in tsMonad? 

{% highlight javascript %}

lift<U>(f: (t: T) => U): Functor<U>;

lift:<U>(f: (t: T) => U) => Maybe<U>;

{% endhighlight %}

What the heck is a functor? It's basically another type in functional programming, which implements a function called fmap. Fmap is a function that can do the following: 

{% highlight javascript %}

fmap<U>(Functor<T>, (f: (t: T) => U)): Functor<U>;

{% endhighlight %}

So it takes a functor in T, runs a function from T to U on it, and returns a Functor in U. Javascript's array function map is a good example of it. Consider: 

{% highlight javascript %}

    let a = ['z','q'];
    let b = a.map((t)={return t.charCodeAt(0);});

{% endhighlight %}

Here the array is a functor. The functor a is of type string. The map function works on that array and a function and returns a new array of a different type (integer).

Maybe implements not only the Monad interface, but also Functor. Here lift is declared in the Functor interface. Therefore it implements lift as defined for functors. Since lift just takes a function that has nothing to do with a monad and makes it return a monad, it can be defined as taking a function from T to U and returning a Maybe of U. 

But now there's a big difference between what we made in part II and how this monad is implemented. In part II lift and bind returned functions that were bound or lifted. This monad is only returning the monad. The lifted function is executed when lift is called. Previously a lifted function could be passed around. Here it is called on the monad and the monad executes it. So the meaning of lift is quite different. 

The Bind Function
=

The bind function has the same differences. It takes an already lifted function, and allows it to accept arguments of the monad type. But rather than doing so explicitly it does it implicitly, and immediately returns the result: 

{% highlight javascript %}

    bind<U>(f: (t: T) => Monad<U>): Monad<U>;

    bind<U>(f: (t: T) => Maybe<U>): Maybe<U>;

    Maybe.prototype.bind = function (f) {
        return this.type === MaybeType.Just ?
            f(this.value) :
            Maybe.nothing();
    };

{% endhighlight %}

This bind function is called on a monad object and unwraps its value during execution. 

So, which way is right? Probably this way. If you want to talk about monads and be understood, it is certainly a good idea to mean a monad class that implements type changes in this way. In part II there was no "Monad" class properly speaking. Here there is a monad class, allowing it to be chained together, and lift and bind are very specific operations that the monad class implements. 

However we did see that unit, lift, and bind can pass all of the monad laws using the approach of a distributed monad system. They can be chained together anywhere using a do command. 






