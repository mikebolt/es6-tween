# ES6 Tween API draft

## `Tween` class

Constructor:

`new Tween(tweenedObject, endValues, duration)`

> I don't think it makes sense to allow a Tween to be constructed without
> the ending values or the duration. The existing API allows for tweens to be
> constructed and then left in an invalid state if the `.to()` method is not
> called immediately after construction. In order to keep the nice chaining
> syntax, we could expose a method on the Tween class itself, such as
> `Tween.object(a).to(b, c)`, or `Tween.object(a).to(b).for(c)`, where c is
> the duration. `Tween.object(a)` wouldn't return a Tween instance, so it would
> cause an error if a user tried to call `.start()` on it.

### `Tween` Methods

* `start(time)`

* `stop()`

* `end()`

* `then(nextTween)` or `chain(nextTween)` or `compose(nextTween)`

  * Will return a new tween-like object instead of modifying the
    existing tween.

* `update(time)`

* `reverse()`

  * Returns a new, reversed version of this tween.



## `Easing` class

`Easing` should be an abstract class. All easing "functions" should be
subclasses of the `Easing` class. These subclasses would only need to
implement the single function `evaluate(x)`.

Each subclass of `Easing` would benefit from some extra methods defined on
the prototype:

### `Easing` Methods

* `compose(otherEasingFunction)`

* `compose(function f(x) { return ... })`

* `reverse()`

* `invert()`

* `add(otherEasingFunction)`

* `multiply(otherEasingFunction)`

## `ProxyObject` class

Sometimes you need to tween only certain properties of tweens, and if other
properties are tweened on accident, the results could be disastrous.
This happens with Three.js's Vector3 class, for example. If you make a tween
such as `new Tween(vectorA, vectorB, 1000)`, then strange things may happen
because properties such as rotation are tweened in addition to position.
The idea behind the `ProxyObject` class is to wrap either a tweened object or
an end values object in order to only expose the properties that should be
tweened. For example:

    new Tween(new ProxyObject(vectorA, ['x', 'y', 'z']), vectorB, 1000)

However, you could just make a simple function such as the following:

    function makeVectorTween(vectorA, vectorB, duration) {
        var endValues = {
	    x: vectorB.x,
	    y: vectorB.y,
	    z: vectorB.z
	};
        return new Tween(vectorA, endValues, duration);
    }

So it's arguable whether this is necessary.

## `TweenSequence` class

Represents a sequence of tweens. This should be the primary method of chaining
tweens together. The constructor will accept an array, or several tweens as
separate arguments.

    new TweenSequence([tweenA, tweenB, tweenC, tweenD, tweenE]);
    new TweenSequence(tweenA, tweenB, tweenC);

A TweenSequence could have every method that `Tween` has (it could be a
subclass). A TweenSequence can even sequence other TweenSequences together:

    new TweenSequence(tweenSequenceA, tweenSequenceB);

This would be equivalent to creating a single TweenSequence with all of the
sequenced tweens in tweenSequenceA followed by all the tweens in
tweenSequenceB.

### `TweenSequence` methods

* onSequencedTweenFinished(callback)

  * Sequences could get very nested. Should this only call if a "direct"
    sub-sequence is completed, or should it call if any Tween in this
    sequence is completed?


## `TweenGroup` class

This class simply provides proxy versions of most of Tween's methods, that
operate on an entire group of tweens instead of only one. It will be very
similar to the existing `TWEEN` global object in Tween.js v1.

### `TweenGroup` methods

* `addTween(tween)` or just `add(tween)`

* `removeTween(tween)`

* lots of proxy methods


## `DynamicTween` class

Can simply extend `Tween` and change the update method so that the end values
object is re-examined during each update step, rather than simply cloned
during initialization. `DynamicTween` is slightly less performant and
it may behave differently.

## `RepeatedTween` class

TODO
