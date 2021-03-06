# Ruby

To update only a single gem:

```sh
bundle update --conservative somegem
```

## Intro

- Interpreted
- Dynamic types
- Multi paradigm (functional and procedural)

## Types

### Booleans

Ruby doesn't have a bool class.

True and false are instances of `TrueClass` and `FalseClass`.
You can't subtype them.

### Number

Ruby has an `Integer` class and 2 derived classes from it: `Fixnum` and `Bignum`.
The language automatically handles switching from one to the other.
Long numbers can be interpolated with `_` for readability.

```rb
100_000_000
```

### Strings

Interpolation

```rb
name = "John"
"This person's name is #{name}"
```

Substitutions

```rb
# is substring contained
"Africa"["ri"]
=> "ri" # in case it is, it returns the string itself
"Bridge"["go"]
=> nil # in case it's not
str = "H^llo, my n^me is M^x"
str.gsub("^", "a")
# => "Hallo, my name is Max"
```

Split a string

```rb
a = "Hello"
a.chars
["H", "e", "l", "l", "o"]
```

### Symbols

Symbols in Ruby are similar to what enums are in other langs.
Symbols are **globally unique and immutable**.

Symbols are written like this:

```rb
:some_symbol
:some_attr
```

Strings could be used in general to replace symbols but since a string is created every time the program reads that part of the code, Symbols have better perf, because the id of the symbol is the same.

### Arrays

```rb
# create an array of words
%w[these will be strings in an array]
# ["these", "will", "be", "strings", "in", "an", "array"]
%w[one\ word]
#  ["one word"]
# array of symbols
%i[one symbol after the other]
# [:one, :symbol, :after, :the, :other]

# append an item
[1,2,3] << 4
# [1, 2, 3, 4]

# filtering
[1,44,3,55,31,2].select { |n| n > 10}
# [44, 55, 31]
```

### Hashes

```rb
state = { "on" => true, "balance" => 223}
# or
state = { on: true, balance: 223}

state[:on] # => true
```

### Parallel assignment and splats

```rb
a, b = 1, 2
# destructure and ignore
hour, _, day, _, year = [20, 30, 12, 04, 1980]
# splats
title, *rest = ["Mr", "John", "Stewart", "Hopkins"]
rest
# ["John", "Stewart", "Hopkins"]
# also works greedily
title, *rest, sr_jr = ["Mr", "John", "Stewart", "Hopkins", "sr"]
rest
# ["John", "Stewart", "Hopkins"]
```

## Some basics

```rb
v = "nope"
v.class
=> String
v.class.superclass
=> Object

3.class
=> Integer

def double(e); e * 2; end
double(2)
=> 4
# and even (wtf)
double([2])
=> [2, 2]
# and even...
double(_)
=> [2, 2, 2, 2]
# surprise though
double({})
# NoMethodError (undefined method `*' for {}:Hash)

# Methods can end with a question mark.
# This is convention for when the method returns a boolean
class Radio
   is_on?
     state.active == true
   end
end

# Methods can end with !
# This is convention for when the mthod does something dangerous
# (like mutating) or slightly unexpected
a = " hi  "
a.trim
=> "hi"
puts a
=> " hi  "
a.trim! # mutates
puts a
"hi"

# parens are optional for method invocation
double 2
double(2)
```

## Classes

- start with capital letter
- user camelcase

Use `@` to assign instance variables.

```rb
class Guitar
  def play(note)
    @note = note
    note
  end
end

strato = Guitar.new
strato.play('re')
strato.inspect
# <Guitar:0x00007fb3eb827840 @note="re">
p strato # same as inspect
# <Guitar:0x00007fb3eb827840 @note="re">
```

instance variables are **private** by default.

```rb
g.note # error
```

Instance methods are **public** by default.

To access an instance variable we have to define a method that returns it.

```rb
class Guitar
  #... as above
  def note
    @note
  end
