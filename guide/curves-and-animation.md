---
layout: guide
title: Curves and Animation - Helm, a functionally reactive game engine
section: Curves and Animation
permalink: /guide/curves-and-animation/
---

## Curves and Animation

This guide is a direct extension of [the Colors guide](/helm/guide/colors/),
building on top of the same code to extend its functionality. If you haven't
already done that one, you should probably do it now.

### Curves

#### ArcShape

In order to use somewhat more generic curves (not **just** circles and ovals),
we will have to utilize the `ArcShape` constructor directly from
[*Helm.Graphics2D*](https://hackage.haskell.org/package/helm-1.0.0/docs/Helm-Graphics2D.html#v:ArcShape).
The signature is as follows:

{% highlight haskell %}
ArcShape (V2 Double) Double Double Double (V2 Double)
{% endhighlight %}

The first argument here is specifying the center of the arc, i.e. the place
where all radii originate from. The next two arguments are angle measurements
(in radians) that dictate the starting and ending angle of the arc. The next
argument specifies the radius. And finally, the last vector scales the entire
shape along the x- and y-axes, according to the x and y components of the
vector. These scaling factors effectively multiply distances along the x- and
y-axes by a scalar.

Another good thing to know about `ArcShape` is that if the arc produced by
these five parameters does not start and end at the same point, then a line
segment will be drawn to connect the two ends. With this in mind, we can just
draw partial circles (sweeping out the radius **less** than `2 * pi` radians
each time) for each slice of the wheel, and the line segment that connects up
each curve with itself will be along the same line as the outer edge of each
slice.

#### Adding Round Caps to the Slices

We're going to update our `slice` function to return **two** `Form`s instead of
just the one triangle, so that each slice can be represented as a triangle with
a round cap (`ArcShape`) on the end. For that, we need a new type signature:

{% highlight haskell %}
slice :: Int -> [Form e]
{% endhighlight %}

Easy enough. Just return a list. Now we change the first line of the function
to apply the color fill to both the triangle and the arc that we're about to
make:

{% highlight haskell %}
slice n = map (filled color) [polygon (path points), cap]
{% endhighlight %}

Now we define what we want our `cap` to be, within the `where` definitions:

{% highlight haskell %}
cap = ArcShape (V2 0 0) t1 t2 r (V2 1 1)
{% endhighlight %}

We want the origin of the arc to be at the origin of the wheel, so we can just
use `(0, 0)`. The start and stop angles for the arc are the same start and stop
angles we set up for the triangle bits, `t1` and `t2`. The radius `r` is also
the same. Since this is just tracing part of an ordinary circle, we can leave
the scaling factors alone at `1` and `1`.

Just a little tweak to the `view` function logic should get this incorporated.
All we need is to `concat` the result of the list comprehension, since it's now
a list of lists of forms:

{% highlight haskell %}
view Model{..} = Graphics2D
  {- ... -}
  $ concat [slice angleOffset n | n <- [0..length colors - 1]]
{% endhighlight %}

Again, using record elision syntax.

### Animation

#### Update the Model and Add an Action

In order to make the wheel spin, we will want to keep the current rotation
angle that the wheel is at in our `Model`:

{% highlight haskell %}
data Model = Model { angleOffset :: Double }
{% endhighlight %}

Now that we have a new `Model`, we update our `init` function accordingly,
setting the initial offset to zero:

{% highlight haskell %}
initial = (Model { angleOffset = 0 }, Cmd.none)
{% endhighlight %}

We can also update once again the last line of the `view` function, passing the
`slice` function another argument &mdash; the angle offset. That way, `slice`
can generate the `Form`s already rotated how we want them; this will just
require making a few changes to `slice` in a little bit.

{% highlight haskell %}
$ concat [slice angleOffset n | n <- [0..length colors - 1]]
{% endhighlight %}

Additionally we now have to make use of an `Action`, since there will need to
be an `Action` fired every frame that will update the model by turning the
wheel (incrementing `angleOffset`). Each constructor for an `Action` will
usually contain some kind of information that is associated with it, so it will
be written in product type form. Here, the information associated with each one
of our "`Animate`" actions will be the time elapsed since the last `Animate`
call, represented as a `Double`:

{% highlight haskell %}
data Action = Animate Double
{% endhighlight %}

With this, the current definition of `update` that pattern matches on
`DoNothing` (which no longer exists) can be rewritten:

{% highlight haskell %}
update Model{..} (Animate dt) =
  ( Model { angleOffset = angleOffset + dt * 2e-3 }
  , Cmd.none
  )
{% endhighlight %}

Here, `2e-3` is just an arbitrary scalar that dictates how quickly and in which
direction the wheel spins. We've named the argument to the `Animate`
constructor `dt`, as in, the change in time since the last time this was
called. Using the current state of the model, plus the timestep times the
angular velocity, we update the model's angle offset so that the wheel's angle
continuously spins.

Note that this example code is using record elision syntax (`Model{..}`), which
becomes very convinent for the sometimes rather large records used with Helm.
To use this syntax you need to put `{-# LANGUAGE RecordWildCards #-}` at the
top of the file.

We also want to update our `subscriptions` function, to use our new sort of
`Action`. We'll do this by first writing `import qualified Helm.Time as Time`
with our imports, and then using its `fps` function to specify both a framerate
and an `Action` to do each frame, respectively:

{% highlight haskell %}
subscriptions = Time.fps 60 Animate
{% endhighlight %}

#### Changing the Slice Function

The finishing touch that we need now is to update the `slice` function due to
the changes we made to `view`. Let's start by just modifying the type
signature:

{% highlight haskell %}
slice :: Double -> Int -> [Form e]
{% endhighlight %}

And then the parameter list:

{% highlight haskell %}
slice offset n =
{% endhighlight %}

And really the only thing we need now is to change the way that `t1` is
calculated, by adding in the `offset`. Since `t2` is calculated based off of
`t1`, that's all we need:

{% highlight haskell %}
t1 = offset + increment * realToFrac n
{% endhighlight %}

### Final Product

The final product renders the wheel from the first tutorial, but with round
edges instead of flat ones. Additionally, the wheel rotates at a specified
velocity.

[Checkout the code on Github →](https://github.com/AugmentedFifth/helm-website-examples/blob/master/src/CurvesAndAnimation.hs)

![final](/helm/img/animated-wheel.png)
