
# 🎭 MultiTween documentation

**Important Notice**

MultiTween has been superseded by [**Timeline Tween**](timeline_tween.md). 

**It's not recommended** to start new animations with MultiTween.

---------

MultiTween is an `Animateable` that animates multiple properties at once.

## Basic usage pattern

Create an `enum` outside your widget class. It contains all properties you want to animate. *(For example: width, height or color)*
```dart
enum AniProps { width, height }
```

Then you create a `MultiTween` by instancing it with the `enum` you created in the first step.
Invoke the `add` method to arrange your animation.

```dart
final _tween = MultiTween<AniProps>()
  ..add(AniProps.width, 0.0.tweenTo(100.0), 1000.milliseconds)
  ..add(AniProps.width, 100.0.tweenTo(200.0), 500.milliseconds)
  ..add(AniProps.height, 0.0.tweenTo(200.0), 2500.milliseconds);
```

Use the created `_tween` with your favorite animation technique. Here we use the `PlayAnimation` widget provided by **Simple Animations**.

```dart
PlayAnimation<MultiTweenValues<AniProps>>(
  tween: _tween, // Pass in tween
  duration: _tween.duration, // Pass in total duration obtained from MultiTween
  builder: (context, child, value) {
    return Container(
      width: value.get(AniProps.width), // Get animated width value
      height: value.get(AniProps.height), // Get animated height value
      color: Colors.yellow,
    );
  },
);
```

## Using the predefined enum for animation properties

It's always good to create your own `enum` that contain the precise animation properties the animation uses.
But we developers are sometimes lazy.

If you feel lazy can also use the `DefaultAnimationProperties` enum that contains a varity of common used animation properties.

```dart
final _tween = MultiTween<DefaultAnimationProperties>()
  ..add(DefaultAnimationProperties.width, 0.0.tweenTo(100.0), 1000.milliseconds);
```

## Some notes on using durations

### Duration tracking

The `MultiTween` tracks the duration for each property you specified when arranging your tween. In the examples above we used this "tracked duration" (`_tween.duration`) to define the total animation duration.

### Use own durations

You also also use your own `Duration`. MultiTween will automatically lengthen or shorten the tween animation.

```dart
PlayAnimation<MultiTweenValues<AniProps>>(
  tween: _tween,
  duration: 3.seconds, // Use own duration
  builder: (context, child, value) {
    return Container(
      width: value.get(AniProps.width),
      height: value.get(AniProps.height),
      color: Colors.yellow,
    );
  },
);
```

### Default duration when adding tweens to MultiTween

Supplying a duration when calling `add` method is optional as well. It will use `Duration(seconds: 1)` as the default value.

```dart
final _tween = MultiTween<DefaultAnimationProperties>()
  ..add(DefaultAnimationProperties.width, 0.0.tweenTo(100.0)); // no duration supplied

_tween.duration; // Duration(seconds: 1)
```

### Inhomogeneous durations

If properties have different durations `MultiTween` will extrapolate the last value for each property.

```dart
final tween = MultiTween<DefaultAnimationProperties>()
  ..add(DefaultAnimationProperties.width, 0.0.tweenTo(100.0), 1.seconds) // width takes 1 second
  ..add(DefaultAnimationProperties.height, 0.0.tweenTo(1000.0), 2.seconds); // height takes 2 seconds
```
After a second the `width` reaches it's target value of `100.0`. Meanwhile the `height` property is halfway at `500.0`. From now on `width` will stay at `100.0` untils `height` reaches it's target value of `1000.0`. The whole tween is completed after 2 seconds.

## Non-linear animations

You can make your animation more interesting by adding non-linear tweens.
When adding your tweens you can use the (optional) fourth parameter to specify a `Curve`.

Flutter comes with a set of predefined curves inside the `Curves` class.

```dart
final tween = MultiTween<DefaultAnimationProperties>()
  ..add(DefaultAnimationProperties.width, 0.0.tweenTo(100.0), 1000.milliseconds, Curves.easeOut);
```

---------

# Examples

