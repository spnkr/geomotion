# Geomotion for [RubyMotion](http://rubymotion.com)

iOS Geometry in idiomatic Ruby. Exhaustively tested. What's not to love?

## Features

### CGRect

```ruby
# Initializers
rect = CGRect.make(x: 10, y: 100, width: 50, height: 20)
another_way = CGRect.make(origin: CGPoint, size: CGSize)

# Getters
[rect.x, rect.y, rect.width, rect.height]
=> [10, 100, 50, 20]

rect_zero = CGRect.empty
=> CGRect(0, 0, 0, 0)
rect_zero.empty?
=> true

# to get the center of the frame, relative to the origin or in absolute coordinates
rect.center
=> CGPoint(25, 10) # center relative to bounds
rect.center(true)
=> CGPoint(35, 110) # center relative to frame

# Other points in the rect can be returned as well, and the same
# relative/absolute return values are supported (defaults to relative)

         top_left  top_center
                |  |
                o--o--o top_right
                |     |
    center_left o  x  o center_right
                |     |
                o--o--o bottom_right
                |  |
      bottom_left  bottom_center

# Operator Overloading
-rect
=> CGRect(-10, -100, -50, -20)

# union of rects
rect + CGRect.make(x: 9, y: 99, width: 10, height: 10)
=> rect.union_with(CGRect.make(x: 9, y: 99, width: 10, height: 10))
=> CGRect(9, 99, 50, 20)

# increases the size, but keeps the origin
rect + CGSize.make(width: 11, height: 1)
=> CGRect(10, 100, 61, 21)
# not the same as `grow`, which grows the rect in all directions

# moves the rect
rect + CGPoint.make(x: 10, y: 10)
=> rect.offset(CGPoint.make(x: 10, y: 10))
=> CGRect(20, 110, 50, 20)

# moves the rect
rect + UIOffsetMake(10, 10)
=> rect.offset(UIOffsetMake(10, 10))
=> CGRect(20, 110, 50, 20)

a_point + a_size
=> CGRect # a point and a size make a rectangle. makes sense, right?

# Union and Intersection
rect.union_with CGRect.make(x: 9, y: 99, width: 10, height: 10)
=> CGRect(9, 99, 50, 20)

rect.intersection_with CGRect.make(x: 9, y: 99, width: 10, height: 10)
=> CGRect(10, 100, 10, 10)

# Growing and shrinking
# The center stays the same. Think margins!
rect.grow(CGSize.make(width: 10, height: 20))
=> CGRect(5, 90, 60, 40)

rect.shrink(10)
=> CGRect(15, 105, 40, 10)

# Powerful layout adjustments with chainable methods
view = UIView.alloc.initWithFrame rect.below.width(100).height(10)
view.frame
=> CGRect(10, 120, 100, 10)

view2 = UIView.alloc.initWithFrame rect.beside(10)
view2.frame
=> CGRect(70, 100, 50, 20.0)

# More examples of adjustments
rect = CGRect.make(x: 10, y: 100, width: 50, height: 20)
[rect.right(20).x, rect.left(20).x, rect.up(20).y, rect.down(20).y]
=> [30, -10, 80, 120]

# Layout "above" and "before" rectangles
# (default offset is the rectangle's width or height)
rect.before(5)
=> CGRect(-45, 100, 50, 20)
rect.before(5, width:20)
=> CGRect(-15, 100, 20, 20)

rect.above(5)
=> CGRect(10, 75, 50, 20)
rect.above(5, height:10)
=> CGRect(10, 85, 50, 10)

# Layout a rect relative to others
rect2 = CGRect.make(x: 50, y: 50, width: 100, height: 100)
rect3 = CGRect.make(x:100, y: 200, width: 20, height: 20)

CGRect.layout(rect, above: rect2, right_of: rect3)
=> CGRect(120, 30, 50, 20)

# Also supports margins
CGRect.layout(rect, above: rect2, right_of: rect3, margins: [0, 0, 10, 15])
=> CGRect(135, 20, 50, 20)

```

