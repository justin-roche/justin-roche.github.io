---
layout: post
title:  "The Most Important Diagram in Javascript"
date:   2016-09-10 19:45:31 +0530
categories: Javascript
---
The Most Important Diagram in Javascript
=
Perhaps that is an overstatement. Maybe there is a more important diagram than the one you are about to see. But diagrams are good for reducing cognitive load, and the element of vanilla javascript that increases cognitive load the most is the area where there have been design mistakes: I am talking about the prototypal system. 

The reason for the confusion is that the word "prototype" applies to something which is a prototype in two senses. Because of the semantic ambiguity, we have to look very closely at what prototype means in Javascript and how it doesn't mean what it could be expected to mean.

Prototypes as Delegation
=
When object A is a prototype of another object B, in the simplest sense it means that failed lookups on B will be delegated to object A. If B does not have a property, the lookup will continue on to its parent, and so on up the prototype chain, until there is an object with no parent or until the lookup is resolved. 

From this it follows that there should have been a property on each object called "prototype", which pointed to that objects prototype, e.g. the object that it delegates its lookups to. This would have been the simplest way. But alas, it didn't happen that way at all. 

The object to which objects delegate their lookups is actually called \__proto\__. This is a private property as designated by the underscores, or is intended to be private, and it is not something you should be assigning to or modifying directly. 

So this is the first wrinkle, the __delegation prototype__ is referred to as __proto__. 

Prototypes as Constructor Methods
=   
Perhaps proto would have been fine, if no one had ever thought to mask Javascript's prototype system in pseudoclassical syntax, in order to mimic other object-oriented languages. In such a world a constructor might look like this:

{% highlight javascript %}

Function Car(driver){
	var obj = Object.create(Car.methods);
	obj.driver = driver; 
	return obj; 
} 

{% endhighlight %}

Here we would just be creating an object that uses another object as its prototype (Object.create(Car.methods) returns such a prototyped object). We create an instance variable "driver", and then return the newly created object. 

Note the clean semantics. The Car.methods contains all the methods that are intended to serve as the prototype. These methods serve as the prototype of the newly minted object. No confusion here. 

Construction Mode in Pseudoclassical Style
=
In pseudoclassical style, the Car function above is not called with a normal function invocation. Because the keyword "new" is used it executes the constructor in such a way that it carries out certain implicit operations without them being specified.

The operations that are carried out at runtime because of the keyword new are commented out below:

{% highlight javascript %}

Function Car(driver){
	//var obj = Object.create(Car.methods);
	//this = obj
	this.driver = driver; 
	//return obj; 
} 

Car.methods = {
	move: function(){this.x += 10;}
}
{% endhighlight %}

So construction mode creates a new object, assigns its prototype, binds the keyword this to the new object, and returns that object. 

But what happened to the Car.methods?

Prototypes in Pseudoclassical Style
=
Maybe because of inattention to the confusing semantics, and because of blending pseudoclassical style with prototypal inheritance, the Car.methods object, which contains everything that will be the prototype, is actually called by the property name "Car.prototype":

{% highlight javascript %}

Function Car(driver){
	//var obj = Object.create(Car.prototype);
	//this = obj
	this.driver = driver; 
	//return obj; 
} 

Car.prototype = {
	move: function(){this.x += 10;}
}
{% endhighlight %}

If we try to understand in what way this Car.prototype is a prototype we find that it is a prototype in one sense, because it is the prototype of the class, but in another sense it is not the prototype because no objects delegate to it by name, instead they delegate to their proto object, which was assigned to be Car.prototype with Object.create. 

Because of this confusing semantics, one word means two very different things: there is both a __construction prototype__, associated with the class, and a __delegation protototype__, associated with the objects. It's important to keep these separated at all times, and that's where the most important diagram comes in. 

The Most Important Diagram
=
The moment you've all been waiting for. It's the picture that must be returned to again and again, not because it makes sense, but because it doesn't make sense: 

<img src="{{ site.url }}/images/proto2.png" class="center" />

Let's call it the triangle of prototypes. But also of proto. Which is a prototype. Anyway, what it means is that there are three objects-- the function constructor, the .prototype object associated with that constructor, and the instantiated object created by the constructor. 

The relationship between the object to its delegation prototype is one of delegation. Failed lookups on the instantiated object go to its \__proto__ object. The relationship of the function to that same object is one of access during construction mode. The functions .prototype property leads to this same object. That object's .constructor property points the constructor. 

The instanceof operator, not depicted, goes from the instance to the \__proto__ and then to the constructor, in order to verify whether or not an object was constructed by that constructor. It is because of this indirect access to the constructor that reassignment of .prototype on a constructor makes the instantiated objects of that constructor forget where they came from. 

Conclusion
=
I think the most important diagram provides a solid mental model for understanding Javascript prototypes. There's much more too it, especially when it comes to subclassing and inheritance, but the most important diagram provides the basis for navigating through those. 