💡 *Note: These examples uses **[supercharged](https://pub.dev/packages/supercharged)** for syntactic sugar.*

## Animate multiple properties

This example animates width, height and color of a box.

![example1](https://raw.githubusercontent.com/felixblaschke/simple_animations_documentation_assets/master/sa_multi_tween/v1/multitween-example-1.gif)

```dart
import 'package:flutter/material.dart';
import 'package:simple_animations/simple_animations.dart';
import 'package:supercharged/supercharged.dart';

void main() => runApp(MyApp());

// Create enum that defines the animated properties
enum AniProps { width, height, color }

class MyApp extends StatelessWidget {

  // Specify your tween
  final _tween = MultiTween<AniProps>()
    ..add(AniProps.width, 0.0.tweenTo(100.0), 1000.milliseconds)
    ..add(AniProps.width, 100.0.tweenTo(200.0), 500.milliseconds)
    ..add(AniProps.height, 0.0.tweenTo(200.0), 2500.milliseconds)
    ..add(AniProps.color, Colors.red.tweenTo(Colors.blue), 3.seconds);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        backgroundColor: Colors.white,
        body: Center(
          child: PlayAnimation<MultiTweenValues<AniProps>>(
            tween: _tween, // Pass in tween
            duration: _tween.duration, // Obtain duration from MultiTween
            builder: (context, child, value) {
              return Container(
                width: value.get(AniProps.width), // Get animated value for width
                height: value.get(AniProps.height), // Get animated value for height
                color: value.get(AniProps.color), // Get animated value for color
              );
            },
          ),
        ),
      ),
    );
  }
}

```




## Chained tweens in single animation

This example moves a box clockwise in a rectangular pattern.

![example2](https://raw.githubusercontent.com/felixblaschke/simple_animations_documentation_assets/master/sa_multi_tween/v1/multitween-example-2.gif)

```dart
import 'package:flutter/material.dart';
import 'package:simple_animations/simple_animations.dart';
import 'package:supercharged/supercharged.dart';

void main() => runApp(MyApp());

// Create enum that defines the animated properties
enum AniProps { offset }

class MyApp extends StatelessWidget {
  // Specify your tween
  final _tween = MultiTween<AniProps>()
    ..add( // top left => top right
        AniProps.offset,
        Tween(begin: Offset(-100, -100), end: Offset(100, -100)),
        1000.milliseconds)
    ..add( // top right => bottom right
        AniProps.offset,
        Tween(begin: Offset(100, -100), end: Offset(100, 100)),
        1000.milliseconds)
    ..add( // bottom right => bottom left
        AniProps.offset,
        Tween(begin: Offset(100, 100), end: Offset(-100, 100)),
        1000.milliseconds)
    ..add( // bottom left => top left
        AniProps.offset,
        Tween(begin: Offset(-100, 100), end: Offset(-100, -100)),
        1000.milliseconds);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        backgroundColor: Colors.white,
        body: Center(
          child: LoopAnimation<MultiTweenValues<AniProps>>(
            tween: _tween, // Pass in tween
            duration: _tween.duration, // Obtain duration from MultiTween
            builder: (context, child, value) {
              return Transform.translate(
                offset: value.get(AniProps.offset), // Get animated offset
                child: Container(
                  width: 100,
                  height: 100,
                  color: Colors.green,
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}

```




## Complex example

This example combines aspects of the examples above, including chaining and multiple properties.

![example3](https://raw.githubusercontent.com/felixblaschke/simple_animations_documentation_assets/master/sa_multi_tween/v1/multitween-example-3.gif)

```dart
import 'package:flutter/material.dart';
import 'package:simple_animations/simple_animations.dart';
import 'package:supercharged/supercharged.dart';

void main() => runApp(MyApp());

// Create enum that defines the animated properties
enum AniProps { offset, color }

class MyApp extends StatelessWidget {
  // Specify your tween
  final _tween = MultiTween<AniProps>()
    ..add(
      // top left => top right
        AniProps.offset,
        Tween(begin: Offset(-100, -100), end: Offset(100, -100)),
        1000.milliseconds, Curves.easeInOutSine)
    ..add(
      // top right => bottom right
        AniProps.offset,
        Tween(begin: Offset(100, -100), end: Offset(100, 100)),
        1000.milliseconds, Curves.easeInOutSine)
    ..add(
      // bottom right => bottom left
        AniProps.offset,
        Tween(begin: Offset(100, 100), end: Offset(-100, 100)),
        1000.milliseconds, Curves.easeInOutSine)
    ..add(
      // bottom left => top left
        AniProps.offset,
        Tween(begin: Offset(-100, 100), end: Offset(-100, -100)),
        1000.milliseconds, Curves.easeInOutSine)
    ..add(AniProps.color, Colors.red.tweenTo(Colors.yellow), 1.seconds)
    ..add(AniProps.color, Colors.yellow.tweenTo(Colors.blue), 2.seconds)
    ..add(AniProps.color, Colors.blue.tweenTo(Colors.red), 1.seconds);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        backgroundColor: Colors.white,
        body: Center(
          child: LoopAnimation<MultiTweenValues<AniProps>>(
            tween: _tween, // Pass in tween
            duration: _tween.duration, // Obtain duration from MultiTween
            builder: (context, child, value) {
              return Transform.translate(
                offset: value.get(AniProps.offset), // Get animated offset
                child: Container(
                  width: 100,
                  height: 100,
                  color: value.get(AniProps.color), // Get animated color
                ),
              );
            },
          ),
        ),
      ),
    );
  }
}

```