## *Relative* vs *Absolute*

When you are positioning frames, you'll be doing so in one of two ways:

1. Two frames relative *to each other*, within a common parent frame
2. A frame being added *as a child* of another frame

*(generally speaking)*

geomotion is optimized for both cases, but the arsenal of methods is different.

###### frames relative to each other

Any of the location methods (`up, down, left, right, beside, before, above, below`)
will return a frame that is in the same coordinate system of the receiver, and
this behavior cannot be changed.

```ruby
frame = CGRect.make(x: 10, y: 10, width:10, height: 10)
frame.beside
# => [[20, 10], [10, 10]]

frame.right(30).down(5).taller(100)
# => [[10+30, 10+5], [10, 10+100]]
# aka
# => [[40, 15], [10, 110]]
```

Any methods that include the *x* or *y* variable in their name will be absolute.

```ruby
frame.x
frame.min_x
frame.max_x
frame.mid_x
frame.y
frame.min_y
frame.max_y
frame.mid_y
```

###### positions relative to the frame's origin

Any of the position methods that do *NOT* include the *x* or *y* variable will
*ignore* the *x* and *y* values unless explicitly told to use absolute
coordinates.

*Note:* These methods will "normalize" the width and height, so even if
the width or height is negative, these methods will always return positive
numbers.  If you specify absolute coordinates, the values might be negative, but
they will also be sorted (x == min, min < mid, mid < max, x + width == max).

```ruby
frame = CGRect.make(x: 10, y: 10, width:10, height: 10)
frame.top_left  # => [0, 0]
frame.top_center  # => [5, 0]
frame.bottom_right  # => [10, 10]

# use absolute coordinates
frame.top_left(true)  # => [10, 10]
frame.top_center(true)  # => [15, 10]
frame.bottom_right(true)  # => [20, 20]

# negative widths and heights are "corrected" when using absolute coordinates
frame = CGRect.make(x: 20, y: 20, width:-10, height: -10)
frame.top_center(true)  # => [15, 10]
frame.bottom_right(true)  # => [20, 20]
```

### CGSize

```ruby
# Initializers
size = CGSize.make(width: 50, height: 20)

# Getters
size_zero = CGSize.empty
=> CGSize(0, 0)
size_zero.empty?
=> true

# Operator Overloading
-size
=> CGSize(-50, -20)

size + CGSize.make(width: 100, height: 50)
=> CGSize(150, 70)

size + CGPoint.make(x: 10, y: 30)
=> CGRect(10, 30, 50, 20)

# Combine with CGPoint
size.rect_at_point CGPoint.make(x: 10, y: 30)
=> CGRect(10, 30, 50, 20)
```

### CGPoint

```ruby
# Initializers
point = CGPoint.make(x: 10, y: 100)

# Return a modified copy
point.up(50).left(5)
=> CGPoint(5, 50)
# original is not modified, a new point is returned
point.down(50).right(5)
=> CGPoint(15, 150)

# Operator Overloading
-point
=> CGPoint(-10, -100)

point + CGPoint.make(x: 20, y: 40)
=> CGPoint(30, 140)

point + CGSize.make(width: 50, height: 20)
=> CGRect(10, 100, 50, 20)

# Combine with CGSize
point.rect_of_size CGSize.make(width: 50, height: 20)
=> CGRect(10, 100, 50, 20)

# Compare with CGRect
point.inside? CGRect.make(x: 0, y: 0, width: 20, height: 110)
=> true

# Distance to point
point.distance_to(CGPoint.make(x: 13, y:104))
=> 5

# Angle between target and receiver
# (up 10, over 10)
point.angle_to(CGPoint.make(x: 20, y:110))
=> 0.785398163397  (pi/4)
```

### CGAffineTransform

These are assigned to the `UIView#transform` parameter.  See `CATransform3D` for
the transforms that are designed for `CALayer` object.

