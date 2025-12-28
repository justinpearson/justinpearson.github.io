---
layout: base-layout
nav_name: projects
extra_css_files:
  - code-syntax-highlight.css
---

# Grumpy Justin's Ruby Tutorial
{:.no_toc}

The Ruby programming language has some weird quirks.

I wrote this for experienced programmers who are new to Ruby, who just need to know the "diff" between, say, Python and Ruby --- the syntax, idioms, and pitfalls. Most Ruby tutorials annoyingly and pedantically assume the reader has no knowledge of programming. (In my opinion, if you have no knowledge of programming, learn Python first: there are fewer oddities and pitfalls.)

Lastly, this is NOT a Ruby On Rails tutorial. In my line of work, many folks come to Ruby because they need to learn Ruby On Rails. It is immensely frustrating trying to learn **both** the Ruby language and also the Rails framework at the same time. Rails is overflowing with strange conventions you must follow, and under the hood, it uses Ruby's weirdest aspects to full advantage. Consequently, newcomers are assaulted with an endless onslaught of baffling error messages because some innocent unwitting typo in Ruby's strange syntax disrupted some subtle machination in Rails. Therefore it is worthwhile to study Ruby in isolation.

{% comment %}
    Table of contents powered by:
    https://ouyi.github.io/post/2017/12/31/jekyll-table-of-contents.html
{% endcomment %}

## Contents
{:.no_toc}

* TOC - need this 1st element to 'seed' the :toc generation
{:toc}


## General tips

### Whitespace doesn't matter

Unlike Python, Ruby doesn't care about whitespace:

```ruby
def f(x,y)
    s = x+y
        return s
            end

f(1,2)  #  => 3
```

### Printing

- Ruby's `print` is like Python's `print`, except that it doesn't print a trailing newline.
- Ruby's `puts` is bad, because it prints the value `nil` (like Python's `None`) as empty-string, which is confusing when debugging.
- Ruby's `pp` is like Python's `pprint` ("pretty-print"), but requires `require 'pp'` before Ruby 2.5.
- Ruby's `p` is best for debugging: it prints its argument **and returns it**, thus facilitating debugging because you can simply jam `p` before an expression to print it without changing the value of the expression:

```ruby
def f(x,y)
    s = p x+y   # print x+y, and set s to x+y
    return s
end

f(1,2)   # prints 3, in addition to f returning 3
```