end
```

While this works it can become tedious quickly. The alternative is to use **attribute accessors**.

```rb
class Guitar
  attr_accessor :note # READ AND WRITE
  attr_reader :note # READ ONLY
  attr_writer :note # WRITE ONLY
end
```

This makes it messy when we are defining local variables. Look at this:

```rb
class Train
  attr_accessor :speed, :make, :color, :destination

  def get_state
    speed = 22 # if instance.speed is 35, and we set speed = 22 here...
    [speed, destination] # what is the returned speed here?
  end
end
```

To circumvent it, Ruby wants you to use `self` to use the accessor

```rb
class Train
  attr_accessor :speed, :make, :color, :destination

  def get_state
    speed = 22
    [self.speed, destination] # returned speed is 35
  end
end
```

Initial args are defined within the initialize method:

```rb
class Tube
  def initialize(line, destination)
    @line = line
    @destination = destination
  end
end

Tube.new("Victoria", "Seven Sisters")
```

## Inheritance

A class can only inherit from 1 class at a time. Multiple inheritance is not supported per se.
If not specified, a class inherits from Object.

Private methods are **inherited** by the subclasses, hence can be used.

```rb
class Product
  attr_accessor :price

  private

  def calc_price(tax, shipping)
    price + tax + shipping
  end
end

class Laptop < Product
  attr_accessor :shipping

  def get_price(tax, shipping)
     calc_price(tax, shipping) # this will work
  end
end

l = Laptop.new
l.price = 200
l.get_price(20, 30) # 250
```

## Executable class body

One can put any arbitrary code inside a class body, and it will be executed.

```rb
def move
  puts "moving"
end

class Train
  puts "initialising"

  move
end
# the next 2 lines are logged straight away at runtime
> initialising
> moving
```

## Open classes

If a class is defined and then another one is created with the same name, the 1st one is augmented with the methods of the 2nd.
Even an instance that was created before the 2nd definition will inherit these new methods, as long as they are used after the 2nd definition.
This is quite surprising...

```rb
class Tube
  def move
    puts "moving"
  end
end

t = Tube.new

class Tube
  def destination=(station)
    @destination = station
  end

  def log
    puts " destination is #{@destination}"
  end
end

t.destination=("Waterloo")
t.log
# destination is Waterloo
```

If we redefine an existing method it will be monkey patched with the new definition.

```rb
class Tube
  def move
     puts "moving faster"
  end
end
t.move
# moving faster
```

NOTE: this also works for stdlib methods!

```rb
class String
  def size
    puts "this method is now broken"
    12
  end
end

s = "hello"
s.size
# "this method is now broken"
# > 12
```

This can be exploited to patch broken 3rd party libs methods.

## Class equality

This is nice.

```rb
class Tube
  attr_reader :number
  def initialize(number)
    @number = number
  end
end

t1 = Tube.new("123")
t2 = Tube.new("123")

t1 == t2
# false

# BUT I can...
class Tube
  def ==(otherTube)
     number == otherTube.number
  end
end

t1 == t2
# true
```

Note that, if we review all we've seen so far the above can also be called like this:

```rb
t1.==(t2)
# true
t1.== t2
# true
```

This is because parens are optional when invoking a method.

## Flow control

### If/else

```rb
cond = true
if cond
  "all good"
else
  "something is off"
end

# or as 1 liner
if cond then "all good" else "something is off" end
# nothe that hre we use the "then" keyword

# or
"all good" if cond
```

### Truthy/falsy

Only `false` and `nil` evaluate to false.
Everythig else will be truthy.

```
""
0
[]
```

All of these are truthy.

### If not

One can use `unless`.

### Conditional initilisation

A pattern that is very used, initialise a class only if not done already.

```rb
tube ||= Tube.new
```

This has a quirk though, because it relies on the truthiness/falsiness of `tube`.
While in this case it looks ok, look at this:

```rb
flag? ||= true
# true
flag? = false
# false
flag? ||= true
# false
```

While I purposefully set flag to true on the last line, the value was not set.

### And, or, ||, &&

**Warning**: these are not really interchangeable!

- `and` and `or` have a much lower precedence than the `&&` and `||` operators
- `&&` has higher precedence than `||`
- `and` and `or` has the same precedence

### Case

```rb
day = 2
case day
when 0 then "Monday"
when 1 then "Tuesday"
when 2 then "Wednesday"
else
  "I have no idea"
