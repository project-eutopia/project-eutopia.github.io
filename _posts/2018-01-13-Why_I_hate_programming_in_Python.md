---
layout: post
draft: true
title: Why I hate programming in Python
---

This post will discuss all the parts of Python that have bothered me after using it significantly over the past few years.
Before we get into details, I would like to note that I actually think Python is a very useful and powerful programming language.
Here are some parts of Python I particularly like.

- It is an interpreted language and interactive Python (e.g. `ipython`, or interactive debugging with `IPython.embed()` or `code.interact(local=locals())`) allows for powerful problem solving and debugging to be done.
- There are a great many fully featured and well supported libraries out there ([`numpy`](https://www.numpy.org), [`scipy`](https://www.scipy.org/), [`matplotlib`](https://matplotlib.org/)).

I actually use Python quite a lot for my own research.
However I also do lots of programming in Ruby, and I feel that this has spoiled myself.
At this point I can only see Python as an old, out-dated language when working on it.
In this post I try to enumerate some of my criticisms toward Python that have accumulated.
I don't expect anything to be done about these criticisms, as changing most of them would require non-backwards compatible changes that I am sure the Python community would not be in favor of.
Furthermore, I am not trying to advocated against people using Python, because if you can live with these idiosyncrasies then Python can be a very powerful tool to do a great many things, whether it is building a website, analyzing complex data sets, or writing a program to automate your smart devices.
That said, for those starting out on a potentially large project, I think that Python is only an appropriate tool to use in the initial prototyping stages.


## 1 -- Does not feel object-oriented

Python is supposed to be an object-oriented programming language, yet it often feels like it is a C-like language masquerading as an object-oriented language.

### 1.1 -- `self` as first argument

In C, one common technique is to define `struct`s to store some data, and have a set of methods which take a pointer to this struct as a first argument.

```c
typedef struct {
  float x, y;
} Point;

void translate_point(Point* point, const Point* translation) {
  point->x += translation->x;
  point->y += translation->y;
}
```

In Python, emulating this behavior with a class would look like this

```python
class Point(object):
    def __init__(self, x, y): # <= why is `self` needed here?
        self.x = x
        self.y = y
    
    def translate(self, other): # <= why is `self` needed here?
        self.x += other.x
        self.y += other.y
```

One can see that the `translate` method here takes `self` as a first parameter.  However, why is it necessary to pass `self` as the first argument to each method here?  The internal justification for this is that `point.translate(other)` is translated to something like `type(point).translate(point, other)`.  Languages like C++, C#, Java, and Ruby do not require this parameter as it is implied, and its presence makes Python feel like the object-oriented capabilities was tacked on.

```ruby
# Ruby
class Point
    attr_reader :x, :y
    
    def initialize(x, y) # no `self`
        @x = x
        @y = y
    end
    
    def translate(other) # no `self`
        @x += other.x
        @y += other.y
    end
end
```

```c++
// C++
class Point {
    public:
        double x, y;
    
        Point(double x_, double y_) : x(x_), y(y_) {} // no `this`
        
        void translate(const Point& point) { // no `this`
            x += point.x;
            y += point.y;
        }
};
```

Similarly, class methods in Python require a class parameter as the first argument:

```python
class PointFactory(object):
    @classmethod
    def build(klass, x, y): # <= Requiring class as 1st argument is verbose
        return Point(x, y)
```

### 1.2 -- `super`

Another place where Python feels like it falls a bit short is the `super` syntax.  It is way more complicated to do something as common as calling the super constructor than it has to be.  This difference is very apparent when you compare similar Python and Ruby code.

```python
class Animal(object):
    def __init__(self, name):
        self.name = name
    
    def make_noise(self):
        print("The animal named {} made noise".format(self.name))

class Dog(Animal):
    def __init__(self, name, breed):
        super(Dog, self).__init__(name) # <= That's a mouthful
        self.breed = breed
        
    def make_noise(self):
        print("The {} dog named {} barked".format(self.breed, self.name))
```

Notice in particular the line `super(Dog, self).__init__(name)`, which is extremely complicated.  One can simplify this by typing directly `Animal.__init__(name)`, but this is not DRY -- the sub-class should not need to directly reference the specific class it is inheriting from within the code because it is implied.  This equivalent line in Ruby is just `super name` as seen below.  The super class is already known by the class at the point that method is invoked.  Another niceity of Ruby is that if the arguments of a constructor or subclass method do not change, just typing `super` is enough.

```ruby
class Animal
    attr_reader :name
    
    def initialize(name)
        @name = name
    end
    
    def make_noise
        puts "The animal named #{name} made noise"
    end
end

class Dog
    attr_reader :breed
    
    def initialize(name, breed)
        super name
        @breed = breed
    end
    
    def make_noise
        puts "The #{breed} dog named #{name} barked"
    end
end
```

Part of the reason for why this is the case is because Python supports multiple inheritance, which greatly complicates object oriented programming.  I feel it is always better to stick with single inheritance, augmented with interfaces/abstract classes (C++ and Java) or with modules/mixins (see Ruby).

### 1.3 -- Lack of public, protected, private distinctions

When creating a class hierarchy, defining certain attributes/methods to be `public`, `protected` and `private` is a common and useful strategy.  While the exact definition of `protected` varies from language to language, the meaning of `public` and `private` are fairly universal.  Python only technically supports `private`, and has no support for `protected` whatsoever.

Python accomplishes making methods private by appending two underscores in the front of the name, for instance `def __check_validations(self)`.  Whenever this method is called inside the class, it works as expected `self.__check_validations()`, but outside this would not work: `foo.__check_validations()`.  This feature is implemented ad-hocly through name mangling;  if the class is `User`, then the actual method name becomes `_User__check_validations`.  This is a gross and verbose hack at adding support for private methods.


## 2 -- Plethora of double-underscore method names

In programming languages, appending `_` or `__` to the front of some name traditionally means "please stay away".  It is often used for denoting internals that should not be messed with by people who do not know exactly what they are doing.  This convention is partially reflected in the double-underscore naming convention for private methods in Python.  However, there are also plenty of methods that are extremely common which have the format `__foo__`!  Here is a partial list of ones that it is not unusual for you to come across a need for (see also http://minhhh.github.io/posts/a-guide-to-pythons-magic-methods).

- `__init__`: This is the constructor of an object, and so you will be writing it very often when you write classes.
- Comparison operators: `__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__`, and `__cmp__`.  For instance, if you want your objects to be sortable, you will need to use these methods.
- Arithmetic operators: unary `__pos__` and `__neg__`, binary `__add__`, `__sub__`, `__mul__`, `__div__`, `__truediv__`, `__floordiv__`, `__and__`, `__or__`, and `__xor__`, in addition to those with an `r` in front to denote reflection (which is needed to allow something like `1 + foo` to work, where the first argument is not an instance of your class).
- String representations: `__str__` and `__repr__`.  The first is used to convert to a human readable string, while the second is used for instance by the command line interpreter to output a representation of objects.
- Attribute access:  `__getattr__`, `__setattr__`, `__hasattr__`, `__delattr__` for getting, setting, checking presence of, and deleting attributes of an object.
- Container behavior:  `__len__`, `__getitem__`, `__setitem__`, `__hasitem__`, `__delitem__`
- Iterators:  `__iter__`
- Calling like a function:  `__call__`
- Context managing:  `__enter__` and `__exit__`
- Main method through the idiom `if __name__ == "__main__":`, which is just plain verbose

Don't you all get tired typing all these underscores?  I sure do!  The reason for these long and awkward names is that it avoids name conflicts.  However there are other approaches that do not make so many underscores necessary.  For example, a language like C++ and C# uses the keyword `operator` to avoid conflicts when writing operator methods.  Ruby uses the `initialize` keyword for the constructor, the appropriate symbols for comparisons and arithmetic operators (`<`, `==`, `+`, etc), `coerce` to avoid a need for reverse operators like `__radd__`, `to_s` and `inspect` convention method names for string representation methods, allows you to define assignment operators like `name=` for attribute assignment which alleviates the need for the `__setattr__` like methods, container like behavior is superceded by strong support for iterators through blocks and allowing definitions of `[]` and `[]=` methods, the `Proc#call` method for function-like calling, and there is no need for context management due to blocks (e.g. `File.open("README.md", "r") {|f| puts f.read}`).


## 3 -- Global methods

Python has some global methods that are commonly used.  These are given the nice name "[built-in functions](https://docs.python.org/2/library/functions.html)".  However, since Python is an object-oriented language, I feel it makes more sense for many of these to be instance methods, or methods associated with a specific module.  Furthermore, by being global methods, these are essentially keywords which further restricts the names available for use (e.g. defining a `len` variable automatically makes the `len` method fail to work where the variable is in scope).  Shown here are some global method calls in Python, and better hypothetical ways to doing the same thing using instance methods or module methods. 

```python
abs(-2)
(-2).abs() # instance method
Math.abs(-2) # Math module class method

len([1,2,3])
[1,2,3].len() # instance method

chr(ord('a'))
'a'.ord().chr() # instance methods
```

`str`, `set`, `list`, `dict` are also types, so these "global functions" are really just constructors, which is a valid use-case.  Having the names be lower case is a bad choice though, as the official style guide itself even says classes should use camel-case: https://www.python.org/dev/peps/pep-0008/#class-names, so the chances of colliding with user defined variables is high.

Functional programming methods `filter`, `reduce`, `sorted`, `map`, and `reversed` make more sense as instance methods on iterable objects.  For example, `[1, 2, 3, 4].filter(lambda x: x % 2 == 0)` would be preferred to `filter(lambda x: x % 2 == 0, [1, 2, 3, 4])`.  This also makes code more readable, as the application of operations is in the order you read, left to right.


## 4 -- Whitespace and lack of block-end delimiters

One use of whitespace in code is to improve readability (e.g. indenting blocks so it is easy to see at a glance what the block consists of).  By forcing users to indent their code, this forces new programmers to develop better programming style habits.  However, this restriction is only useful for absolute beginners to programming languages, as any decent developer should know how to write readable code using indentation already so this is an unnecessary feature.

Related to this is the fact that there are no ending delimiters to blocks like conditionals and loops.  This means that, for example, if you have a code fragment that has had its whitespace mangled, it is literally impossible to know where whitespace should be.  When writing code in your favorite editor, there is often a way to fix indentation of code, either to bring everything into the same style (space vs tabs, number of spaces) or to quickly repair code (for instance if you delete surrounding namespace markers and want to have everything inside de-indent correctly automatically this can be a quick way of doing it).  This is not possible in Python.  I come across this very often while programming in Python.  For instance I will copy a useful line or block of code from some other file and paste it into the file I am currently working on, only to have the indentation be completely off.  And my trusty `=` vim command to re-align indentation does not work, so I have to manually increase/decrease the indentation to fit.  This problem does not exist in other languages which have ending delimiters.

This lack of ending delimiters also means that jumping to the end of a large block in your editor cannot be done simply.  For instance, using the `%` key in vim will jump to the matching brace, and using this on `{}` curly braces allows for quick jumping to the start/end of  `for` and `if` statements.  Doing this kind of code navigation requires an advanced Python-aware editor, and vim for instance does not do this out of the box.  It has the motions `]]` and `[[` for moving to next/previous class, and `]m`, and `[m` for methods, but what do you do about `if` statements and `while` blocks?


## 5 -- List comprehensions are backwards

List and dictionary comprehensions in Python are useful constructs for doing common manipulations.  For instance, to build a list of squares, you just type `[x**2 for x in range(10)]`.  Nice and simple and readable.  The concern I have with this is that it feels backwards, and readability I feel can be improved when you flip the order.

To see what I mean, consider the following Python and Ruby code which sums the squares of odd integers:

```python
# Python
sum([x**2 for x in [x for x in range(10) if x % 2 == 1]])
```

```ruby
# Ruby
(0...10).select {|x| x.odd?}.map {|x| x**2}.sum
```

The Ruby is much easier to read because reading things left-to-right exactly corresponds to the order of operations:  start with the numbers from 0 up to 10, then select the odd ones, then map them to their squares, then sum the result.  It requires a lot more mental work to dissect the Python code.  While I wrote the above Ruby code in one pass, when writing the Python list comprehension I wrote it in the following complicated steps from the inside out:

```python
                               range(10)
                   [x for x in range(10) if x % 2 == 1]
    [x**2 for x in [x for x in range(10) if x % 2 == 1]]
sum([x**2 for x in [x for x in range(10) if x % 2 == 1]])
```


## 6 -- Lack of multiline blocks

A common and useful feature of major modern programming languages is the ability to define anonymous functions/classes within code at the point of usage.  A prominent example of where this is useful is in user interface programming.  When a user clicks a button, it is necessary to write code that does something when the click occurs,.  This is called a callback.  In C++ this is done through lambda expressions, in Java through anonymous classes or lambda expressions, and in Ruby using blocks.

Python has support for lambdas, for example in `sorted([1, 2, 3, 4], key = lambda x: -x)` to sort numbers in reverse order, however these lambdas are limited to a single line.  What if you wanted a button callback to get the name field, pull out the first name, and then pop-up a message that says "Hello ${first_name}"?  In Ruby you could hypothetically do it like this:

```ruby
text_field = MyUI::TextField.new(label: "Full name")
button = MyUI::Button.new

# When the button is pressed, we will do more than just one thing,
# so the existence of a multi-line block is very convenient
button.on_press do
  first_name = text_field.text.split(" ").first
  MyUI::PopUp.new("Hello #{first_name}")
end
```

Multiline blocks also come in handy when doing functional programming (`map`, `filter`, `reduce`).