This leverages the surprising fact that function calls do not require parentheses to delimit their arguments, as discussed in [this section](#parens-are-optional-in-a-function-call). That is, `p x+y` is parsed as `p(x+y)`, but `p x+y` is faster for quick debugging because you don't have to write the close-paren.


### Help docs

You will often have an object and want to know something about it.

In Python, at the interpreter, you can say `dir(x)` to list the methods, then use `help(x.f)` to see the docs for method `f`:

```
>>> x = [1,2,3]

>>> dir(x)
[ '__add__', '__class__', '__contains__', '__delattr__', '__delitem__', 
  ... ignore double-underscore methods ...
 'append', 'clear', 'copy', 'count', 'extend', 'index', 
 'insert', 'pop', 'remove', 'reverse', 'sort' ]

>>> help(x.append)

# append(object, /) method of builtins.list instance
#    Append object to the end of the list.
```

In Ruby, sadly, it is not that simple. 

The first stumbling block is that you cannot assume your version of Ruby has the docs installed, so you might have to install them yourself, see [here](https://stackoverflow.com/questions/3178900/how-do-i-install-the-ruby-ri-documentation/13886612#13886612).

The next stumbling block is that there are two versions of Ruby documentation: the command-line utility `ri` ("ruby info"), and the function `help` inside IRB, and you will get errors if you use one in the wrong environment:

```
WRONG:

$ help('Array.min')     # bash: syntax error near unexpected token `'Array.min''
$ irb
irb> ri 'Array.min'     # undefined method `ri'


RIGHT:

$ ri 'Array.min'         #  "Returns the object in ary with the minimum value. ..."
$ irb
irb> help('Array.min')   #  "Returns the object in ary with the minimum value. ..."
```

The next stumbling block is that the Rails framework is often installed without the documentation, so even if you have the Ruby docs installed, it won't be able to find docs for Rails functions. 

The *next* stumbling block is that it's hard to find the relevant methods of an object, because unfortunately, `x.methods()` lists ALL methods, including weird ones inherited from `Object`. It's hopeless to try to deduce what a `String` can do when you're faced with literally 183 methods, most of which are unrelated to strings:

```ruby
'asdf'.methods()  #  => [ :encode, :include?, :%, :*, :+, ...
                  #        ... 183 methods ...
                  #       :singleton_methods, :protected_methods, :private_methods, :!, :equal?,
                  #       :instance_eval, :instance_exec, :!=, :__id__, :__send__ ]
```

(In Ruby's defense, the methods are listed in the order in which they're inherited, so the string-related ones are typically listed first. But they are not alphabetical (unlike Python), so you can't quickly find a method by name.)

Also, and this is a minor point, but the methods are presented as a list of symbols: the leading colon in `:encode` indicates that it's a symbol literal. Symbols are an important part of Ruby, and we discuss them in [this section](#symbols). But they are very foreign to newcomers. It is unfortunate that a newcomer can't even find the methods of an object without having to confront this strange new datatype.

But, back to the problem of finding the relevant methods for an object.

There are two workarounds to remove the irrelevant methods from the list. The easiest and hackiest way is simply to "subtract" away the methods added by `Object`:

```ruby
x = 'asdf'
x.methods - Object.methods   # Lists 119 methods, instead of 183.
```

The fancier way (but harder to remember) is to ask the object's class what its instance methods are, passing the enigmatic argument `false` to specify that superclass methods shouldn't be listed:

```ruby
x = 'asdf'
x.methods
x.class     # String
x.class.instance_methods    # same as x.methods
x.class.instance_methods(false)   # Good: [:encode, :include?, 
                                  #          ... 128 methods ...
                                  #        :unicode_normalize, :encode!, :unpack, :unpack1]
```


Once you find the method of interest `x.foo`, you are ready to encounter the **next** stumbling block when you type `help(x.foo)` (like Python). This doesn't work because `x.foo` actually **calls** the `foo` method because, incredibly, parens are optional in function calls (discussed in [this section](#parens-are-optional-in-a-function-call)). The result is that `help` shows NOT the docs for the `foo` method, but the docs for whatever `x.foo` returns! (Very confusing!)

```ruby
x = 'asdf'
x.encode               #  => 'asdf'
help(x.encode)         # NO: calls x.encode, and therefore runs help('asdf') :(
```

So you have to call `help` with the class and method *as a string*, in a special format, like this:

```ruby
x = 'asdf'
x.class                # String
help('String#encode')  # YES: "The first form returns a copy of str transcoded to ..."
```




### Debug via breakpoints

It is insanely useful to set breakpoints in your code. 

Unfortunately, Ruby's debugging-breakpoint support is not as clean as Python's [pdb](https://docs.python.org/3/library/pdb.html): Ruby has multiple debugging libraries, and the most official one, `pry`, doesn't support single-line stepping --- only breaking and examining variables. To get feature-parity with Python's `pdb`, use Deivid Rodriguez's [`pry-byebug`](https://github.com/deivid-rodriguez/pry-byebug), which combines the two debugging utilities `pry` and `byebug`. And even then, you don't get the shortcuts `n`, `s`, `c`, `f` for `next`, `step`, `continue`, and `finish`; for those, you need to create a `~/.pryrc` file and dump some special Ruby code in it (see his Readme for details).

For simplicity, here's how to use the basic `pry` gem ([Github link](https://github.com/pry/pry)).

Install globally:

```
$ gem install pry
```

... or install in your project (if using Bundler):

```
# Your project's Gemfile

gem 'pry', '~> 0.12.2'  # see Github link for current version #
```

(... don't forget to `bundle install` afterward.)

Set breakpoints:

```ruby
# File prog.rb

def f(x,y)
    puts('Inside f!')
    require 'pry'; binding.pry
    s = x+y
    puts("Now s=#{s} !!")
    require 'pry'; binding.pry
    return s
end

puts('About to call f!!!')
f(1,2)
puts('Bye!')
```

Run your program to hit the first debug point:

```
~ $ ruby prog.rb 
About to call f!!!
Inside f!

From: /Users/justin/prog.rb @ line 5 Object#f:

     3: def f(x,y)
     4:     puts('Inside f!')
 =>  5:     require 'pry'; binding.pry
     6:     s = x+y
     7:     puts("Now s=#{s} !!")
     8:     require 'pry'; binding.pry
     9:     return s
    10: end

[1] pry(main)> 
```

Examine variables:

```
[1] pry(main)> x 
=> 1
[2] pry(main)> y
=> 2
```

Use CTRL-d to exit from the current breakpoint and continue execution, hitting the next breakpoint:

```
[2] pry(main)> 
Now s=3 !!

From: /Users/justin/prog.rb @ line 8 Object#f:

     3: def f(x,y)
     4:     puts('Inside f!')
     5:     require 'pry'; binding.pry
     6:     s = x+y
     7:     puts("Now s=#{s} !!")
 =>  8:     require 'pry'; binding.pry
     9:     return s
    10: end
```

Type `!!!` to immediately quit out of `pry`. 



### Debug via introspection 

(This section uses some advanced concepts, so you may want to skip it for now.)

Ruby does support some interesting introspection that can help with debugging. Here is a summary:

An object:

```ruby
x = 'asdf'
```

List all methods, even ones inherited from superclasses, and ones defined in included modules:

```ruby
x.methods     #  => [:encode, :include?, :%, :*, :+, :count, :partition, :to_c, ... ]
```

To dig deeper, gotta talk to the object's class:

```ruby
x.class       # String
```

List the classes and modules that will be searched when you do a method call:

```ruby
x.class.ancestors   #  => [String, Comparable, Object, Kernel, BasicObject]
```

Some of these are classes, some are modules. Amazingly, even classes and modules are themselves objects (inheriting from the `Class` and `Module` classes), and therefore have methods that you can call on them. Here we call the `class` method on each of `x`'s ancestors to see whether each ancestor is a class or a module:

```ruby
x.class.ancestors.map { |c| c.class }  # => [Class, Module, Class, Module, Class]
```

It can be useful to know which class or module defined a given method.
You can find out by creating a `Method` object that points to the object's method of interest, and asking for its owner:

```ruby
m = x.method(:encode)             #  => #<Method: String#encode>
m.class                           #  => Method
m.owner                           #  => String
```

Different methods are defined by different classes and modules in the hierarchy:

```ruby
x.method(:encode).owner     #  => String
x.method(:between?).owner   #  => Comparable
x.method(:nil?).owner       #  => Kernel
```

A very useful debugging trick is to use the `source_location` method ask a method in which file (and line #) it was defined. For example:

A library:

```ruby
# my-lib.rb

class C
    def f(x,y)
        x+y
    end
end    
```

Your program:

```ruby
# File prog.rb

require './my-lib.rb'
c = C.new

# Add breakpoint for debugging:
require 'pry'; binding.pry
```

Run it and ask which file defines `c.f`:

```
$ ruby prog.rb 

From: /Users/justin/prog.rb @ line 7 :

    2: 
    3: require './my-lib.rb'
    4: 
    5: c = C.new
    6: 
 => 7: require 'pry'; binding.pry

[1] pry(main)> c.method(:f).source_location
=> ["/Users/justin/my-lib.rb", 4]
```


## Datatypes and Control Flow


### nil

The value `nil` denotes the absence of a value:

```ruby
['a','b','c'].index('c')    # => 2
['a','b','c'].index('z')    # => nil
```

Use the method `nil?` to check if a value is `nil`:

```ruby
i = ['a','b','c'].index('b')
j = ['a','b','c'].index('z')
i.nil?      # => false 
j.nil?      # => true

'hi'.nil?    # => false
false.nil?   # => false
3.14.nil?    # => false
nil.nil?     # => true
```

### Only nil and false are falsey

That is,

```ruby
x = nil
if x then puts('nil is true') else puts('nil is false') end       # prints "nil is false"

y = false
if y then puts('false is true') else puts('false is false') end   # prints "false is false"
```

**Everything besides `nil` and `false` is truthy.**

(This is different from Python and Javascript.)

In particular, the following objects are **truthy**: 

```ruby
[]   # empty array
{}   # empty hash
''   # empty string
0    # integer zero
0.0  # float zero
```

You can use the cute trick of double-not (`!!`) to see an object's truthiness easily:

```ruby
!!nil     # false
!!false   # false
!!true    # true
!![]      # true
!!{}      # true
!!''      # true
!!0       # true
!!0.0     # true
```

(As an intriguing aside, I left out `//` (empty regex literal) (we discuss regex [later](#regex)) because although a *variable* whose value is a `Regexp` is truthy, the truth value of any regex *literal* actually depends on the value of the mysterious built-in global variable `$_`:

```ruby
x = //
!!x          # true (expected)
!!//         # false (wtf!)
$_ = 'abc'
!!//         # true (wtf!!)
!!/b/        # true
!!/z/        # false (wtf!!!)
```

This bizarre behavior is explained as follows. Inspired by Perl, Ruby has a few global, built-in variables that start with dollar-sign `$`. They are used in special-cased, non-obvious, implicit ways to enable powerful one-liners on the command line, for example, filtering lines of text from stdin. The variable `$_` is the global variable for "the most recent line read from stdin". To enable this command-line magic, Ruby adopts the following sneaky behavior: When asked to compute the truth value of a regex literal, instead of simply having regex literals be truthy like the other literals, Ruby instead converts the regex literal from `/foo/` to `/foo/.match?($_)` --- that is, Ruby checks if the most recent line from stdin matches the given regex literal. (In the example above, where we set `$_` to `'abc'`, the regex `/b/` matches the `'b'` in the string `'abc'`, but the regex `/z/` does not match.) This design decision surely has useful and impressive applications (see [here for examples](https://robm.me.uk/ruby/2015/10/31/dollar-underscore.html)), but I think it confuses more folks than it helps (see [here on StackOverflow](https://stackoverflow.com/questions/58340585/why-is-a-regexp-object-considered-to-be-falsy-in-ruby)).

End aside.)




### String interpolation syntax

Like Bash, single-quoted string literals preserve their characters, whereas double-quoted strings support escaped characters and interpolation with `#{}`:

```ruby
'\n'.length   # => 2  (has literal backslash)
"\n".length   # => 1  (has newline character \n)

x = 100
s = 'Not interpolated: #{x}'         #  => "Not interpolated: \#{x}"
                                     # ( Note: IRB prints # as \# )
t = "I ate #{x} pizzas"              # I ate 100 pizzas
u = "Tim ate #{x**x} pizzas"         # Tim ate 100...000 pizzas

v = "I feel #{                       # Can interpolate multi-line code
        if x < 5
            "Great"                  # Quotes are matched smartly
        else
            "Horrible" 
        end
        }"  
```

**Warning.** The value `nil` interpolates to empty-string, which is bad for debugging; not even `p` can save you, if it's printing a string where the `nil`s have already been interpolated to empty-strings! The workaround is to call `inspect`:

```ruby
x = nil
puts("x=#{x}")            # BAD: "x="
p("x=#{x}")               # BAD: "x="
puts("x=#{x.inspect}")    # GOOD: "x=nil"
```


### Symbols

A symbol is a datatype similar to a string, but with less functionality. 

```ruby
s = 'asdf'
t = :asdf
s.class     #  => String
t.class     #  => Symbol
```

A symbol literal is delimited with a single leading colon:

```ruby
s = :asdf
t = :+       # weird chars ok
u = :has_underscores
v = :'Symbols can have whitespace and other chars, but don\'t do it!'
w = :'123'   # leading digit => symbol must be quoted
```

There are two motivations for symbols.

**Motivation 1**: General-purpose strings are good for general-purpose natural language, which can have whitespace, multiple lines, text encodings, etc. Symbols are simpler than strings: Symbols are meant for *names of things*, for example, the name of a method, the name of a method's argument, or an HTTP verb (`GET`, `PUT`, `DELETE`, `PATCH`). Names typically do not have the concept of trailing whitespace, multiple lines, etc. You do not need to do "string surgery" (indexing, concatenating, substituting, etc) on the name of a thing. Consequently, symbols have fewer methods than strings:

```ruby
:asdf.methods.count     # 84
'asdf'.methods.count    # 183
```

Strings have many methods that Symbols do not have:

```ruby
'asdf'.methods - :asdf.methods    # => [:encode, :include?, :%, :*, :+, :count, ... ]
```

Symbols have only a couple methods that Strings don't have:

```ruby
:asdf.methods - 'asdf'.methods    # => [:to_proc, :id2name]
```

- `id2name` is an antiquated name for `to_s` ("to string").
- `to_proc` is part of a bloated, self-indulgent addition to the language that we discuss [here](#passing-symbol-instead-of-code-block). Basically, `:a.to_proc().call(obj)` performs `obj.a()` --- roughly speaking, it turns a symbol *s* into a function that takes an object and calls the method named *s* on it.


**Motivation 2**: (This next motivation is not as compelling, but you see pedants cite it, so you need to recognize it.) Historically, symbols were appealing because they were more memory-efficient than strings: if two variables point to symbols, and the symbols have the same value, then the symbols are stored in the same memory address:

```ruby
x = :asdf
y = :asdf
x.object_id     # => 1271388
y.object_id     # => 1271388  (same!)
```

This is not true of strings:

```ruby
x = 'asdf'
y = 'asdf'
x.object_id    # => 70287991526840
y.object_id    # => 70287991798420  (different)
```

This is not a big selling point. Besides, strings used as hash keys are optimized in a similar way since Ruby 2.2 (Dec 2014), and ALL string literals will be optimized in Ruby 3.0 (Dec 2020). (In Ruby terminology, strings will be "frozen" by default.)


**Important**: newcomers to Ruby are often confused between variables and symbols. They worry that the symbol `:a` is somehow linked to a variable named `a`. They are not linked in any way. That is like worrying if you're allowed to have a string literal `'a'` if you don't have a variable named `a`. A string is just a bunch of arbitrary characters! So is a symbol. 


### Hash 

A Hash is a set of key/value pairs, like Python's `dict` or a Java's `HashMap`. Hash literals are delimited with curly braces, and the rocket-ship operator `=>` separates a key from its value:

```ruby
h = { :a => 1, 'b' => 2, nil => :asdf, 999 => { 10 => 20 } }

h[:a]           #  => 1
h['b']          #  => 2
h['c']          #  => nil  <-- missing key doesn't raise KeyError!
h[nil]          #  => :asdf
h[999][10]      #  => 20
h.dig(999, 10)  #  => 20  dig() is useful for nested hashes
```

Confusingly, when defining a hash literal, there is a special optional syntax for defining a key/value pair whose key is a symbol: You may move the symbol's colon from its left to its right, and remove the rocket-ship `=>`:

```ruby
h = { a: 1 }    # => { :a => 1 }
h[:a]           # => 1
h[a]            # undefined method or variable 'a'
```

It is **critical** to note that when using this "right-hand colon" shortcut notation for hash literals, the `a` in the hash literal `{ a: 1 }` is **not a variable**, it is the symbol `:a`. It is irrelevant whether or not a variable `a` has been defined. You can see that IRB prints it out in the traditional explicit form `{ :a => 1 }` with the rocket-ship operator:

```ruby
{ a: 1 }    #  => { :a => 1 }
```

Make sure you understand this behavior:

```ruby
a = 'wow'         # defines variable named a
h = { 'a' => 10 } # key is the string 'a'
h = { :a => 10 }  # key is the symbol :a
h = { a: 10 }     # key is the symbol :a (fancy syntax)
h = { a => 10 }   # key is the string 'wow' (!!)
```

This "right-hand colon" shortcut is very bad: Newcomers already feel uneasy about the relationship between symbols with variables, and this shorthand syntax for hash literals deepens their confusion. Worse, this syntactic trick with symbolic keys is easily confused with another syntactic shortcut involving a function's keyword arguments, discussed in [this section](#automatic-conversion-btwn-hash-and-keyword-args).

You might wonder whether a company could simply adopt a style guide forbidding this weird implicit confusing syntax. Unfortunately the syntax is pervasive in Rails and therefore all Rails docs. So it is easier for newcomers to grit their teeth and do as the Romans-On-Rails do.


### if / while / case returns the last thing executed

Like functions, control flow statements "return" values; they return the last statement executed. You can therefore assign an `if` statement to a variable --- impossible in Python:

```ruby
p = 85
s = if p > 90
        puts 'Great work!'
        'A'
    elsif p > 80
        puts 'Good job!'
        'B'
    else
        puts 'Try harder!'
        'C-'
    end

# prints "Good job!"

s   #  => "B"
```

This is used with `case` statements too:

```ruby
x = 'pizza'
double = case x
         when Integer
            2*x
         when String
            "#{x} #{x}"
         end

double  #  => "pizza pizza"
```

*Note*. You might be wondering how `case` compares its argument `x` with its cases `Integer` and `String`. The insane answer is that case statements use the `===` operator:

```ruby
x = 'pizza'
Integer === x  # => false
String === x   # => true
```

The `===` operator is NOT like in JavaScript. It is an equality operator that is meant to be overloaded for your specific application, so that you don't have to overload the normal `==` operator. But that is weird, so don't do it.

Incredibly, the `===` operator is not commutative:

```ruby
x = 'pizza'
String === x   # => true
x === String   # => false, wtf
```

(The reason is that, in the 1st case, `===` is being called on the `String` class itself, whereas in the 2nd case, `===` is being called on the string *instance* `x`. Those two entities define `===` differently: The 1st definition is "is my arg an instance of String?", and the 2nd definition is probably something like "is my arg a string whose characters are equal to mine?".)

Never use the `===` operator.


### Regex

Ruby offers a dedicated datatype for regular-expressions, and dedicated weird syntax for regex literals and regex matching.

A regex literal is delimited with slashes:

```ruby
r = /asdf/
r.class     # => Regexp
```

The strange `=~` operator, defined both on `String` and on `Regexp`, searches a string for a regex:

```ruby
s = 'This string has asdf as a substring'
s =~ r      # => 16  (the index where asdf appears) (=~ called on s)
r =~ s      # => 16  (=~ called on r)
```

Luckily, unlike `===` from the previous section, the `=~` operator has the same definitions on `String` as on `Regexp`. That is, it is somewhat "commutative". 

IMHO it's better to use the explicltly-named `match?` method than the weird `=~` operator:

```ruby
r = /[a-z0-9\.]+@gmail\.com/   # Ruby regexs support the normal regex syntax
s = 'bob1985@gmail.com'
t = 'joe@hotmail.com'
s.match?(r)   # => true
t.match?(r)   # => false
```

- Add `i` after the trailing slash to make the regexp case-insensitive.
- Special characters `\d` for `[0-9]`, etc.

**Note:** For "beginning/end of line", you will see `\A` and `\z` used instead of the normal `^` and `$` characters. It can be confusing to know which to use. They differ only with multi-line strings:

- `^`  Matches beginning of any line in a possibly multi-line string
- `$`  Matches end of any line in a possibly multi-line string
- `\A`  Matches beginning of the string.
- `\Z`  Matches end of the string. If string ends with a newline, it matches just before newline.
- `\z`  Matches end of the string.

Example:

```ruby
r = /^hit$/          # Will match any string containing a line 'hit'
q = /\Ahit\Z/        # Will match only the string 'hit'
s = "Tom\nhit\nBob"
s =~ r               # => 4: 4th char of string contains the line 'hit'
s =~ q               # => nil: the string does not equal the string 'hit'
```

As in all languages, regexes quickly become incomprehensible and are best used clearly and sparingly.


## Functions

### Functions return the last thing executed

Consequently, this:

```ruby
def f(x,y)
    s = x+y
    return s
end
```

... can be shortened to:

```ruby
def f(x,y)
    x+y
end
```

... and for quick debugging, since `p` returns its arg, you could do:

```ruby
def f(x,y)
    p x+y
end
```

(However, if you had used `puts`, which returns `nil`, the function would always return `nil`.)

Note that `return`  can be used to force the return of a function:

```ruby
def sqrt(x)
    if x < 0
        return   # returns nil
    end
    # ...
end
```

Or equivalently, using the one-line `if` syntax, we can rewrite the guard clause as a one-liner:

```ruby
def sqrt(x)
    return if x < 0
    # ...
end
```

Here is a common confusion arising from "functions return the last thing executed". I once encountered a function of this form:

```ruby
def f(list)
    list.map do |x|
        x.foo do |y|
            y.bar do |z|
                z.baz()
            end
        end
    end
end
```

(We discuss `map` more in [Code Blocks](#code-blocks), but you're probably familiar with it already.) 

This function doesn't return anything explicitly. But this function's first, and last, thing executed is the call to `map`. So the function returns whatever `map` returns. And `map` returns the array `list` where each element has been passed through a mapping function. But, the mapping function is the long, complicated lambda function that begins with `do |x|` and which calls other functions that themselves take code blocks (eg, `do |y|`). The newcomer is immediately lost in the details of the mapping function, and might actually forget whether or not this function is supposed to return anything. This is a serious problem, because in Ruby On Rails, many functions simply write to the database, and their return values are irrelevant. Worse, `map` is almost identical to `each`, except that `map` returns the mapped array, whereas `each` simply returns the original array. In a database-writing function, you'd use `each`, whereas if you're processing a list to return, you'd use `map` to return the processed list.

My point is that I had to understand several low-level details in order to guess what this function might do. Many of these details can trip up a newcomer, and many of them (but not all) could be avoided by clarifying the code with an explicit `return`:

```ruby
def f(list)
    return list.map do |x| 
        # ...
    end
end
```

### Parens are optional in a function call

This is one of the most striking features of Ruby's syntax: 
when calling a method, you may omit the parentheses that would normally delimit the args.

That is, the method call

```ruby
o.m(a,b,c)
``` 

can be written equivalently as

```ruby
o.m a, b, c
```

For example:

```ruby
def f(x,y)
    x+y
end

f(1,2)   # => 3
f 1,2    # => 3
```

Besides being very weird syntactically, the absence of parens confuses Python programmers, because in Python, `o.m` accesses the instance variable named `m`, whereas `o.m()` is calling the method `m`. 

But Ruby is very strict: instance variables are not available outside the class. The only thing you can do with objects is call their methods. (Ruby has convenient ways of generating setter and getter methods to access instance variables, which we will see later.) (This idea is from [Smalltalk](https://en.wikipedia.org/wiki/Smalltalk#Object-oriented_programming).) So `o.m` is always calling the method `m` on the object `o`, and is never directly accessing an instance variable.

Omitting parens invites operator-precedence bugs, e.g., 

```ruby
x.is_a? Hash || x.respond_to? :[] 
```

The author intended

```ruby
x.is_a?(Hash) || x.respond_to?(:[])
```

(That is, is `x` a Hash object, or can I at least use square-brackets on it?)

But stupidly, `||` binds tighter than method-call, so it is parsed as

```ruby
x.is_a?( Hash || x.respond_to?(:[]) )
```

and the `||` evaluates to the first non-`nil` argument --- `Hash` --- so the line reduces to 

```ruby
x.is_a?( Hash )
```

which will not return `true` for non-Hash objects that support `[]`, as the author intended.



### Positional args, Keyword args, default values

As you know, the simplest functions use positional arguments:

```ruby
def f(x,y)
    x+y
end    

f(1,2)      #  => 3
f()         # ArgumentError (wrong number of arguments (given 0, expected 2))
```

This error message is not very helpful, since it doesn't tell you the names of the required args. We will see a better way shortly.

Like Python, the splat notation unrolls an arg list into positional args:

```ruby
def f(x,y)
    x+y
end

args = [1,2]
f(*args)      # => 3
```

Default values for positional arguments are defined like this:

```ruby
def f(x=10, y=20)
    x+y
end

f()     #  => 30
f(1)    #  => 21 
f(y=1)  #  => 21 (WRONG, we expected 11)
```

The above "WRONG" example occurs because the expression `y=1` is executed, defining the variable `y` in the caller's namespace (BAD), and then the function assigns the value of the expression `y=1`, which is `1`, to the function argument `x`, leaving the function argument `y` to its default value of 20. This silently conflicts with Python's function-calling syntax.

Instead of positional arguments, I prefer **keyword arguments**:

```ruby
def f(x:, y:)
    x+y
end

f(1,2)  # ArgumentError (wrong number of arguments (given 2, expected 0; required keywords: x, y))
f()     # ArgumentError (missing keywords: x, y)
f(x: 1, y: 2)   # => 3
f(y: 2, x: 1)   # => 3  (can specify args in any order!)
```

The first error message is a bit confusing: when it says it `expected 0` arguments, it means it expected zero *positional arguments*.

Keyword arguments force the caller to explicitly write the names of the inputs, which is nice for clarity. A famous OOP consultant once told me: 
"A piece of code is written once, but read many times." So it's worth writing a little more now, to help your future readers.

Default values for keyword args can be specified:

```ruby
def f(x: 10, y: 20)
    x+y
end

f(y: 1)  #  => 11, as desired
```

Like Python, the double-splat notation unrolls an arg hash into keyword args:

```ruby
def f(x:, y:)
    x+y
end

h = {x: 1, y: 2}
f(**h)            # => 3
```

A function's definition can specify both positional args and keyword args:

```ruby
def f(x, y, verbose: false)
    s = x+y
    if verbose
        p("Sum = #{s}")
    end
    s
end
```

... but why not just always use keywords for uniformity and clarity? 


### Automatic conversion btwn hash and keyword args

There is a confusing quirk to keyword argument syntax, which muddies the waters: 

When calling a function that expects keyword args, you can instead provide a hash whose keys match the keyword arg names:

```ruby
def f(x:, y:)
    x+y
end

h = {x: 1, y: 2}
f(h)               #  => 3, but you'd expect a "Missing keywords" error!
```

To avoid parsing ambiguity, this only works if the keyword args are the rightmost arguments in the function definition, in which case the hash will be the last positional argument in the function call.

Maddeningly, Ruby also permits being sloppy in the opposite way:

When calling a function that expects a Hash as the last argument, you can provide the hash's key/value pairs without the curly braces:

```ruby
def f(x,y,h)
    puts("x: #{x}, y: #{y}")
    puts("Keys: #{h.keys}")
    puts("Values: #{h.values}")
end

f(1, 2, a: 1, b: 2, 'hello' => 'world', 1000 => 'thousand')  # Note: NO CURLY BRACES!
```

prints

```
x: 1, y: 2
Keys: [:a, :b, "hello", 1000]
Values: [1, 2, "world", "thousand"]
=> nil
```

This is confusing because it makes the hash's keys look like explicit keyword arguments, which doesn't match the function's definition. 

(It becomes utterly baffling and incomprehensible if you then proceed to omit the parens in the function call.)


This bizarre syntactic sloppiness comes from Ruby's history: Ruby did not support keyword arguments until Ruby 2.0 in 2013. Before then, the workaround was the convention of using a hash as a final positional argument --- called an "options hash" --- as an attempt to name the arguments of a function call. Removing the requirement for curly braces around the options hash provided a minor convenience until keyword args were officially supported in 2013.

These two symmetric features, that let you be sloppy about keyword args versus hash args, were actually going to be 
[removed from the language in Ruby 3.0](https://www.ruby-lang.org/en/news/2019/12/12/separation-of-positional-and-keyword-arguments-in-ruby-3-0/),
starting with deprecation warnings in Ruby 2.7 
(discussed [here](https://bugs.ruby-lang.org/issues/14183) 
and
announced [here](https://www.ruby-lang.org/en/news/2019/12/25/ruby-2-7-0-released/)), 
but the Rails community relies on these sneaky shortcuts so heavily that Matz decided to 
[postpone the removal](https://discuss.rubyonrails.org/t/new-2-7-3-0-keyword-argument-pain-point/74980).




### Methods ending in ? or !

Conventionally, methods ending in a question mark (`?`) return `true` or `false`:

```ruby
def palindrome?(s)
    return s == s.reverse
end
```

Conventionally, methods ending in an exclamation point (`!`) (also called a "bang") do something dangerous: have a subtle side-effect, raise an exception, etc. The bang reminds the programmer to "use caution" when calling this method.

For example, `Array#map` returns a mapped array, but leaves the original object unchanged:

```ruby
x = [1,2,3]
x.map() { |e| e*100 }   #  => [100,200,300]
x                       #  => [1,2,3]  (unchanged)
```

However, `Array#map!` returns a mapped array, but ALSO modifies the original object!:

```ruby
x = [1,2,3]
x.map!() { |e| e*100 }  #  => [100,200,300]
x                       #  => [100,200,300]  (CHANGED!)
```

Two important misconceptions:

- Newcomers think that every "bang" method must have a non-bang counterpart. This is not true. Just because an object has a `foo!` method doesn't mean it has a `foo` method, and vice versa. They are separate methods. In principle they could do totally separate things. But often they do related things, and the bang simply means that you need to read the docs to understand what dangerous thing the bang-method does.

- Important: Newcomers see the `map!` example and conclude that "bang" means "it modifies the caller". This is not true. "Bang" simply means "use caution": This method might do something weird. For example, in Ruby On Rails, an `ActiveRecord` object has both a `save` method and a `save!` method. They both insert the object's data into the database. If that fails (possibly because the database server is down, or a db validation fails), `save` returns `false`, whereas `save!` raises an exception. Since `save` just returns `false`, your program will continue running. But with `save!`, unless you rescue that exception, your program will die. The bang in `save!` means "Use caution". Specifically: "Use caution --- this method may raise an exception, which will kill your program if you don't rescue it!"



### Weird ways to call methods

Method names are often stored as symbols:

```ruby
x = 100
meths = x.methods  #  => [ :+, :-, :/, :*, :size, :succ, :<, :>, ... ]
```

We all know the normal way of calling a method on an object:

```ruby
x = 100
x.succ()    #  => 101
x.succ      #  => 101  (parens optional)
```

Ruby enables lazy programming with the "safe method call" operator, which checks if the receiver is `nil` before trying to call a method on it:

```ruby
x = nil
x.succ()   #  Raises exception: NoMethodError (undefined method `succ' for nil:NilClass)
x&.succ()  #  => nil
```

(This is very dangerous because it can mask bugs.)

If you have a method name as a symbol, you can use `send` to call that method on an object:

```ruby
x = 100
m = :succ
x.send(m)    #  => 101

y = 999
m = :<       # a perfectly valid symbol
x.send(m, y) #  => true   (100 is less than 999)
```

You occasionally see `send` used in meta-programming applications.

Surprisingly, Ruby allows you to call even *non-alphanumeric* method names with the canonical "dot" method-call syntax:

```ruby
x = 100
x < 999      #  => true
x.<(999)     #  => true  (using "dot" method call syntax)
x.< 999      #  => true  (omitting parens)

y = 999
x+y            # => 1099  (use normal arithmetic infix operator "+")
x.send(:+, y)  # => 1099  (use Object#send with symbol :+ and arg y)
x.+(y)         # => 1099  (use dot notation w parens)
x.+ y          # => 1099  (use dot notation w/o parens)
```

(I've never seen this used in real life.)



## Code blocks

We have seen that Ruby's method-call syntax is pretty weird:

1. parens are not required.
2. keyword args are interchangeable with positional-arg hashes

Amazingly, there is an even weirder feature of Ruby's method-call syntax, which we will now discuss:

**Every method call may be passed an anonymous function literal called a *code block*.**

For example (the curly braces `{}` delimit the code block; the pipes `||` delimit its parameter list):

```ruby
[1,2,3].map() { |e| e*100 }   #  => [100, 200, 300]
```

A code block is written AFTER all the positional and keyword args, OUTSIDE the closing paren of the method-call:

```ruby
[1,2,3].map() { |e| e*100 }     # YES: => [100, 200, 300]
[1,2,3].map   { |e| e*100 }     # YES: => [100, 200, 300] (Parens optional in Ruby!)
[1,2,3].map(  { |e| e*100 } )   #  NO: syntax error
```

Keywords `do` `end` can be used in place of curlies. (Looks better for multi-line code blocks.):

```ruby
[1,2,3].map() do |e| 
    puts("In the code block! e=#{e}")
    e*100 
end
```

prints

```
In the code block! e=1
In the code block! e=2
In the code block! e=3
=> [100, 200, 300]
```

Technically, **any** method call may be passed a code block, even if the method wasn't expecting it and doesn't invoke it:

```ruby
def f(x,y)
    x+y
end

f(1,2) { |a,b,c| puts("Hello world!") }   #  => 3 
```

Note that the code block remains *unevaluated* until the method invokes it. If the method never invokes it, the code block never executes. For example:

```ruby
def f(x,y)
    x+y
end

f(1,2) { raise 'Uh oh!' }  #  => 3 (code block is not invoked)
```

It is very important to understand that a code block is part of a method call. It is essentially a special kind of argument.

For example, in the examples above, `map()` expects NO positional or keyword arguments, but it does expect to be given a code block when it is called.

**Important:** a code block can ONLY appear as part of a method call. For example, you cannot assign a code block to a variable:

```ruby
b = { |e| e*100 }  # syntax error
```

If you can't assign code blocks to variables, then how do you make lambda functions? We discuss that next.


### Lambda functions.

To make a lambda function, give a code block to the `Proc` class constructor (Ruby's class for encapsulating lambda functions), which will embed your code block into a `Proc` object:

```ruby
b = Proc.new { |e| e*100 }
b.call(3)  # => 300
b.(3)      # => 300  (NEVER use this subtle confusing syntax)
b[3]       # => 300  (same)
```

For historical reasons, there is a built-in function `proc` that is an alias for `Proc.new`.

You may also have seen the function `lambda`, which (confusingly) also makes `Proc` instances:

```ruby
b = lambda { |e| e*100 }        #  => #<Proc:0x00007fc2c495ee88@(irb):43 (lambda)>
b.class                         #  => Proc
b.lambda?                       #  => true
```

The `proc` and `lambda` functions both create `Proc` instances. They differ in how they treat missing arguments: `proc` and `Proc.new` set missing args to `nil` and ignore extra args, but `lambda` will throw an error if you don't call it with the right number of args:

```ruby
f = proc       { |x, y| puts("x=#{x.inspect}, y=#{y.inspect}") }
g = Proc.new   { |x, y| puts("x=#{x.inspect}, y=#{y.inspect}") }
h = lambda     { |x, y| puts("x=#{x.inspect}, y=#{y.inspect}") }

# Missing args:
f.call(1)  # x=1, y=nil
g.call(1)  # x=1, y=nil
h.call(1)  # ArgumentError (wrong number of arguments (given 1, expected 2))

# Extra args:
f.call(1,2,3)  # x=1, y=2
g.call(1,2,3)  # x=1, y=2
h.call(1,2,3)  # ArgumentError (wrong number of arguments (given 3, expected 2))
```

Ruby supports a confusing second syntax for defining lambda functions, called a *lambda literal*:

```ruby
f = ->(x,y) { x+y }   # #<Proc:... (lambda)>
f.call(1,2)           # => 3

g = -> { puts 'hi' }  # Takes no args
g.call                # Prints 'hi', returns nil
```

This syntax saves a few characters of typing, yet must be learned by every Ruby developer. 

Worse, it is confusing because it looks too similar to a code block, making it look like you can assign a code block to a variable:

```ruby
f = -> { puts('hello') }   # ok 
g =    { puts('world') }   # syntax error
```

It is critical to observe that a lambda literal, like a code block, is not executed when it is defined:

```ruby
f = -> { raise 'uh oh' }   # ok!
g = -> { 1/0 }             # ok!
h = -> { 10.barf() }       # ok!
```

Therefore you see people use lambda literals as a quick, easy way to defer execution of their code, sometimes even defining the lambda literal inside a method call.



### With lambdas, why need Code Blocks?


You might wonder why Ruby has special syntax for tacking on an extra argument (an anonymous function literal) to a method call, since Ruby DOES support lambda functions, which can be passed as normal positional or keyword arguments, thereby making code blocks unnecessary:

```ruby
# map() without code blocks:
def map(func, list)
    out = []
    for e in list
        out.append( func.call(e) )
    end
    return out
end

# Define a lambda function object:
f = lambda do |e|
    puts("In the code block! e=#{e}")
    e*100
end

map(f, [1,2,3])  # map the lambda function over the list
```

prints

```
In the code block! e=1
In the code block! e=2
In the code block! e=3
=> [100, 200, 300]
```

As far as I can tell, the main advantage of the special code block syntax is that you don't have a bunch of leftover closing parens when you have nested code blocks.


### Passing a Proc as a code block

What if I have a `Proc` object that I want to give to a method-call, instead of defining a code block literal? The clumsy way is to do this:

```ruby
b = Proc.new { |e| e*100 }
[1,2,3].map { |e| b.call(e) }   #  => [100, 200, 300]
```

... but there is a better way:

When calling a method, instead of providing a code block literal, you can pass a `Proc` object as the last argument, prepended with an ampersand (`&`), which Ruby will treat as if you had provided a code block literal:

```ruby
b = Proc.new { |e| e*100 }
[1,2,3].map(&b)                 # => [100, 200, 300]
```

This is confusing because `map` takes no arguments, yet it looks like `map` is being given an argument. It results in hard-to-catch errors. Can you spot it?:

```ruby
b = Proc.new { |e| e*100 }
[1,2,3].map(b)   # ArgumentError (wrong number of arguments (given 1, expected 0))
```


### Defining a method that expects a code block

So far we have only talked about how to **use** a method that expects a code block. Now we discuss how to **write** a method that expects a code block.

Inside the method's body, the `yield` keyword invokes the code block:

```ruby
puts("Defining the method.")
def f(x,y)
    s = x+y
    puts("Sum = #{s}")
    z = yield(s)    # Call the code block
    puts("The code block returned z=#{z}")
    z # return the result of passing the sum thru the code block
end

puts("Calling the method.")
f(1,2) do |e| 
    puts("In the code block! e=#{e}")
    e*100
end
```

prints

```
Defining the method.
Calling the method.
Sum = 3
In the code block! e=3
The code block returned z=300
=> 300
```

(Very important: Ruby's `yield` seems similar to Python's `yield`, in that `yield` returns from the currently-executing function in a manner that allows the function to resume execution later. But the similarity ends there. In Python, `yield` converts a normal function into a generator that returns a succession of values when you call `next()` repeatedly on it. In Ruby, `yield` is how a function calls the lambda function that was provided to the function.)

Pay close attention to how the control of the program flowed: First the method author defined a method that expects a code block. Then the caller called the method. Then the method ran partway. Then the method called the code block, which effectively returned control to the caller (who authored the code block). After the block returned, the method resumed. Then the method finished, returning control to the caller once again. Complicated! Code blocks introduce a lot of complex redirection into your code.

If a function calls `yield`, but the caller didn't provide a code block, a `no block given` error is raised:

```ruby
def f(x,y)
    s = x+y
    yield(s)  # Pass sum to the code block
              # and return whatever the code block returns
end

f(1,2)        # error: no block given
```

You can use the `block_given?()` boolean-valued function to see if the caller provided a block:

```ruby
def f(x,y)
    s = x+y
    z = block_given? ? yield(s) : s
end

f(1,2)               #  => 3
f(1,2) {|e| e*100}   #  => 300
```

A downside of `yield` is that the method body doesn't have a variable referring to the code block, which is handy if you want to pass the code block down to a sub-function. To address this, Ruby offers a special method-definition syntax:

Just as you can use `&` to pass a `Proc` as a code block in a method-call, you can use `&` in a method *definition* to give a variable name to your method's code block:

```ruby
def f(x, y, &b)  # Note the ampersand (&)
    s = x+y
    z = b.call(s)  # b is a Proc instance
    z
end

# Same as before:
f(1,2) do |e| 
    puts("In the code block! e=#{e}")
    e*100
end
```

This is confusing because the method definition looks like it takes 3 positional arguments, but it only takes 2 (and a code block).



### Code blocks are over-used

Code blocks are pervasive in Ruby. Even simple iterators are demonstrated using code blocks.

For example, many Ruby tutorials begin with an example that uses the `times()` method of `Integer`, which idiotically is meant to read like English when used with the `do end` code block syntax:

```ruby
total = 0
10.times do |i|
    puts("In the code block! i=#{i}")
    total += i
end
total
```

... which prints

```
In the code block! i=0
In the code block! i=1
In the code block! i=2
In the code block! i=3
In the code block! i=4
=> 45
```

This is a terrible way to begin your introduction to Ruby, because it combines three bizarre aspects of Ruby into a single incomprehensible mess:

1. you can call methods on Integer literals (weird)
2. parens are not needed on method calls (very weird)
3. every method call may be given an anonymous function literal called a code block (wtf!)

Moreover, students probably think the word `times` means "multiply", which causes further confusion. Shouldn't `10.times 3` return `30`??

In my opinion, code blocks are over-used. There are many places in Ruby On Rails where code blocks are used, when the same logic could be achieved without them. Each code block adds a new stack frame, and since code blocks are often nested (especially in Ruby On Rails), stack traces become long and hard to follow.



### Implementing Python's "with" as a code block

Despite being over-used, code blocks have several interesting uses. 

For example, consider Python's `with` statement, which is used to perform setup and teardown of an object. The following common Python example uses `with` to close a file after writing to it:

```python
with open('foo.txt','w') as f:
    f.write('hello')
    f.write('world')
```

Under the hood, `with` essentially does this:

```python
f = open('foo.txt','w')
f.__enter__()

### execute body of the `with`:
f.write('hello')
f.write('world')
###

f.__exit__()   # closes the file
```

Note that `with` allows a file object to open a file, then run some user-specified code, and finally close the file.

Ruby does not have a `with` statement, but code blocks allow us to write a `with()` function that looks like it's built-in to Ruby:

```ruby
def with(x, &b)
    b.call(x)
    x.close()
end

with open('foo.txt','w') do |f|
    f.write('hello')
    f.write('world')
end
```

Note that since Ruby doesn't require parens, the `open` call is the first and only positional argument to `with`. 

(Astute readers may criticize this example because `with` assumes that its arg supports a `close()` method. But that assumption was also needed in Python, whose `with` statement assumes its arg supports methods `__enter__()` and `__exit__()`.)

As an aside, it turns out that Ruby's `open` actually accepts a code block, which closes the file for you!:

```ruby
open('foo.txt','w') do |f|
    f.write('hello')
    f.write('world')
end
```

(So `with` is not needed for this use case!)

This example highlights an important aspect of code blocks: It requires careful coordination between the person who writes a method, and the person who calls the method:

1. When I call your method, does it expect a code block in addition to the normal positional and keyword args?
2. How many args should my code block accept? (That is, how many args will your method give to my code block?)
3. How many times will your method call my code block? 

For example, here is a snippet from the `open` method's docs ([apidoc.com](https://apidock.com/ruby/Kernel/open)):

```
If a block is specified, it will be invoked with the IO object as a parameter, 
and the IO will be automatically closed when the block terminates. 
The call returns the value of the block.
```


### Break vs Return

**Pitfall**: in a code block, use `break` to return early out of the block, whereby the calling method will continue. Astonishingly, using `return` in a code block actually forces the entire method to return!:

```ruby
def f(x, y, &b)
    s = x+y
    z = b.call(s)
    print("sum = #{s}, block output = #{z}")
    z
end

f(1,2) { |e| return e*100 }    # LocalJumpError (unexpected return)
```



### Passing Symbol instead of Code Block

You will probably see this strange idiom: 

```ruby
['a','b','c'].map(&:upcase)   # => ["A", "B", "C"]
```

This code is equivalent to

```ruby
['a','b','c'].map() { |e| e.upcase }    # => ["A", "B", "C"]
```

This ridiculous confusing shortcut saves only a few characters of typing, but it is a whole new idiom that assaults newcomers.

To understand this idiom, recall that normally the ampersand argument takes a `Proc` object, like this:

```ruby
p = Proc.new { |e| e.upcase }
['a','b','c'].map(&p)
```

However, astoundingly, if the ampersand argument is not a Proc, Ruby calls `to_proc` on it to try to convert it to a Proc. **(This is totally subtle and non-obvious.)**

Therefore, in the above example, Ruby calls `:upcase.to_proc`. As we saw in the section on [Symbols](#symbols), `Symbol#to_proc` is one of the few methods that Symbols have that Strings don't have. It is defined like this:

```ruby
class Symbol
    def to_proc()
        return Proc.new { |o| o.send(self) }
    end
end
```

(Recall that `o.send(m)`, where `m` is a symbol, is simply the method call `o.m()`)

That is, `:f.to_proc()` returns a `Proc` that takes an object and calls its `f()` method. So a convoluted way to write `4.odd?()` would be `:odd?.to_proc.call 4`.

The whole idea of expanding method call syntax to accept anything that defines `to_proc`, instead of simply a proc or a code block, is very bad for newcomers. They see `map(&:upcase)` and they mistakenly think `&:` has special meaning separate from `&`. They parse it as `map( &: upcase )`, whose bare `upcase` exacerbates their confusion between symbols and variables. To correct this mistake, they have to learn how `&` doesn't just take procs or code blocks, but ALSO calls `to_proc` as a backup, and conveniently, Symbol implements `to_proc`, despite Symbol supposedly being a simpler concept than String, and its implementation involves the meta-programmatic `send`, and if you look up `Symbol#to_proc` in the [official Ruby documentation](https://apidock.com/ruby/Symbol/to_proc), the entire help article is one enigmatic and ungrammatical line accompanied by a cryptic example that doesn't even use `to_proc` directly:

```
irb> help('Symbol#to_proc')

  sym.to_proc

Returns a Proc object which respond to the given method by sym.

  (1..3).collect(&:to_s)  #=> ["1", "2", "3"]
```

(Note that `to_proc` does not even appear in the given example!)

And all this complexity for a single weird idiomatic shortcut!

Oh Ruby! Truly a Programmer's Best Friend!!


## Classes and Modules

### Classes can be nested

Drill down into nested classes with `::`, like this:

```ruby
class C
    class D
        def f(x,y)
            x+y
        end
    end

    class E
        def f(x,y)
            x**y
        end
    end
end

d = C::D.new
e = C::E.new
d.f(1,2)      # => 3
e.f(2,3)      # => 8
```

### Instance vars are inaccessible outside the class

In Ruby, instance variables are named with a leading "at" (`@`), and they are not accessible outside the class:

```ruby
class C
    def initialize()
        @x = 1
    end
end

c = C.new
c.@x        # syntax error
c.x         # undefined method 'x'
```

Instead, to interact with instance vars from outside, you must write getter and setter methods:


```ruby
class C
    def initialize(v)
        @x = v
    end

    def x()
        @x
    end

    def x=(v)
        @x = v
    end
end

c = C.new(5)
c.x            # => 5  uses getter x()
c.x = 6        # uses setter x=(); now @x = 6 internally
c.x=( 7 )      # same, but uses dot notation w/ the x= method
c.x            # => 7
```

A minor point is that instance variables are "lifted" like in JavaScript, in the sense that they are automatically initialized to `nil` if they are mentioned anywhere in the class body:

```ruby
class C
    def f()
        @x
    end
end

c = C.new      # @x init'd to nil
c.f            # => nil, not "undefined" or "uninitialized"
```

Ruby provides the functions `attr_reader`, `attr_writer`, and `attr_accessor` to meta-programatically add a getter, setter, or both:

```ruby
class C
    attr_reader :x      # same as def x() ; @x ; end
    attr_writer :y      # same as def y=(v) ; @y = v ; end
    attr_accessor :z    # does both of the above
end

c = C.new
c.methods    #  => [:x, :y=, :z=, :z, ... ]

c.x          # => nil (all instance variables are initialized to nil)
c.x = 5      # error: undefined method x=()

c.y = 5      # instance variable @y is set to 5
c.y          # error: undefined method y()

c.z = 5      # ok
c.z          # => 5
```

A couple notes on this: 

1. It is legal to have a method named `x=`. Indeed, Ruby converts the expression `c.x = 5` into the canonical method-call form `c.x=(5)` --- calling the `x=` method on the object `c` with the argument `5`. Using `Object#send` makes this explicit: `c.send( :x= , 5 )`.

2. It looks like `attr_reader` (and friends) is a language keyword or macro, but it isn't. It is a normal function being called without parens, executing at *class-definition time*, not at runtime. Surprisingly, this is allowed (even in Python).
Moreover, in Ruby, every function has a receiver, and `self` is used as the receiver if one isn't specified. Therefore the call could be written more explicitly as:

```ruby
class C
    puts("In the class definition! self = #{self}")
    self.attr_accessor(:x)
end

# prints: "In the class definition! self = C"

c = C.new
c.x = 5
c.x         # => 5
```

Note that in the class body, `self` takes the value `C` --- the class that's in the process of being defined --- and therefore the method call could've been written explicitly as `C.attr_accessor(:x)`. This shows that `attr_accessor` is a class method of the class `C`; in Ruby, even classes are themselves objects --- instances of the `Class` class --- and therefore they have a class hierarchy that gets searched when you call methods on them:

```ruby
class C ; end  # define an empty class
C.methods      #  => [:new, :allocate, :superclass, ...
               #       :attr_accessor, ... ]

C.class                         #  => Class   (C is an instance of the Class class)
C.class.superclass              #  => Module  (the Class class is a subclass of the Module class)
Module.class                    #  => Class   (the Module class is a class)
C.class.superclass.class        #  => Class   (same)
C.class.ancestors               #  => [Class, Module, Object, Kernel, BasicObject]
C.method(:attr_accessor).owner  #  => Module  (the Module class defines attr_accessor)

```

Here we see that the `Module` class, a superclass of the `Class` class, defines `attr_accessor`. 

It is not worth spending much time studying Ruby's object model, because it is convoluted and you can do plenty of damage without it.



### Can modify existing classes at runtime

You might conclude from Ruby's strict stance on hiding instance variables that Ruby would take a similarly strict stance on the immutability of classes and objects. You would be wrong.

Astoundingly, you can "open up" existing classes at runtime and add, delete, or redefine their methods:

```ruby
class C          # some class
    def f(x,y)
        x+y
    end
end

c = C.new
c.f(1,2)    # => 3

class C  # open up C 
    def f(x,y)  # rewrite existing method
        x*y
    end

    def g(x,y)  # add new method
        x**y
    end
end

c.f(1,2)   # => 2 (different!! new def'n applies immediately!)
c.g(2,3)   # => 8 (no need to "reload" object c!)
```

For example, you can call the `attr_accessor` class method **even outside the class,** to add new methods dynamically at runtime:

```ruby
class C              # a class that doesn't expose its instance variable
    def initialize
        @x = 1
    end
end

c = C.new
c.x            # error, good

C.attr_accessor :x   # now we can "reach into" the class 
                     # and modify @x

c.x = 6     # No need to "reload" object c!
c.x         # => 6
```

It is fantastically dangerous to circumvent Ruby's protections of instance variables.

Ruby allows this even for built-in classes like String and Integer:

```ruby
4.odd?          # => false

class Integer
    def odd?
        true
    end
end

4.odd?          # => true
```

Rails uses this power to do all sorts of mischief. For example, it adds the `days` method to the `Numeric` class (superclass of `Integer` and `Float`), so that you can write:

```ruby
4.days.ago   #=> Tue, 08 Aug 2020 02:38:28 PST -08:00
```

Unsurprisingly, this is a horrible idea, because:

- It's subtle.
- It makes it look like you can program in plain English, which nobody would want to do in real life (ask a non-native English speaker, or anyone who's tried to write AppleScript!)
- It blurs the line between Ruby and the Rails framework, making it harder to debug: do I look at Ruby docs, or Rails docs? Can I use this in my non-Rails program? (No.)
- `days` converted my simple `Integer` into some weird Rails `ActiveSupport::Duration` object.
- The API is inconsistent: Rails adds the method `fortnight`, but no method `year`. It offers the conversion method `in_milliseconds`, but no `millisecond`. 
- And why stop with time-based units?? Rails 3.0 added `kilobyte`. Maybe more units will be added later. This pollutes `Integer`. The concept of a number-with-units is different than the concept of a number. You shouldn't cram the unit concept into the number concept.

### Can add methods to individual objects

We saw that Ruby permits adding, deleting, or rewriting existing classes at runtime --- even built-in classes. What is even more insane is that Ruby offers a mechanism for doing this *on individual objects*:

```ruby
a = 'a'
b = 'b'

def b.upcase
    'z'
end

a.upcase  #  => 'A'
b.upcase  #  => 'z'   # oh noooo
```

Ruby makes this possible through a completely bizarre mechanism called the *singleton class*. The top Google hit for "ruby singleton class" is an article about Ruby's apparently beautiful object model, which concludes as follows:

```
The superclass of the singleton class of a class is the singleton class of the class’s superclass.
```

"If you understand, no explanation is necessary. If you do not, no explanation is possible."

You should know how to check for meta-programming mischief:

```ruby
a.singleton_methods   #  => []
b.singleton_methods   #  => [:upcase]
```



### Modules

A module is a set of methods that you inject into your classes using `include`:

```ruby
module M
    def f(x,y)
        x+y
    end
end

class C
    include M
end

c = C.new
c.f(1,2)    #  => 3
```

The methods may refer to `self`, in which case they are instance methods of whatever class you `include` them in. This is known as a *mixin* in general OOP. It is Ruby's form of multiple-inheritance.

Like classes, modules may be nested, which is nice for name-spacing; nested modules are accessed with `::`. 

When you call a method on an object, Ruby searches the object's class hierarchy for that method's definition, as well as the modules included in each class:

```ruby
x = 5                             # => 5
x.class                           # => Integer
x.class.ancestors                 # => [Integer, Numeric, Comparable, Object, Kernel, BasicObject]
x.class.ancestors.map(&:class)    # => [Class, Class, Module, Class, Module, Class]

x.method(:abs).owner              # => Integer
x.method(:positive?).owner        # => Numeric
x.method(:between?).owner         # => Comparable
```

There's more to say on this, but I'm tired. 


## Closing Thoughts

Ruby's tagline is "A Programmer's Best Friend". It gives you a lot of power and flexibility, even to modify itself. The argument for this is, "Well, you need to know what you're doing. The language will let you do dangerous things. It's like a gun." This is not a compelling argument for newcomers who are faced with hidden global variables, strange new datatypes, sloppy inconsistent syntax, and bizarre method-calling conventions. What's worse, there are countless introductory tutorials touting Ruby as a beautiful, simple language. Ruby isn't a gun --- guns are simple. Ruby is a bomb inside a colorless Rubik's cube on opposite day, with a big label "So Easy To Use!"

As we saw, the learning curve is incredibly steep: to understand these apparently simple one-liners, which are pervasive in Ruby tutorials,

```ruby
['a','b','c'].map {|e| e.upcase}  # => ["A", "B", "C"]
['a','b','c'].map(&:upcase)       # => ["A", "B", "C"]
```

... we had to understand:

1. no parens for method call
2. code block syntax & usage
3. symbols
4. the `&`-based syntax for passing code blocks
5. procs
6. the `to_proc` method of Symbol

This is crazy! 

I've studied teaching, I've taught university courses, and I've been told I'm a gifted teacher. I taught AppFolio's "Intro to Ruby" workshop to new software engineers for a couple years at my job. But I never figured out how to introduce Ruby in a way that left my students feeling good about the language. They appreciated my guidance, but they worried the language (and/or Rails) would blow up in their face.

From here, my advice is to slowly and thoughtfully read *The Ruby Programming Language* written by Ruby's designer ("Matz"). It only covers Ruby 1.9, but nevertheless straightforwardly explains many of the language's oddities.

Good luck!

-Justin Pearson

Sep 2020