---
layout: post
title: 'Refresher on SOLID principles'
permalink: '/posts/refresher-on-solid-principles'
---

Object Oriented Programming (OOP) has been around for over four decades since Alan Kay and his colleagues introduced it to the world. Languages have flourished and died in it's wake imparting sound practices that stood the test of time.

Robert C. Martin (Uncle Bob), in the early 2000s, compiled a set of principles to write maintainable software and detect code smells. These eventually came to be collectively known as the SOLID principles.

SOLID is a mnemonic acronym that stands for:

* [Single Responsibility Principle](#srp)
* [Open Closed Principle](#ocp)
* [Liskov Substitution Principle](#lsp)
* [Interface Segregation Principle](#isp)
* [Dependency Inversion Principle](#di)

<h2 id="srp">Single Responsibility Principle (SRP)</h2>

A class must have only one responsibility. Often a good way to ensure this is to have a comment on the top that defines the intention of the class. If you think you need to use 'and' in the comment, there is a potential chance for a new class.

Consider the following system where you're modelling a library.

```
class Book
  attr_accessor :title, :content, :shelf_position
  //initialization logic

  def move_to(new_position)
    shelf_position = new_position
  end
end
```

Here apart from `Book` being spatially aware, the behaviour of how it should move around also resides in `Book`. We could have a situation later where a book could know how to `print` itself. In no time, your `Book` could turn into a monolith and unwieldy to manage.

In the above code sample, `Shelf` would be a good candidate to store book locations and house the behaviour of moving books.

```
class Book
  attr_reader :title, :content
end

class Shelf
  attr_reader :books
  //initializers

  def move_book(new_position, book)
    books[new_position] = book
  end
end
```

<h2 id="ocp">Open Closed Principle (OCP)</h2>

You must have heard that code should be open for extension but closed for modification. The gist of it is that rather than modifying existing objects, favour creating new objects. This leads to having a more stable application as your not changing existing contracts. If you spot decorators, presenters, policy objects and such, it's the mark of OCP.

Let's take our previous `Book` example. We have requirement to print books in a specific format. Rather than having a humungous print function in Book, this could be delegated to a BookPrinter object, like so.

```
class Book
  attr_accessors :content, :title

  def print
    BookPrinter.new(self).print
  end
end

class BookPrinter
  def initialize(book)
    @book = book
  end

  def print
    // printing logic goes here
  end
end
```

This keeps the API surface area for the book object clean as you don't have a proliferation of print function and their helpers sitting in every `Book` instance. In a lot of aways, it's a throw back to SRP.

<h2 id="lsp">Liskov Substitution Principle (LSP)</h2>

Anywhere you see inheritance, the first question you should consider is if it's possible to shift to a better composable approach. LSP is a good litmus test to see if your inheritance model is sound.

The general idea is to validate if the subtype (child class if it's inheritance) can be substituted in place of the parent type. This is applicable even for duck-typing approaches.

I'm going to demonstrate this with the classic example.

```
class Rectangle
  attr_accessors :width, :height
  //initialization

  def area
    width * height
  end
end

class Square < Rectangle
  attr_accessors :side

  def height=(side)
    @height = side
    @width = side
  end

  def width=(side)
    @height = side
    @width = side
  end

  def area
    side * side
  end
end

shapes = [Rectangle.new, Square.new]
shapes.each do |shape|
  shape.height = 4
  shape.height = 5
  puts shape.area
end
# 20 for rectangle - correct
# 25 for square - wut
```

Clearly inheritance has forced us to mutate Square's internal state (setters for height and width) which throws everything out of the window. Conceptually, Square may be a Rectangle but the code clearly defines two separate contracts. If you see `if` checks, for a particular subtype if your method calls, that could potentially be a violation of LSP.

<h2 id="isp">Interface Segregation Principle (ISP)</h2>

The idea is a class should require only required behaviour and nothing more. This sort of ties into languages with interface constructs, as you inherit the complete API when you implement the interface in your class. Let' see how this can affect our `Book` example.

```
module Utils
  def title_cased(text)
    text.titlecase
  end

  def log(message, level)
    Logger.send(level, message);
  end
end

class Book
  include Utils

  attr_accessors :content, :title
end
```

Book might need only title_cased behaviour but mixing in Utils brings in logger to. Better split into separate interfaces and require only what's needed.

```
module Utils
  module TextFormatter
    def title_cased(text)
      text.titlecase
    end
  end

  module Logger
    def log(message, level)
      Logger.send(level, message);
    end
  end
end
```

<h2 id="dip">Dependency Inversion Principle (DI)</h2>

I'll dive right into the code by taking our Book example. From OCP we've seen, the benefits of splitting out printer object.

```
class Book
  attr_accessors :content, :title

  def print
    BookPrinter.new(self).print
  end
end
```

On the outset, this looks clean, but we can clearly see that `Book` is strongly coupled with `BookPrinter`.

Let's fix that.

```
class Book
  attr_accessors :content, :title

  def print(printer = BookPrinter)
    printer.new(self).print
  end
end
```

This introduces the flexibility of injecting other custom printer classes into `Book`. Test driving your code forces you in a lot of ways to follow DI.

## Conclusion

Following the guidelines outlined in these principles is a good way forward in writing maintainable code. This does not mean you should religiously stick to them. There might be situations where it's warranted in breaking a few. You're good as long as you don't lose the big picture.

If you find any of the examples above or my understanding of something atrociously wrong, do drop a comment.