end
#...
```

### Begin/end

Begin is a way to group multiple expressions into one expression.
`Begin` creates its own execution context.
A common usage is to catch errors and recover gracefully.

```rb
begin
  some_non_existing_method()
rescue
  puts "all good"
end
```

## Iterators

### Each

```rb
[1,2,3].each do |a|
  puts a
end

# same as
[1,2,3].each { |a| puts a }
```

## Exceptions

All exceptions in Ruby [here](https://airbrake.io/blog/ruby-exception-handling/ruby-exception-classes) and more below.

`rescue` catches all exceptions.
To catch exceptions per type:

```rb
begin
  some_non_existing_method()
rescue NoMethodError
  puts "all good"
rescue ZeroDivisionError
  puts "can't divide by 0"
end
```

To inspect the type of exception once can map the exception object to a variable like so:

```rb
begin
  some_non_existing_method()
rescue StandardError => msg
  puts msg
  puts e.backtrace.inspect
end
```

Only 1 exception will be caught, so **order matters**, one wants to list rescue blocks from more specific to more generic.

```rb
begin
  some_non_existing_method()
rescue NoMethodError
  puts "all good"
rescue ZeroDivisionError
  puts "can't divide by 0"
rescue StandardError
  puts "soemthing happened"
end
```

### Raising exceptions

```rb
begin
  name = "Andy"
  if name != "andy"
    raise NameError.new "The name doesn't match"
  end
end
# NameError: The error doesn't match
```

To execute some logic after any sort of exception is raised, use `ensure`:

```rb
begin
  some_non_existing_method()
rescue NoMethodError
  puts "all good"
rescue ZeroDivisionError
  puts "can't divide by 0"
rescue StandardError
  puts "soemthing happened"
ensure
  puts "this will always execute after any exception is thrown"
end
```

If you want tp execute code in case there were no exceptions, but after those checks, you can add an `else`:

```rb
begin
  3 + 1
rescue StandardError
  puts "soemthing happened"
else
  # this will execute if no exceptions is thrown
  puts "else clause"
ensure
  puts "this will always execute after any exception is thrown"
end
```

Note that the ensure will **always** run.

## Methods in depth

### Varargs

```rb
def aggregate_by_type(type, *numbers)
  agg = numbers.reduce(0) { |sum, i| sum + i }
  puts "#{type}: #{agg}"
end
aggregate_by_type("oranges", 32, 34, 55, 89, 33)
# oranges: 243
```

### Named args & default values

```rb
# implementation
def aggregate_results(list: [], key_name: :amount, description: "the result of this call is ")

end
```

Note that the default values are **required** in the method definition.

More on this:

```rb
# default value. Note that this is NOT a named arg
def aggregate_results(list = [])

end
```

## Blocks, Procs & Lambdas

https://blog.appsignal.com/2018/09/04/ruby-magic-closures-in-ruby-blocks-procs-and-lambdas.html

## Constants

Not really constants in Ruby, they can be reassigned.

```rb
MAX_AGE = 206000
# later on...
MAX_AGE = 334000 # totally ok, you'll just get a warning
```

## Modules

- used for namespacing

```rb
module API
  def self.get_users
    # ...
  end
end

API.get_users

# can also contains classes
module API
  class Users
    def get_by_id(id)
      #...
    end
  end
end

API::Users.get_by_id(1)

# can also contains classes
module V2
  module API
    class Users
      def get_by_id(id)
        #...
      end
    end
  end
end

V2::API::Users.get_by_id(1)
```

- used for mixins

```rb
module ErrorReporter
  def timeLogger
    puts Time.now
  end
end

class User
  include ErrorReporter
end

u = User.new
u.timeLogger
```