```ruby
# you *can* create it manually
transform = CGAffineTransform.make(a: 1, b: 0, c: 0, d: 1, tx: 0, ty: 0)

# but don't!  the `make` method accepts `translate`, `scale`, and `rotate` args
transform = CGAffineTransform.make(scale: 2, translate: [10, 10], rotate: Math::PI)

# identity transform is easy
CGAffineTransform.identity

# just to be sure
CGAffineTransform.identity.identity?  # => true

# Operator Overloading
transform1 = CGAffineTransform.make(scale: 2)
transform2 = CGAffineTransform.make(translate: [10, 10])
# concatenate transforms
transform1 + transform2
transform1 << transform2  # alias
transform1 - transform2
# => transform1 + -transform2
# => transform1 + transform2.invert
transform1 - transform1  # => CGAffineTransform.identity

# create new transforms by calling `translate`, `scale`, or `rotate` as factory
# methods
CGAffineTransform.translate(10, 10)
CGAffineTransform.scale(2)  # scale x and y by 2
CGAffineTransform.scale(2, 4)  # scale x by 2 and y by 4
CGAffineTransform.rotate(Math::PI / 4)

# "shearing" turns a rectangle into a parallelogram
# see sceenshot below or run geomotion app
CGAffineTransform.shear(0.5, 0)  # in x direction
CGAffineTransform.shear(0, 0.5)  # in y direction
# you can combine these, but it looks kind of strange.  better to pick one
# direction

# or you can chain these methods
CGAffineTransform.identity.translate(10, 10).scale(2).rotate(Math::PI / 4)
```

###### Shearing

![Shearing](https://raw.github.com/colinta/geomotion/master/resources/shearing.png)

### CATransform3D

`CALayer`s can take on full 3D transforms.

```ruby
# these are really gnarly
transform = CATransform3D.make(
  m11: 1, m12: 0, m13: 0, m14: 0,
  m21: 0, m22: 1, m23: 0, m24: 0,
  m31: 0, m32: 0, m33: 1, m34: 0,
  m41: 0, m42: 0, m43: 0, m44: 1,)

# accepts transforms like CGAffineTransform, but many take 3 instead of 2 args
transform = CATransform3D.make(scale: [2, 2, 1], translate: [10, 10, 10], rotate: Math::PI)

# identity transform
CATransform3D.identity
CATransform3D.identity.identity?  # => true

# Operator Overloading
transform1 = CATransform3D.make(scale: 2)
transform2 = CATransform3D.make(translate: [10, 10])
# concatenate transforms
transform1 + transform2
transform1 << transform2  # alias
transform1 - transform2
# => transform1 + -transform2
# => transform1 + transform2.invert
transform1 - transform1  # => CATransform3D.identity

# create new transforms by calling factory methods
CATransform3D.translate(10, 10, 10)
CATransform3D.scale(2)  # scale x and y by 2
CATransform3D.scale(2, 4, 3)  # scale x by 2, y by 4, z by 3
CATransform3D.rotate(Math::PI / 4)

# "shearing" works the same as CGAffineTransform
CATransform3D.shear(0.5, 0)  # in x direction
CATransform3D.shear(0, 0.5)  # in y direction
# "perspective" changes are better than rotation because they make one side
# bigger and one side smaller
# see sceenshot below or run geomotion app
CATransform3D.perspective(0.002, 0)  # similar to rotating around x-axis
CATransform3D.perspective(0, 0.002)  # "rotates" around the y-axis

# or you can chain these methods
CATransform3D.identity.translate(10, 10, 10).scale(2).rotate(Math::PI / 4)
```

###### Perspective

![Perspective](https://raw.github.com/colinta/geomotion/master/resources/perspective.png)

## Install

1. `gem install geomotion`

2. Add `require 'geomotion'` in your `Rakefile`.


## Forking

If you have cool/better ideas, pull-request away!
