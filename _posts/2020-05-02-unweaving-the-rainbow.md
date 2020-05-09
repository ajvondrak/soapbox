---
layout: post
title: "Classes & Inheritance: Unweaving the Rainbow"
---

> Philosophy will clip an Angel's wings,<br/>
> Conquer all mysteries by rule and line,<br/>
> Empty the haunted air, and gnomed mine&ndash;<br/>
> Unweave a rainbow, as it erewhile made

&mdash; [Lamia (John Keats)](https://en.wikipedia.org/wiki/Lamia_%28poem%29)

I'm an academic at heart. I don't think that's a bad thing for a software engineer, but it gets kind of lonely.

I notice it when phone screening. As an interviewer, I might ask something about object-oriented design. "Can you tell me how inheritance works?" Typically, I'll hear answers about code reuse, subclasses delegating to superclasses, quirks of specific programming languages, [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance), etc.

These answers aren't wrong. But I die a little bit inside when it's all I hear. The academic part of me yearns for more depth. In my experience, most programmers can describe what inheritance *does*, but not what it really *is*.

Nineteenth-century poet John Keats purportedly [quipped](https://www.scientificamerican.com/article/unweaving-the-heart/) that scientist Issac Newton "destroyed the poetry of the rainbow by reducing it to a prism". And sure, learning how a magic trick works might rob you of a certain joy. But I think the broader programming community would do well not to treat language constructs as magic tricks&mdash;as some syntactic incantation you just learn by rote. Not only can there be a joy in knowing, there are practical benefits to digging deeper. You'll take the lessons learned with you on all your future programming endeavors.

So instead of waiting to hear it from an interviewee like some secret academic handshake, I want to educate. Let's destroy a rainbow together.

## Classes

You can't take two steps in Programming Town without [tripping over object orientation](https://en.wikipedia.org/wiki/List_of_object-oriented_programming_languages): C++, C#, Java, Kotlin, Perl, PHP, Python, Ruby, Scala, Swift, ... The syntax, nomenclature, and other gory details vary, but I'll assume you're familiar with the ideas in general. The gist being: you define your own classes of objects (the nouns of your program) that then contain constructors, variables, methods, etc. An individual object constructed by the class is called an instance, and the instance's methods are what do the main work of your program.

While this post isn't really specific to any language, Ruby is a quintessential example of an object-oriented language, so I'll go ahead and use it for the code throughout. Let's define a simple class.

```ruby
class Rainbow
  attr_reader :colors

  def initialize(colors)
    @colors = colors
  end
end
```

The `Rainbow` class has an initializer that takes in an array of colors and saves them in an instance variable. That variable is accessible by a corresponding getter method.

To represent the individual colors in a rainbow, we define yet another class.

```ruby
class Color
  attr_reader :alpha

  def transparent?
    alpha == 0.0
  end

  def translucent?
    0.0 < alpha && alpha < 1.0
  end

  def opaque?
    alpha == 1.0
  end
end
```

To make things interesting, we define the `Color` class with an [alpha channel](https://en.wikipedia.org/wiki/Alpha_compositing)&mdash;a percentage used to represent the opacity of a pixel when we render the color. Not only do we have a getter method for this alpha, we define several methods implementing logic involving its value.

So far, so good. But we can't really construct a meaningful color yet. The problem is that there are several different ways to represent colors in a computer. That's where we'll use inheritance.

## Inheritance

The term "inheritance" is meant to evoke the concept of genetics. When children inherit traits from their parents, the parents don't lose those traits. Children may also have additional traits beyond the ones they've inherited.

So, too, can we propagate traits between classes in a program. We define subclasses that inherit from superclasses. Inherit what, though? Generally, the methods and variables. Features like access modifiers (e.g., `private` in Ruby) change which exact things get inherited, but the details aren't relevant to this post.

Defining a subclass is largely the same as defining a standard class.

```ruby
class RGB < Color
  attr_reader :red
  attr_reader :green
  attr_reader :blue

  def initialize(red, green, blue, alpha = 1.0)
    @red = red
    @green = green
    @blue = blue
    @alpha = alpha
  end
end
```

The `RGB` class is a subclass of `Color` (and `Color` is a superclass of `RGB`). It represents a point in the [RGB color model](https://en.wikipedia.org/wiki/RGB_color_model): three values between 0 and 255 for the redness, greenness, and blueness of a pixel. Combined, these 3 primary colors add together to form a secondary color, ranging from black (0, 0, 0) to white (255, 255, 255).

Using `RGB`, we can finally instantiate meaningful colors.

```ruby
red = RGB.new(255, 0, 0)
orange = RGB.new(255, 165, 0)
yellow = RGB.new(255, 255, 0)
green = RGB.new(0, 255, 0)
blue = RGB.new(0, 0, 255)
indigo = RGB.new(75, 0, 130)
violet = RGB.new(238, 130, 238)
```

Due to inheritance, every instance of `RGB` responds to methods defined by `Color`. This is how inheritance promotes code reuse: basically by "copying" code (if only virtually) so you don't have to.

```ruby
dimmed_red = RGB.new(255, 0, 0, 0.5)
dimmed_red.transparent? #=> false
dimmed_red.translucent? #=> true
dimmed_red.opaque? #=> false
```

We can keep defining more subclasses, and even subclasses of those subclasses.

```ruby
class Cylindrical < Color
  attr_reader :hue
  attr_reader :saturation
  attr_reader :component

  def initialize(hue, saturation, component, alpha = 1.0)
    @hue = hue
    @saturation = saturation
    @component = component
    @alpha = alpha
  end
end

class HSL < Cylindrical
  alias lightness component
end

class HSV < Cylindrical
  alias value component
end
```

The `Cylindrical` class is another subclass of `Color`. It is a generalized representation of a point in a [cylindrical-coordinate color model](https://en.wikipedia.org/wiki/HSL_and_HSV). Such a color has three pieces: a hue represented as an angle between 0-360°, a saturation between 0 and 1, and a third component whose name & possible values vary by the specific model.

Some examples of such models are given by the `Cylindrical` subclasses `HSL` (hue, saturation, lightness) and `HSV` (hue, saturation, value). They inherit the getters for hue and saturation, as well as the getter for the generic third component. But `component` isn't very descriptive, so each subclass [aliases](https://www.rubyguides.com/2018/11/ruby-alias-keyword/) it, creating a getter with a more specific name.

We can represent a color like [forest green](https://colorpicker.me/#228b22) using equivalent instances of each class.

```ruby
rgb = RGB.new(34, 139, 34)
hsl = HSL.new(120, 0.61, 0.34)
hsv = HSV.new(120, 0.76, 0.55)
```

Even more [color models](https://en.wikipedia.org/wiki/Color_model) exist, but digging into all of them isn't the sort of rainbow wrecking I had in mind. 😉

## Classes unwoven

My point is that, as programmers, we know how to define classes mechanistically in our language of choice. In fact, there's quite a lot we know about how to use them: writing initializers, instantiating objects, making getter methods over instance variables, defining methods that use those values. You probably know even more than what I'm summarizing here. The `Rainbow` class and all the different `Color` classes are pretty humdrum in most languages.

But if we know classes merely by how we use them, then a class is whatever our language makes of it&mdash;all the syntax, semantics, patterns, and boilerplate. In Python, you use [explicit `self`s](https://neopythonic.blogspot.com/2008/10/why-explicit-self-has-to-stay.html). In Java, you have to be mindful about the [difference between `int` and `Integer`](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html). In C++, [initialization is bonkers](https://blog.tartanllama.xyz/initialization-is-bonkers/). In Kotlin, you need to declare a class [`open`](https://kotlinlang.org/docs/reference/classes.html#inheritance) to allow subclasses. In Go, [methods](https://golang.org/ref/spec#Method_declarations) are declared separately from [structs](https://golang.org/ref/spec#Struct_types), which only have a limited form of inheritance via [embedding](https://golang.org/doc/effective_go.html#embedding). And so on.

Really, I want to highlight the difference between the [imperative](https://en.wikipedia.org/wiki/Imperative_programming) and the [declarative](https://en.wikipedia.org/wiki/Declarative_programming)&mdash;doing vs being. Programming languages implement classes as a feature that *does* something for you in some specific way. But they only bother because of what a class *is* in the abstract, which doesn't change between languages. So what is a class, really?

My answer may seem underwhelming: a class can be understood as a [set](https://en.wikipedia.org/wiki/Set_%28mathematics%29) of all its possible instances.

![A Venn diagram of the RGB set with elements corresponding to the red, yellow, green, blue, indigo, and violet instances we created previously.](https://user-images.githubusercontent.com/600924/80325991-346fdf80-87ec-11ea-9897-9f366491aeda.png)

For illustration, the `RGB` class contains not just the [ROYGBIV](https://en.wikipedia.org/wiki/ROYGBIV) instances we created before, but also every other valid instance. Using just the red/blue/green components, that's 256^3 = 16,777,216 possible combinations. Along with the alpha channel, which is a real number between 0 and 1, there is an [uncountably infinite](https://en.wikipedia.org/wiki/Uncountable_set) number of elements in this set (although computers can only represent a finite range of floating point numbers in practice).

The exciting part of pulling on this thread is that it leads us to the very [foundations of mathematics](https://en.wikipedia.org/wiki/Foundations_of_mathematics). Virtually all mathematical theorems can be formulated as theorems of [set theory](https://en.wikipedia.org/wiki/Set_theory). Thanks to over a century of work by mathematicians, we know a wide variety of abstract properties about sets. Things that are true not because of a specific implementation, but because they *must* be true.

## Inheritance unwoven

One such truth: if classes are sets, then subclasses are [subsets](https://en.wikipedia.org/wiki/Subset). One set is a subset of another if every element of the first is also an element of the second. In mathematical notation, we write $x \in S$ if element $x$ is a member of set $S$ and we write $S \subseteq T$ if set $S$ is a subset of set $T$. So $S \subseteq T$ if for every $x \in S$ it's the case that $x \in T$ as well.

When an object-oriented language says "everything is an object", one thing they often mean is that every class inherits from some base `Object` superclass. Considering all the classes we've defined so far, we can draw the Venn diagram showing how these sets nest.

![A Venn diagram showing the nesting of the classes we've defined in this post.](https://user-images.githubusercontent.com/600924/80559926-fad9d880-8993-11ea-83c3-047fb442d51a.png)

Elements of the `Object` set include the elements of the `Rainbow` and `Color` sets. Inside of `Color` are all its possible instances; some of those instances are also in the `RGB` and `Cylindrical` sets. Inside `Cylindrical` are `HSL` and `HSV`.

In mathematical notation:

<div>
$$
  \begin{aligned}
    \texttt{Rainbow} &\subseteq \texttt{Object} \\
    \texttt{Color} &\subseteq \texttt{Object} \\
    \texttt{RGB} &\subseteq \texttt{Color} \\
    \texttt{Cylindrical} &\subseteq \texttt{Color} \\
    \texttt{HSL} &\subseteq \texttt{Cylindrical} \\
    \texttt{HSV} &\subseteq \texttt{Cylindrical} \\
  \end{aligned}
$$
</div>

Now consider an instance of the `HSL` class for the color [pink](https://colorpicker.me/#FFC0CB).

```ruby
pink = HSL.new(350, 1.0, 0.88)
```

We can say that `pink` is in the `HSL` set. By the definition of subsets, this also means the following.

<div>
$$
  \begin{aligned}
    \texttt{pink} &\in \texttt{HSL} \\
    \texttt{pink} &\in \texttt{Cylindrical} \\
    \texttt{pink} &\in \texttt{Color} \\
    \texttt{pink} &\in \texttt{Object} \\
    \texttt{pink} &\notin \texttt{Rainbow} \\
    \texttt{pink} &\notin \texttt{RGB} \\
    \texttt{pink} &\notin \texttt{HSV} \\
  \end{aligned}
$$
</div>

Given these set memberships, and the fact that classes are sets of instances, this means `pink` is *an instance* of `HSL`, `Cylindrical`, and `Color` simultaneously.

### Methods

These interactions become important when we consider what methods mean through the lens of subsets. Say we add a method to the `RGB` class that returns the corresponding [complementary color](https://en.wikipedia.org/wiki/Complementary_colors).

```ruby
class RGB < Color
  def complement
    self.class.new(255 - red, 255 - green, 255 - blue, alpha)
  end
end
```

We think of this method as taking no arguments. After all, we pass the message to an instance of `RGB` with no parameters.

```ruby
red = RGB.new(255, 0, 0)
cyan = red.complement # RGB 0, 255, 255
```

If we try to call this method on some other class that does not define it, we get an error.

```ruby
red = HSL.new(0, 1.0, 0.5)
red.complement #=> NoMethodError
```

Imagine for a moment that the language didn't have this concept of methods, though: just top-level functions. The spelling might be different, but you could achieve the same effect. In our example, if an instance is in the `RGB` class, we can use the logic as defined; otherwise raise an exception.

```ruby
def complement(o)
  case o.class
  when RGB
    o.class.new(255 - o.red, 255 - o.green, 255 - o.blue, o.alpha)
  else
    raise NoMethodError
  end
end
```

A method is then a function whose first argument is tacitly an instance of the class on which it's defined. This is made more evident in languages like Python, where methods are always defined with an explicit first argument named `self`.

But conceptually, using just one function gets more complicated. Methods with the same name might be defined for several different classes. So if we only have this one function, it needs to have a branch for each class on which it's defined.

Say we added an equivalent method for `Cylindrical`.

```ruby
class Cylindrical < Color
  def complement
    self.class.new((hue + 180) % 360, saturation, component, alpha)
  end
end
```

The top-level function would need an extra clause for the `Cylindrical` case.

```ruby
def complement(o)
  case o.class
  when RGB
    o.class.new(255 - o.red, 255 - o.green, 255 - o.blue, o.alpha)
  when Cylindrical
    o.class.new((o.hue + 180) % 360, o.saturation, o.component, o.alpha)
  else
    raise NoMethodError
  end
end
```

But this implementation still **incorrect**, thanks to inheritance. In the above code, we're checking set *equality*. As was pointed out in the previous section, an instance can be a member of multiple sets at once, even if the sets aren't equal. For example, even though an instance of `HSL` is also an instance of `Cylindrical`, $\texttt{HSL} \ne \texttt{Cylindrical}$. Yet the actual method is equally valid to call on subclasses of `Cylindrical`.

```ruby
red = HSL.new(0, 1.0, 0.5)
cyan = red.complement # HSL 180, 1.0, 0.5

green = HSV.new(120, 1.0, 0.50)
purple = green.complement # HSV 300, 1.0, 0.5
```

We know classes are sets. So instead of checking if the object's class is *equal* to the definer's class, what if we perform a subset check, $\subseteq$? In Ruby, you can actually compare two classes this way using the `<=` operator.

```ruby
def complement(o)
  case
  when o.class <= RGB
    o.class.new(255 - o.red, 255 - o.green, 255 - o.blue, o.alpha)
  when o.class <= Cylindrical
    o.class.new((o.hue + 180) % 360, o.saturation, o.component, o.alpha)
  else
    raise NoMethodError
  end
end
```

Finally, this is a correct definition for our complementary color method. Languages don't necessarily implement methods this way, but that doesn't invalidate the conceptual understanding.

### Method overriding

Things get more complicated when we throw [method overriding](https://en.wikipedia.org/wiki/Method_overriding) into the mix. This allows a subclass to redefine a method of their parent class. The subclass can usually delegate to the original superclass method with some form of `super` keyword, but this feature complicates our discussion too much for the scope of this post. I encourage you to destroy that rainbow for yourself. 😉

One common (though admittedly not particularly interesting) pattern in Ruby, which lacks explicit [abstract classes](https://en.wikipedia.org/wiki/Class_%28computer_programming%29#Abstract_and_concrete), is to define a method that raises `NotImplementedError` on the superclass. Subclasses are expected to override the method, because otherwise they'll inherit the error-throwing version. This is desirable simply for the fact that `NotImplementedError` is more informative than `NoMethodError`.

For example, we might expect to be able to convert cylindrical-coordinate colors into RGB points. However, the implementation details vary based on the specific cylindrical model.

```ruby
class Cylindrical
  def to_rgb
    raise NotImplementedError
  end
end

class HSL < Cylindrical
  def to_rgb
    puts '(HSL to RGB algorithm)'
  end
end

class HSV < Cylindrical
  def to_rgb
    puts '(HSV to RGB algorithm)'
  end
end
```

If we subclass `Cylindrical` without overriding the method, we're tipped off to the fact that we didn't implement something we should have.

```ruby
class HSI < Cylindrical
  alias intensity component
end

magenta = HSI.new(300, 1.0, 0.67)
magenta.to_rgb #=> NotImplementedError
```

Once again, for the sake of illustration, let's think about how we would define this method as a function that compares subsets. This isn't how languages necessarily implement it, but it helps us understand how we might get the logic wrong.

```ruby
def to_rgb(o)
  case
  when o.class <= Cylindrical
    raise NotImplementedError
  when o.class <= HSL
    puts '(HSL to RGB algorithm)'
  when o.class <= HSV
    puts '(HSV to RGB algorithm)'
  else
    raise NoMethodError
  end
end
```

The issue is that our big conditional statement is sensitive to the order of the branches. In the **incorrect** implementation above, the `Cylindrical` subset check is done first. So say we pass an `HSL` instance into this top-level function. Since $\texttt{HSL} \subseteq \texttt{Cylindrical}$, we'll immediately execute the branch that raises the `NotImplementedError`.

```ruby
coral = HSL.new(5, 0.90, 0.72)
to_rgb(coral) #=> NotImplementedError
```

So we need to order the branches correctly. How do we know we can even find such an ordering? Because of set theory!

Firstly, a [partial order](https://en.wikipedia.org/wiki/Partially_ordered_set) of a given set is a relationship between any two of its elements such that the following properties hold.

* **Reflexivity**: every element is related to itself.
* **Antisymmetry**: any two distinct elements related to each other in one direction cannot be related in the other direction. Said another way, the only way two elements can be related in both directions is if they're actually the same element.
* **Transitivity**: if one element is related to a second element and the second is related to a third, then the first is related to the third as well.

For example, the set of integers is partially ordered by the "less than or equal to" relationship.

* Every integer $x$ is less than or equal to itself: $x \le x$.
* If $x \le y$ and $y \le x$, then $x = y$.
* If $x \le y$ and $y \le z$, then $x \le z$.

So, too, is the set of *classes* (in Ruby, this would be the `Class` class) partially ordered by the "is a subset of" relationship, $\subseteq$.

* Every set is a subset of itself: $A \subseteq A$.
* If $A \subseteq B$ and $B \subseteq A$, then $A = B$.
* If $A \subseteq B$ and $B \subseteq C$, then $A \subseteq C$.

But a key difference between $\le$ and $\subseteq$ is that "less than or equal to" is a [total order](https://en.wikipedia.org/wiki/Total_order) on its corresponding set (i.e., integers). This means the relationship satisfies one additional property.

* **Connexity**: any two elements must be related in some direction.

Given any two integers $x$ and $y$, either $x \le y$ or $y \le x$. That is, integers are always comparable.

However, the subset relationship is **not** a total order. Although some sets are subsets of others, there are [disjoint sets](https://en.wikipedia.org/wiki/Disjoint_sets) with nothing in common. We can't say `HSL` is a subset of `HSV` or vice versa; `RGB` is not a subclass of `Cylindrical`, nor is `Cylindrical` a subclass of `RGB`. So classes aren't always comparable.

This matters because we'd like to order our conditional branches correctly. We must be able to sort classes such that the "smallest", most specific subset is first&mdash;so that `HSL` & `HSV` come before `Cylindrical`. But if $\subseteq$ is not a total order, how can we sort classes relative to each other? What order do `HSL` & `HSV` go in, since they're not comparable?

The answer is not to use a traditional [comparison sorting](https://en.wikipedia.org/wiki/Comparison_sort) algorithm that CS majors learn so much about in school. Rather, we can use [topological sorting](https://en.wikipedia.org/wiki/Topological_sorting#Relation_to_partial_orders). Theory tells us that for any partial order there exists at least one topological ordering. Incomparable elements may appear in any order relative to each other, as long as they come before/after comparable elements as appropriate.

Any such ordering is sufficient for our current example. One possible topological ordering puts `HSL` before `HSV`.

```ruby
def to_rgb(o)
  case
  when o.class <= HSL
    puts '(HSL to RGB algorithm)'
  when o.class <= HSV
    puts '(HSV to RGB algorithm)'
  when o.class <= Cylindrical
    raise NotImplementedError
  else
    raise NoMethodError
  end
end
```

Just as valid would be to put `HSV` before `HSL`. As long as they both come before their superset `Cylindrical`, the topological ordering keeps us from executing the wrong logic.

```ruby
def to_rgb(o)
  case
  when o.class <= HSV
    puts '(HSV to RGB algorithm)'
  when o.class <= HSL
    puts '(HSL to RGB algorithm)'
  when o.class <= Cylindrical
    raise NotImplementedError
  else
    raise NoMethodError
  end
end
```

This might seem trivial. Our classes all have one immediate superset, so we can readily see this "straight line" topological order from `HSV` to `Cylindrical` to `Color` to `Object`. Never fear: there's still more rainbow left to untangle.

### Multiple inheritance

Where the topological order matters more is in when a subclass can have several superclasses, as is the case with [multiple inheritance](https://en.wikipedia.org/wiki/Multiple_inheritance).

Languages don't seem to advertise their multiple inheritance loudly. If you ask me, it's a case of [no true Scotsman](https://en.wikipedia.org/wiki/No_true_Scotsman). Many languages have some *form* of multiple inheritance, they just won't call it that. They implement different limitations & algorithms to handle some of the ambiguities that can arise, making their flavor of multiple inheritance different from "pure" multiple inheritance&mdash;whatever that means. As a result, trying to understand the concept in terms of any one language will be difficult: Ruby's approach will lead you one way while Python's will lead you another. Once again, it serves us to think about the topic in the abstract (and not too pedantically).

In terms of sets, multiple inheritance declares that a class is a subset of the [union](https://en.wikipedia.org/wiki/Union_%28set_theory%29) of multiple superclasses. The union of sets $S$ and $T$, denoted $S \cup T$, is a set such that if $x \in S$ or $x \in T$, then $x \in S \cup T$.

In Ruby terms, you can't inherit from multiple classes. But you *can* [mix-in](https://en.wikipedia.org/wiki/Mixin) multiple modules. In the language, modules differ from classes in that they don't have instances. That is, you can't call `M.new` for a module `M`. However, the two are still very much related. In fact, `Class` is a subclass of `Module` in Ruby. Realizing this, it's not a stretch to think that modules are still sets. If you can conceptualize classes as sets of instances, a module might be a set of objects that respond to certain methods. Despite the technical differences, the conceptual effect is the same: the child acquires traits from multiple parents.

So you want a convoluted example? You've got it! We'll first define modules how we normally would in Ruby, then think again about how methods could theoretically be implemented as top-level functions.

Let's say we have some shared functionality for objects that are suitable subjects of poetry. According to Keats, this might include an angel's wings, gnomed mines, and of course rainbows. For us to write a poem about any given object, we might need to compute properties about it, like words that rhyme with the object's name or the source of its artistic beauty.

```ruby
module Poetic
  def rhymes
    puts '(Poetic rhymes algorithm)'
  end

  def beauty
    puts '(Poetic beauty algorithm)'
  end
end
```

While angels, gnomes, and rainbows aren't subclasses of each other, we can factor out their shared poetic functionality.

Compare this to cold, calculating science. In science, we care about objects that we can measure. As Keats laments, all the object's mysteries will be conquered by rule and line. But to a scientist, the beauty is in knowing.

```ruby
module Scientific
  def measurements
    puts '(Scientific measurements algorithm)'
  end

  def beauty
    puts '(Scientific beauty algorithm)'
  end
end
```

Any number of objects might have scientific functionality, but we care primarily about rainbows. Moreover, a rainbow is both a poetic *and* a scientific phenomenon.

```ruby
class Rainbow
  include Poetic
  include Scientific
end
```

The spelling might be different from subclassing, but this declares that `Rainbow` is a subset of the union of `Poetic` and `Scientific`.

![A Venn diagram showing conceptual multiple inheritance, where the Rainbow set overlaps the union of the Poetic & Scientific sets.](https://user-images.githubusercontent.com/600924/80845191-66eb5500-8bbd-11ea-908c-dbc4dde1f835.png)

In mathematical notation, $\texttt{Rainbow} \subseteq \texttt{Poetic} \cup \texttt{Scientific}$. In Ruby, you can even compare the sets the same way as before.

```ruby
Rainbow <= Poetic #=> true
Rainbow <= Scientific #=> true
```

Defining the functional logic of the unique methods is simple.

```ruby
def rhymes(o)
  case
  when o.class <= Poetic
    puts '(Poetic rhymes algorithm)'
  else
    raise NoMethodError
  end
end

def measurements(o)
  case
  when o.class <= Scientific
    puts '(Scientific measurements algorithm)'
  else
    raise NoMethodError
  end
end
```

However, what do we do about the `beauty` function, whose method is defined for both `Scientific` and `Poetic`? Even though a topological sort would put `Rainbow` before either of these modules, `Scientific` and `Poetic` themselves are incomparable. So the behavior is dependent on the order.

```ruby
def beauty(o)
  case
  when o.class <= Poetic
    puts '(Poetic beauty algorithm)'
  when o.class <= Scientific
    puts '(Scientific beauty algorithm)'
  else
    raise NoMethodError
  end
end

rainbow = Rainbow.new([red, orange, yellow, green, blue, indigo, violet])
beauty(rainbow) #=> (Poetic beauty algorithm)
```

With `Poetic` before `Scientific`, the poetic algorithm is used on the `Rainbow` instance.

```ruby
def beauty(o)
  case
  when o.class <= Scientific
    puts '(Scientific beauty algorithm)'
  when o.class <= Poetic
    puts '(Poetic beauty algorithm)'
  else
    raise NoMethodError
  end
end

rainbow = Rainbow.new([red, orange, yellow, green, blue, indigo, violet])
beauty(rainbow) #=> (Scientific beauty algorithm)
```

With `Scientific` before `Poetic`, the scientific algorithm is used on the `Rainbow` instance.

In a way, our topological sort has failed us: either of the above could be valid. This is where language implementations resolve the ambiguity in different ways&mdash;sometimes even as full-fledged features.

* In Ruby, classes may mix in multiple modules. [`include`](https://ruby-doc.org/core-2.7.1/Module.html#method-i-include) inserts the module directly after the current class in the topological order, so a more recent `include` has higher precedence than a less recent one. [`prepend`](https://ruby-doc.org/core-2.7.1/Module.html#method-i-prepend) inserts a module directly before the current class in the topological order, which allows for some [interesting patterns](http://gshutler.com/2013/04/ruby-2-module-prepend/).
* In Common Lisp ([CLOS](https://en.wikipedia.org/wiki/Common_Lisp_Object_System)), [classes](http://www.gigamonkeys.com/book/object-reorientation-classes.html) may have multiple parents. [Generic functions](http://www.gigamonkeys.com/book/object-reorientation-generic-functions.html) sort applicable methods with tie-breakers based on the declaration order. This list can be applied in [standard](http://www.lispworks.com/documentation/HyperSpec/Body/07_ffac.htm) or [customized](http://www.lispworks.com/documentation/HyperSpec/Body/m_defi_4.htm#define-method-combination) ways.
* In Scala, classes may mix in multiple [traits](https://docs.scala-lang.org/tour/traits.html). The spec defines a standard [class linearization](https://www.scala-lang.org/files/archive/spec/2.13/05-classes-and-objects.html#class-linearization) that winds up working similarly to Ruby's, where the order of declarations matters.
* In Python, classes may inherit from any number of parent classes. The method resolution order (MRO) is given by the [C3 linearization algorithm](https://www.python.org/download/releases/2.3/mro/).
* In [Factor](https://factorcode.org/), ambiguities in the [class linearization](https://docs.factorcode.org/content/article-class-linearization.html) are resolved through a couple different tie-breaking rules based on [metaclasses](https://en.wikipedia.org/wiki/Metaclass) & lexicographic order.
* [And so on.](https://en.wikipedia.org/wiki/Multiple_inheritance#Mitigation)

There's no right answer. But knowing the abstract problem helps you to better understand the algorithms used by your favorite language.

### Types

I'd be remiss not to mention [type theory](https://en.wikipedia.org/wiki/Type_theory), since we're used to dealing with various notions of types in programming languages.

If you dig into set theory and the foundations of mathematics, you'll find various equivalent formalisms. While there is ostensibly a [difference between a subclass and a subtype](https://softwareengineering.stackexchange.com/questions/362316/whats-the-difference-between-a-subclass-and-a-subtype), in practice I think it's largely pedantic. At any rate, the *theories* of sets and types are [equivalent](https://plato.stanford.edu/entries/type-theory/#TypeTheoTheo). That being the case, it's still useful for programmers to think of a classes as sets, sets as types, and thus classes as types.

For example, a practical concept that falls out of this kind of thinking is the difference between [has-a](https://en.wikipedia.org/wiki/Has-a) and [is-a](https://en.wikipedia.org/wiki/Is-a) relationships. Intuitively, a rainbow is not a type of color, but RGB is. However, a rainbow contains many colors. Thus, `Rainbow` should use a has-a (or really, has-many) relationship to `Color` by keeping objects in instance variables, whereas `RGB` should use an is-a relationship by subclassing `Color`. But you don't even need to know the buzzwords if you can think critically about the types (and therefore classes) involved in your program. Doing so means you're actively thinking about how to write good code. And that's a skill transferable to any language, whatever the subclassing (or subtyping) mechanisms might be.

This also means that despite whatever misconceptions people may have, even a dynamic language like Ruby has types. After all, it has classes. Just because one language doesn't use the same type-checking algorithms as another, it doesn't mean that types cease to exist. They're there abstractly even if your language doesn't call them out by name. As such, it still behooves you to think about types in a general sense, whether you're writing dynamically-typed PHP or statically-typed Java.

Conversely, even [functional](https://en.wikipedia.org/wiki/Functional_programming) languages that we don't normally think of as object-oriented have the same theoretical underpinnings. Instead of full-blown classes you might see a construct like [Elixir's `defstruct`](https://elixir-lang.org/getting-started/structs.html), which defines a type with specific fields but no methods. Or there may be very sophisticated ways of defining custom types, as in [Haskell's algebraic data types](http://learnyouahaskell.com/making-our-own-types-and-typeclasses). Such types might include something like [Racket's anonymous unions](https://docs.racket-lang.org/ts-guide/types.html#%28part._.Union_.Types%29), allowing us to formulate multiple inheritance in terms of type constraints. And as we've seen, rather than attach methods to specific classes, you could define functions that dispatch based on the type of their first argument. But why stop there? A system like [CLOS](https://en.wikipedia.org/wiki/Common_Lisp_Object_System) can use [multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch) to select the most specific function to apply to the *combination* of parameters. Of course, you could also be [OCaml](https://ocaml.org/releases/4.10/htmlman/index.html) and fuse object orientation with the type system & syntax of the functional language [ML](https://en.wikipedia.org/wiki/Standard_ML).

Programming languages vary widely, but the concepts can still be understood in general terms. If you understand the general, then learning any specific language&mdash;and writing good code in it&mdash;becomes that much easier.

## Rainbows unwoven

So there it is. Classes & inheritance analyzed to death; the underlying theory exposed. Armed with this knowledge, what have you lost? If you ask me, not the beauty, but the preconceptions.

* Object orientation isn't at odds with functional programming. Either can be understood (at least at a high level) in terms of the other.
* Mixins, multiple inheritance, and union types are more similar than they are different.
* Even among object-oriented languages, one system isn't fundamentally different from another. They're all different flavors of set theory.

A major benefit of abstraction is unifying ideas into an elegant whole that can be kept in your head. If you understand that classes are sets, and you already know how sets behave, you're capable of grokking a wide variety of object-oriented systems. Language becomes an implementation detail.

But despite what you might automatically think that means, implementation *does* matter. Each object system is beautiful in some way. Taking advantage of your language's strengths makes your job easier as a programmer. Playing to these strengths lets you organize your code in beautiful ways. What could be more important?

> [W]e want to establish the idea that a computer language is not just a way of getting a computer to perform operations but rather that it is a novel formal medium for expressing ideas about methodology. Thus, programs must be written for people to read, and only incidentally for machines to execute.

&mdash; [Structure and Interpretation of Computer Programs](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-7.html)

So go forth and destroy some rainbows. You never know what you might learn. 🌈
