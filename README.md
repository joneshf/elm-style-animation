
# elm-html-animation

A library to simplify creating html animations in elm. My focus was to create something I could use as a UI designer to prototype animations quickly, accurately, and without sneaky errors.

  1. Showing a menu on hover - [demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/SideMenu.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/SideMenu.elm)
  2. Chaining animations together - [demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/Chaining.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/Chaining.elm)
  3. Managing multiple animations - [demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/Showcase.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/Showcase.elm)
  4. Stacking transformations - [demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/StackingTransforms.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/StackingTransforms.elm)
  5. Staggering animations - [demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/Stagger.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/Stagger.elm)
  6. Realistic scenario (flower menu) (separate repo) - [demo](https://mdgriffith.github.io/elm-html-animation/examples/FlowerMenu/) / [view code](https://github.com/mdgriffith/elm-html-animation-flower-menu/blob/master/FlowerMenu.elm)


## Installation

Try it out!

First have [Elm installed](http://elm-lang.org/install), then

If you just want to play with the examples, run the following in a console:

```bash
git clone https://github.com/mdgriffith/elm-html-animation.git
cd elm-html-animation/examples
elm-reactor
# Make sure to cd into the examples folder.
# The library and the examples have different dependencies
# So, running elm-reactor on the base folder will not work
# if you just want to run examples.
```

Or, if you want to install the package in one of your elm projects.

```bash
elm-package install mdgriffith/elm-html-animation
```


## The Basics

I recommend checking out [the Elm Architecture](https://github.com/evancz/elm-architecture-tutorial/) if you haven't already.  These examples will be much easier if you're already familiar with the standard `model`, `update`, and `view` pattern.  As well you should understand the basics of using the `Effects` library.

So, with all that in mind, here's a basic overview of what you'll need to do to use this library.

To add animations to a module, you'll need to do the following:

  * Store the styling data in your `model`, and define an initial style.

__Note__ all properties that are going to be animated need to be accounted for in the initial style.

```elm
import Style
import Style.Properties exposing (..)
import Color exposing (rgba)

type alias Model = { style : Style.Animation }

init : Model
init = { style = 
            Style.init 
                [ Left -350.0 Px
                , Opacity 0.0 
                , Color (rgba 50 50 50 1.0)
                ]
        }
```



 * In your `view`, render the animation as a css style.

```elm
view : Address Action -> Model -> Html
view address model =
          div [ style (Style.render model.style) ] []

```


  * Add a new action to your Action type to allow updates to be sent to an animation.

```elm
type Action = Show -- This action triggers the animation
            | Animate Style.Action -- This action forwards all updates to the elm-html-animation core.



-- In our update function, we can use the helper function trigger animations 
-- and to pass updates to an animation
update : Action -> Model -> ( Model, Effects Action )
update action model =
  case action of

    Show ->
      Style.animate 
         |> Style.to
             [ Opacity 1
             ] 
         |> Style.on model.style

    Animate action ->
      onMenu model action
```

 
  * Start creating animations!  Now that we're set up, we can begin animating. 


# Example 1: Showing a Menu on Hover

[demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/SideMenu.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/SideMenu.elm)

Our first example is a menu that is shown when the mouse enters a certain area, and hides when the mouse leaves.

So, our first step is to add two values to our Action type, `Show` and `Hide`, and in the update function, we start an animation when those actions occur.  Let's take a look at how we construct an animation.


```elm
-- ... we add this to the case statement in our update function.
    Show ->
      Style.animate 
      -- |> Style.delay (0.5*second)
         |> Style.to 
             [ Left 0 Px
             , Opacity 1
             ] 
         |> Style.on model.style

```

Notice we are programming declaratively by defining what style property should be by using `Style.to`.  A delay is present but commented out, just to make you aware of how to do delays.

Instead of using a duration and an easing function, this library defaults to animating by using a __spring__.  By modeling real-world springs, we can create organic animations by defining two numbers, _stiffness_ and _damping_.


```elm
-- ... we add this to the case statement in our update function.
    Show ->
      Style.animate 
      -- |> Style.spring Style.wobbly -- you can use a UI preset
      -- or specify manually.
         |> Style.spring 
              { stiffness = 400
              , damping = 28
              }
         |> Style.to 
             [ Left 0 Px
             , Opacity 1
             ] 
         |> Style.on model.style

```


> Alternatively, we also have the option of defining a _duration_, and an _easing function_.  I've generally found that springs are a more natural way for animating user interfaces, but I believe there are some cases where easing and duration could be preferable.  Here's how it's done.

 ```elm
       Style.animate 
           |> Style.easing (\x -> x)  -- linear easing
           |> Style.duration (0.5*second)
           |> Style.to 
               [ Left 0 Px
               , Opacity 1
               ] 
           |> Style.on model.style
  ```
> 
> __Note__ The duration you provide will not do anything unless you also provide an easing function.  This is because spring based animations set the duration dynamically.
> Make sure to check out this [library](http://package.elm-lang.org/packages/Dandandan/Easing/2.0.1/Easing#easing-functions) if you're looking for easing functions.

Now that we have this animation, it has a few properties that may not be immediately apparent.  If a `Hide` action is called halfway through execution of the `Show` animation, the animation will be smoothly interrupted. 

However, there may be a situation where we don't want our animation to be interrupted.  Instead we might want the current animation to play out completely and for our new animation to play directly afterwards.  To do this, we would use `Style.queue` instead of `Style.animate`


# Example 2: Chaining Animations Together

[demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/Chaining.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/Chaining.elm)

We also have option of chaining animations together.  So, let's make a square that cycles through a few background colors when you click it.

```elm
import Color exposing (rgba)

-- in our update function:
    ChangeColor ->
      Style.animate 
        |> Style.to 
            [ BackgroundColor (rgba 100 100 100 1.0)
            ] 
        |> Style.andThen -- create a new keyframe
        |> Style.to 
            [ BackgroundColor (rgba 178 201 14 1.0)
            ] 
        |> Style.andThen 
        |> Style.to 
            [ BackgroundColor (rgba 58 40 69 1.0)
            ] 
        |> on model.menuStyle

```


In this case we can use `Style.andThen` to create a new key frame.  This new keyframe will have it's own independent delay, properties, and spring (or easing + duration).  Again, it can be interrupted at any point.


# Example 3: Managing Multiple Animations

[demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/Showcase.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/Showcase.elm)

It's also fairly common to have a list of elements that all need to animated individually.  In order to do this we need to make a few changes to our boilerplate code we had in the beginning.


First, our model would be something like this:

```elm

type alias Model = { widgets : List Widget }

type alias Widget = 
          { style : Style.Animation
          }

```


```elm
      -- where j is the index of the widget we want to animate.
      let 
        widgets = 
            List.indexedMap 
              (\i widget ->
                  if i == j then
                     Style.animate
                        |> Style.duration (5*second)
                        |> Style.to 
                            [ Opacity 0  
                            ] 
                        |> Style.on widget
                  else
                    widget
              ) model.widgets
           

      in
        ( { model | widgets = widgets }
        , Cmd.none )

```


Because of this change, we also have to slightly change our `Animate` action to include the index of the widget that is being animated.

So, our Animate action gets an Int.

```elm

type Action = Animate Int Style.Action
```

And the update function forwards an animation update to a specific widget using our forwardToWidget.
```elm
    Animate i action ->
      let
        (widgets, fx) = 
            onWidget i model.widgets action 
      in
        ( { model | widgets = widgets }
        , fx )

```


From here, we're able to animate each widget independently.


# Example 4: Stacking Transformations

[demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/StackingTransforms.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/StackingTransforms.elm)

CSS has support for `transforms` such as `translate`, `rotate`, and `scale`.  We also have access to some more complicated transformations such as `rotate3d` and `transform3d`.

When using these transformations in normal css, you're able to stack them.  So, in your css file you can have something like the following:

```css
.transformed {
  transform: rotate(20deg) translateY(100px) rotate(-20deg)
}
```

In this case, the transforms are performed in order. Rotate by 20deg, translateY (which is now angled 20deg) by 100px, and then rotate again -20deg.

This can be very useful, especially if we can animate each transform element individually. 

Here's how we're able to do this in Html.Animation.

First, we define our initial style.  This will define the order that the transforms will appear. 

```elm
initialWidgetStyle = 
        Style.init 
            [ Rotate 0 Deg
            , TranslateY 0 Px
            , Rotate 0 Deg
            ]

```


Now let's animate these properties.  Let's say we want do the following animation:

  1. Make the first `Style.Rotate` move to 20deg.  
  2. Once that's finished, we want to translateY to -200px
  3. Now we want to rotate 1 revolution locally
  4. Now we want to rotate 1 revolution around the original center
  5. Then once that's finished, reset everything to 0.

```elm
  Style.animate 
      |> Style.duration (0.5*second)
      |> Style.to 
          [ Rotate 20 Deg
          ] 
      |> Style.andThen
      |> Style.duration (0.7*second)
      |> Style.props 
          [ TranslateY -200 Px
          ] 
      |> Style.andThen
      |> Style.duration (0.7*second)
      |> Style.update
          (\index prop ->
              case prop of 
                  Rotate angle unit ->
                     if index == 2 then
                        Rotate 360.0 unit
                    else
                        Rotate angle unit
                  _ -> prop
          ) 
          
      |> Style.andThen
      |> Style.duration (0.7*second)
      |> Style.to 
          [ Style.Rotate 380 Style.Deg 
          ] 
      |> Style.andThen
      |> Style.delay (1*second)
      |> Style.to
          [ Rotate 0 Deg
          , TranslateY 0 Px
          , Rotate 0 Deg
          ] 

```

There's only one new function and that's `Style.stay`.

`Style.stay` just keeps a property at it's current value.  Normally this is just assumed for any property that isn't explicitly animated, but we need `Style.stay` in this case to access that second `Rotate` property.


# Example 5: Staggering Animations

[demo](https://mdgriffith.github.io/elm-html-animation/3.0.0/examples/Stagger.html) / [view code](https://github.com/mdgriffith/elm-html-animation/blob/master/examples/Stagger.elm)

Staggering animations can add a very pleasing effect to an animation.

Staggering is running an animation on a list of things, and having that thing's index in that list affect some part of the animation.

A classic example is to delay each element of the list just slightly compared the the previous one.

How can we do this?  Well, essentially a stagger is just a function that takes the element's position and returns the animation it should have.  So that's what we do!


```elm

List.indexedMap
    (\i widget -> 
       Style.animate
         |> Style.delay (i * 0.05 * second) -- i is the index of the widget in the list.
         |> Style.duration (0.3 * second)
         |> Style.to 
             [ Left 200 Px
             ] 
         |> Style.andThen
         |> Style.delay (2.0 * second)
         |> Style.duration (0.3 * second)
         |> Style.to 
             [ Left -50 Px
             ]
         |> Style.on widget
    ) model.widgets



```


The cool part about this way of approaching it is that you can stagger anything you could normally animate: the delay, duration, props, etc. 


# Notes and Common Pitfalls


## Uninitialized Style Animations

If you attempt to animate a property that hasn't been provided in the initial style, _that property will not be animated_ and a warning will be logged.

This was done because it seems to be the better option than animating with an arbitrarily chosen default value and with arbitrarily chosen default units.

So, if a property isn't animating, check your Style.init!


## Doubly Specified Units

If you specify different units for a property in the Style.init vs the animation, _that property animation will also not be animated_ and a warning will be logged.

I know this is not the best solution, but there are a few reasons why it's the case.

 1. Html.Animation can't automatically convert between units.  A classic example of why not, would be attempting to convert px to %. This can't be done without querying the DOM.

 2. Again we want to avoid making up arbitrary default values.

I'm currently working on a solution by making units only specified in the Style.init, however this turns out to be fairly complicated (see __eliminating-twice-declared-units__ branch if you're curious and want a headache.)  

So, maybe this will be solved in the future?


## Slow?

If you're implementing a lot of animationatable things, you'll want to use [Html.Lazy](https://github.com/evancz/elm-html/blob/master/src/Html/Lazy.elm).  Basically any function that creates Html can be called with Html.lazy

```elm
-- So a function that has this signature
view : Model -> Html

-- would be called as
Html.Lazy.lazy2 view model

-- instead of just 
view model

-- Instant performance!

```






