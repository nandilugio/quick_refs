Ruby
====

**/!\ NOTE:** This document uses _true statemens_ to describe the language. To aid on this purpose, we'll use `all_equal` to compare multiple values. For the curious, can be defined as: `def all_equal(*a) a.all? { |x| x == a[0] } end`.

TODO: talk about the language, the MRI and other implementations, etc.

Types, Literals and Operators
-----------------------------

```ruby
# Booleans

true.class == TrueClass
false.class == FalseClass
nil.class == NilClass

true == true
true != false
true && true
true || false
!false

#TODO: and, or, not

# Numbers

16.class == Integer
all_equal 16, 0x10, 0b10000, 020
all_equal 1000000, 1_000_000

120.0.class == Float
all_equal 120.0, 1.2e2, 12e1

(3+2i).class == Complex
3+2i == Complex(3, 2)
(3+2i).real == 3
(3+2i).imag == 2

(3/2r).class == Rational
3/2r == Rational(3, 2)
(3/2r).numerator == 3
(3/2r).denominator == 2

(1..3).class == Range
(1..3) == Range.new(1, 3)
(1..3).to_a == [1, 2, 3]
(1..3).begin == 1
(1..3).end == 3

1 + 1 == 2
1 - 1 == 0
1 * 2 == 2
5 / 2 == 2
5 % 2 == 1
5 / 2.0 == 2.5
5.0 / 2 == 2.5
5 ** 2 == 25

1 == 1
1 != 2
1 < 2; 2 > 1
1 <= 2; 1 >= 1
1 > 2 ? 1 : 2 == 2  # Ternary operator

# Symbols

:foo.class == Symbol
:foo.object_id == :foo.object_id  # singletons
:foo.to_s == 'foo'
'3'.to_sym == :'3'  # to write symbols that are not valid identifiers

# Strings

'foo'.class == String
'foo'.object_id != 'foo'.object_id  # not singletons

bar = 'baz'
"foo #{bar}" == 'foo baz'  # Double quotes allow interpolation

'foo' + 'bar' == 'foobar'
'foo' * 3 == 'foofoofoo'

s = 'foobar'
s[0] == 'f'  # same indexing as arrays
s[2, 2] = 'baz'  # same assignments as arrays
s == 'fobazar' 

'foo'.to_sym == :foo
'1.1'.to_i == 1
'1.1'.to_f == 1.1

'foo'.length == 3
'foo'.upcase == 'FOO'
'foo'.capitalize == 'Foo'

# Heredocs

hd = <<-A_NAME
  Heredoc
  String
A_NAME
hd == "  Heredoc\n  String\n"  # Note the indentation

hd = <<~A_NAME  # Squiggly Heredoc to remove indentation
  Heredoc
  String
A_NAME
hd == "Heredoc\nString\n"  # Note the trailing newline

hd = <<~A_NAME.chomp  # #chomp to remove trailing newline (any method can be called)
  Heredoc
  String
A_NAME
hd == "Heredoc\nString"  # No indentation, no trailing newline

# Arrays

[1, 2, 3].class == Array
all_equal [1, 2, 3], Array[1, 2, 3], Array(1..3), Array.new(3) { |i| i+1 }
all_equal ["a", "b", "c"], %w(a b c)
all_equal [:a, :b, :c], %i(a b c)

a = [1, 2, 3, 4, 5, 6]

a[1] == 2
all_equal a[0], a.first, 1
all_equal a[-1], a.last, 6
a[2..4] == [3, 4, 5]  # start..end
a[2, 4] == [3, 4, 5, 6]  # start, length

a[2..4] = [7, 8]
a == [1, 2, 7, 8, 6]
a[1,2] = 0
a == [1, 0, 8, 6]

a = [1, 2]
a << 3
a == [1, 2, 3]
a.push(4)
a == [1, 2, 3, 4]
a.pop == 4
a == [1, 2, 3]
a.shift == 1
a == [2, 3]
a.unshift(1)
a == [1, 2, 3]

[1, 2] + [3, 4] == [1, 2, 3, 4]
[1, 2, 2, 3, 3, 4] - [2, 3] == [1, 4]
[1, 2, 3, 4, 5, 6] & [2, 4, 6, 8] == [2, 4, 6]
[1, 2, 3, 4, 5, 6] | [2, 4, 6, 8] == [1, 2, 3, 4, 5, 6, 8]
[1, 2] * 3 == [1, 2, 1, 2, 1, 2]

# /!\ NOTE: See below for methods that take blocks { |x| ... }
[1, 2, 3, 4, 5, 6].each { |x| puts x }
[1, 2, 3, 4, 5, 6].map { |x| x * 2 } == [2, 4, 6, 8, 10, 12]
[1, 2, 3, 4, 5, 6].select { |x| x % 2 == 0 } == [2, 4, 6]  # alias #filter
[1, 2, 3, 4, 5, 6].reject { |x| x % 2 == 0 } == [1, 3, 5]
[1, 2, 3, 4, 5, 6].reduce(0) { |acc, x| acc + x } == 21
[1, 2, 3, 4, 5, 6].reduce(0, :+) == 21

# Hashes

{a: 1, b: 2}.class == Hash
all_equal({a: 1, b: 2}, Hash[a: 1, b: 2], Hash[[[:a, 1], [:b, 2]]])

h = {a: 1, b: 2}
h[:a] == 1
h.keys == [:a, :b]
h.values == [1, 2]

h2 = {"c" => 3, "d" => 4}  # Keys can be any type
h2.keys == ["c", "d"]
h.merge(h2) == {a: 1, b: 2, "c" => 3, "d" => 4}

# Structs

Struct.new(:a, :b).class == Class
S = Struct.new(:a, :b)
S.class == Class
s = S.new(1, 2)
s.inspect == "#<struct S a=1, b=2>"  # gets the name of the first constant it is assigned to (TODO: black magic??)
s.class == S  # If it was not assigned to a constant, it would be an anonymous class like #<Class:0x00007f9d1c0a1b80>

all_equal s.a, s[:a], s[0], 1
s.members == [:a, :b]
s.values == [1, 2]
s.to_h == {a: 1, b: 2}

# Defining struct methods by:
# - passing a block to #new
S = Struct.new(:a, :b) do
  def foo
    "foo"
  end
end
# - reopening the class (see below)
S = Struct.new(:a, :b)
class S
  def foo
    "foo"
  end
end
# - directly subclassing it (see below)
class S < Struct.new(:a, :b)
  def foo
    "foo"
  end
end
# are all equivalent. In all cases:
s = S.new(1, 2)
s.a == 1
s.b == 2
s.foo == "foo"

# Regular Expressions

/abc/.class == Regexp
/abc/ == Regexp.new('abc')
(/abc/ =~ 'abc') == 0
(/abc/ =~ 'def') == nil
(/abc/ =~ '123abcdef') == 3
# TODO: expand

Time.now.class == Time
t = Time.new(2018, 1, 1, 0, 0, 0.125, "+00:00")
t.to_i == 1514764800
t.to_f == 1514764800.125
t.to_s == "2018-01-01 00:00:00 +0000"
# TODO: Time deserves its own section
```

Variables and Constants
-----------------------

```ruby
# Variables are dynamically typed. They're not declared, just assigned.
# They're local by default, but can be global, instance or class.
# They're lowercase by convention, usually snake_case.

foo = 1
foo = 'bar'
$foo = 1  # global
@foo = 1  # instance, see below
@@foo = 1  # class, see below

# Constants are not really constants, they can be reassigned, but a warning is issued.
# They're uppercase by convention, usually ALL_CAPS but CamelCase for types (clases).

FOO = 1
FOO = 2  # warning: already initialized constant FOO

# Constants can be defined inside classes and modules, and are scoped to them.
# They can be accessed from outside using the scope resolution operator `::`, or
# from inside using the constant lookup operator, also `::`. See below. TODO check this! AI generated
```

Control Structures
------------------

```ruby
if true  # or `unless false`
  1
elsif false  # only for `if`
  2
else
  3
end == 1

case 'b'
when 'a'
  1
when 'b', 'c'
  2
else
  3
end == 2

i = 0
while i < 3  # or `until i == 3`
  i += 1
end
i == 3

eights = 0
for i in 1..10
  next if i < 5  # skips first numbers
  eights += 1 if i == 8
  puts "i=#{i}, eights=#{eights}"
  redo if i == 8 && eights < 3 # redo iterations according to condition
  break if i == 9  # breaks early
end
i == 9

begin
  # do many things and implicitly return
  3
end == 3
```

Methods, Blocks, Procs and Lambdas
----------------------------------

```ruby
# All functions in Ruby are called _methods_, wether they're defined globally or in a class or module.

def double(x)
  x * 2  # return is implicit
end
method(:double).class == Method  # to access the method object and pass it around

double(2) == 4
(double 2) == 4  # parens are optional, when not ambiguous

# Methods receive
# - positional arguments, which can have defaults, and captured as an array using `*`
# - keyword arguments, which can have defaults, and captured as a hash using `**`
# - a block, which can be captured explicitly using `&block`, or implicitly (just ommit the arg)
#   if only using `yield` in the method. See below.
def foo(pos1, pos2='pos2_default', *other_pos, kw1: 'kw1_default', kw2:, **other_kw, &block)
  # `yield` calls the block. No need for the `&block` arg for this
  block_result = yield(9) if block_given?

  # To process the block directly, it can be explicitly captured using the `&block` arg as we did here
  block_class = block.class  # Proc

  "pos1: #{pos1}, pos2: #{pos2}, other_pos: #{other_pos}\n" +
  "kw1: #{kw1}, kw2: #{kw2}, other_kw: #{other_kw}\n" +
  "block class: #{block_class}\n" +
  "block result: #{block_result}"
end

# Note that:
# - pos2 has a default but is passed so we can also pass some extra positionals
# - kw1 is not passed and default is used
# - the block is captured explicitly with `&block`, but we could have skipped it if only using `yield`
foo(1, 2, 3, 4, kw2: 5, kw3: 6, kw4: 7) { |x| x } == <<~RESULT.chomp
  pos1: 1, pos2: 2, other_pos: [3, 4]
  kw1: kw1_default, kw2: 5, other_kw: {:kw3=>6, :kw4=>7}
  block class: Proc
  block result: 9
RESULT

# The `*` and `**` operators can also be used to deconstruct arrays and hashes into positional and keyword args.
# Also note we ommit the block in this call. Note the effect in the result.
pos_args = [1, 2, 3, 4]
kw_args = {kw2: 5, kw3: 6, kw4: 7}
foo(*pos_args, **kw_args) == <<~RESULT.chomp
  pos1: 1, pos2: 2, other_pos: [3, 4]
  kw1: kw1_default, kw2: 5, other_kw: {:kw3=>6, :kw4=>7}
  block class: NilClass
  block result:
RESULT

# Blocks are like anonymous functions that can be passed to methods as an implicit or explicit last argument.
# Above, `{ |x| x }` is the block. In the method, the received block can be treated either:
# - implicitly, executing it via `yield`, or
# - explicitly, capturing it as above with `&block` and processing it at will.
# They don't check arity, extra args are ignored, and missing args are nil.
# They're written as a one-liner `{ |args| ... }` or in its multi-line form `do |args| ... end`.

# Procs are block encapsulations that can be passed around. As such, they don't check arity either.
# In procs, the `return` keyword exits the embracing method, not the block or proc. If written outside the method,
# throws LocalJumpError when called. To return from a block or proc, use `next` or `break`.
# They're written as `Proc.new <block>` or just `proc <block>`.

p = Proc.new { |x| "received #{x}" }
p = proc { |x| "received #{x}" }  # Equivalent using Kernel#proc
p.class == Proc
p.inspect  # => "#<Proc:0x00007f9d1c0a1b80>"

# Procs don't check arity. Extra args are ignored, missing args are nil.
p.call(2) == "received 2"
p.call(2, 3) == "received 2"
p.call == "received "

# Alternative call syntaxes
p.(2) == 4
p[2] == 4

# Procs deconstruct single array args if the proc has multiple args
p = proc { |a, b| a + b }
p.call(2, 3) == 5
p.call([2, 3]) == 5

def foo() yield; 1 end
foo { 2 } == 1
foo { return 2 }  # LocalJumpError: return is written outside #foo
foo { break 2 } == 2  # `break` returns the caller of the block

def foo
  Proc.new { return 1 }.call
  2
end
foo == 1  # `return` is defined in the method, thus returns from it when called. Procs don't have their own scope.

def first_greater_than_3(arr)
  arr.each { |x| return x if x > 3 }  # This is how the above `return` behavior can be useful
  nil
end
first_greater_than_3([1, 2, 3, 4, 5]) == 4

# Lambdas are a special kind of block that _do_ check arity and return normally.
# They're written as `->(args) { ... }` or `lambda <block>`.

l = ->(x) { x * 2 }
l = lambda { |x| x * 2 }  # Equivalent using Kernel#lambda
l.class == Proc
l.inspect  # => "#<Proc:0x00007f9d1c0a1b80 (lambda)>"  <-- shows it's a lambda

l.call(2) == 4
l.call(2, 3)  # ArgumentError: lambdas check arity
l.(2) == 4
l[2] == 4

def foo
  lambda { return 1 }.call  # the expression evals to 1. Here the result is discarded to make the point
  2
end
foo == 2  # lambdas return normally

# The `&` syntax can be used to convert procs and lambdas to blocks and viceversa.
# Note how we captured the block using `&block` in the method definition above.

# Procs and lambdas can be passed as blocks to methods as last args with `&`
def f() yield * 2 end
f(&proc { 2 }) == 4
f(&lambda { 2 }) == 4

# Methods can be passed as blocks to methods as last args with `&`
def f(x) x * 2 end
[1, 2, 3].map(&method(:f)) == [2, 4, 6]
```

Classes
-------

This example shows the basic syntax and some of the most important features of Ruby classes.

```ruby
class Vehicle  # inherits from Object by default

  @@vehicle_names = []  # class variable, shared by all `Vehicle` instances and its subclasses instances (here `Car`s)
  # TODO: comment on class instance variables here

  #### Instance methods ####

  # There's only instance methods in Ruby. In the class body the "def target" is the class method table used by `Vehicle` instances.
  # Inside instance methods, the default receiver `self` is the instance, being `self.class == Vehicle`.

  # Constructor
  def initialize(name)
    if @@vehicle_names.include?(name)
      raise ArgumentError, "Vehicle name #{name} already taken"
    end
    @@vehicle_names << name
    @name = name
    # Note there's no need to declare instance variables
    # They're also hidden from the outside, so we need accessors to read and write them (see below)
  end

  # Auto @name getter. There's also `attr_writer` and `attr_accessor` for both
  attr_reader :name

  # Explicit @name setter, since we need custom logic
  def name=(name)
    if @@vehicle_names.include?(name)
      raise ArgumentError, "Vehicle name #{name} already taken"
    end
    @@vehicle_names.delete(@name)
    @@vehicle_names << name
    @name = name
  end

  # Operator overloading
  def +(other)
    "Train of #{@name} and #{other.name}!!"
  end

  # Regular method
  def start
    "Starting #{@name}"
  end

  #### Class methods ####

  # There's no actual class methods in Ruby. `self` in the class body is just `Vehicle`, which is an instance of `Class`.
  # As an instance, when defining a method over it, we're actually defining a method on its eigenclass `*Vehicle`, the
  # one between `Vehicle` and its class `Class`.

  def self.vehicle_count
    @@vehicle_names.length
  end
end


class Car < Vehicle
  def initialize(name, diesel: false)  # Override
    super(name)  # calls `Vehicle` constructor to ensure full initialization
    @diesel = diesel
  end

  def start  # Override
    message = super  # calls `Vehicle#start`
    if @diesel
      message[0] = message[0].downcase
      message = "Preheating diesel engine and " + message
    end
    message
  end
  alias_method :turn_key, :start
end


pilder = Vehicle.new("Pilder")
pilder.class == Vehicle
pilder.class.ancestors == [Vehicle, Object, Kernel, BasicObject]
pilder.start == "Starting Pilder"

Vehicle.vehicle_count == 1

kitt = Car.new("Kit")
kitt.class == Car
kitt.class.ancestors == [Car, Vehicle, Object, Kernel, BasicObject]
kitt.start == "Starting Kit"  # overriden behavior though not diesel. Same message

Vehicle.vehicle_count == 2

kitt.name = "Kitt"  # setter. Fixes name
kitt.name = "Kitt"  # second call produces ArgumentError: Vehicle name Kitt already taken
kitt.name == "Kitt"  # getter

herbie = Car.new("Herbie", diesel: true)
repeat = Car.new("Herbie")  # ArgumentError: Vehicle name Herbie already taken
herbie.start == "Preheating diesel engine and starting Herbie"  # overriden behavior
herbie.turn_key == herbie.start  # alias_method

Vehicle.vehicle_count == 3

kitt + herbie == "Train of Kitt and Herbie!!"
```

This example focuses on the different kinds of variables and their scopes: instance, class and class instance.

```ruby
class Parent
  #### Class variables ####
  # Part of the language. Shared between:
  # - `Parent` class and instances
  # - `Parent` subclasses (here `Child`) and their instances
  @@family_things = []
  def self.family_things
    @@family_things
  end 
  def family_things
    @@family_things
  end

  #### Class instance variables ####
  # The "class" in the name is misleading. Just an instance var in `Parent`, instance of `Class`.
  # Shared between:
  # - `Parent` class
  # - `Parent` instances
  @shared_things  = []
  def self.shared_things
    @shared_things
  end
  def shared_things
    self.class.shared_things  # Notice `@` doesn't work here. We access through the class
  end

  #### Instance variables ####
  # Not shared. Each instance has its own copy.
  def initialize
    @my_things = []
  end
  attr_reader :my_things
end

class Child < Parent
  # Accessors are inherited, but not the instance variable. We initialize it here.
  # Note how we're then creating a different array for Childs.
  @shared_things = []
end

mama = Parent.new
papa = Parent.new
joey = Child.new
suzy = Child.new

Parent.family_things << :house
papa.family_things   << :vacuum
mama.shared_things   << :car
papa.shared_things   << :blender
papa.my_things       << :quadcopter
joey.my_things       << :bike
suzy.my_things       << :doll
joey.shared_things   << :puzzle
suzy.shared_things   << :blocks

p Parent.family_things #=> [:house, :vacuum]
p Child.family_things  #=> [:house, :vacuum]
p papa.family_things   #=> [:house, :vacuum]
p mama.family_things   #=> [:house, :vacuum]
p joey.family_things   #=> [:house, :vacuum]
p suzy.family_things   #=> [:house, :vacuum]

p Parent.shared_things #=> [:car, :blender]
p papa.shared_things   #=> [:car, :blender]
p mama.shared_things   #=> [:car, :blender]
p Child.shared_things  #=> [:puzzle, :blocks]  
p joey.shared_things   #=> [:puzzle, :blocks]
p suzy.shared_things   #=> [:puzzle, :blocks]

p papa.my_things       #=> [:quadcopter]
p mama.my_things       #=> []
p joey.my_things       #=> [:bike]
p suzy.my_things       #=> [:doll]
```

TODO: reopening, private & protected (like friend in C++)

Modules, Mixins and Inheritance
-------------------------------

Modules are like classes, but they can't be instantiated so not meant to store state.
They're used mainly to group methods and constants.

```ruby
module OneNamespace
  FOO = 1

  # Methods are defined the same way as in classes.
  # To access them without an instance, they must be defined on the module itself as `self.<name>`.
  def self.bar
    2
  end
end

OneNamespace.class == Module
OneNamespace::FOO == 1
OneNamespace.bar == 2
```

Modules can be included in classes to add functionality as instance vars and methods.
Modules used for that purpose are called mixins. They can be included in other modules too.

```ruby
module Voice
  attr_writer :greeting

  def greet
    @greeting || "hey!"
  end
end

class Person
  include Voice
end

p = Person.new
p.class.ancestors == [Person, Voice, Object, Kernel, BasicObject]  # In order of inclusion (and member resolution)
p.greet == "hey!"
p.greeting = "hello"
p.greet == "hello"
```

Modules can also extend classes to add functionality as "class methods" (actually instance methods of its class' eigenclass).
These are called class mixins. They can be extended in other modules too.
TODO: also @@class_variables?

```ruby
module SpeciesDescription
  def description
    "I'm a #{self.name.downcase}"
  end
end

class Dog
  extend SpeciesDescription
end

Dog.description == "I'm a dog"
```

By using the hooks `included` and `extended`, modules can be notified when they're included or extended in a class or module.
The following example shows a typical pattern: adding class methods to a regular mixin:

```ruby
module Plugin
  def instance_method
    "I'm a plugin instance method"
  end

  def self.included(klass)
    klass.extend ClassMethods
  end

  module ClassMethods
    def class_method
      "I'm a plugin class method"
    end
  end
end

class PluginUser
  include Plugin
end

u = PluginUser.new
u.instance_method == "I'm a plugin instance method"
PluginUser.class_method == "I'm a plugin class method"
```

Errors and Exceptions
---------------------

```ruby

# Exceptions are raised with `raise` or `fail`. They can be rescued with `rescue`.
# They're instances of classes that inherit from Exception.

raise "foo"  # RuntimeError: foo
raise ArgumentError, "foo"  # ArgumentError: foo
raise ArgumentError.new("foo")  # ArgumentError: foo
fail "foo"  # alias for `raise`, same syntax

# Exceptions can be rescued with `rescue ... else ... ensure` within either
#   - `begin ... end` sections
#   - `do ... end` blocks
#   - method bodies
#   - end of loops

begin
  raise ArgumentError, "foo"
rescue ArgumentError => e if e.message == "foo"
  "rescued ArgumentError with message foo"
rescue ArgumentError => e
  "rescued ArgumentError with message that's not foo"
rescue IndexError => e, NameError
  # We can also fix the error somehow and
  retry  # restart the begin block
rescue => e
  "rescued any other exception"
else
  "there was no exception"
ensure
  puts "always executed, not returned"
end == "rescued ArgumentError with message foo"
```

Pattern Matching
----------------

TODO

Other
-----

TODO: useful things for scripts `if __FILE__ == $0`, `BEGIN` and `END`, data at the end of files, etc.
TODO: basic IO (gets, puts), tap
TODO: frozen, tainted, etc.

Debugging
---------

TODO binding.irb
p (like puts but returns)
pp (pretty print)?

The Object Model, Metaprogramming and Reflection
------------------------------------------------

From:
- The Ruby Object Model by Dave Thomas: https://www.youtube.com/watch?v=X2sgQ38UDVY
- A Deep Dive into the Ruby Object Model by Peter Cooper: https://www.youtube.com/watch?v=by5fFOBhtPQ

### C implementation (MRI)

Objects are structs with pointers to:
- Class (klass): the type of the object
- State (iv_tbl): a table for instance variables

Classes are objects too, adding pointers to:
- Behavior (m_tbl): a table for methods
- Parent (super): the superclass type
- Shared State (cvar_tbl): a table for class variables (@@)

#### The singleton, meta, virtual or eigenclass

It's created lazily as soon as you define things on instances, in between the instance and its class.

- instance variables: looked up in `self`
- methods: looked up in `self.class` method table, following the `klass` reference starting one the eigenclass and continuing up the chain of ancestors

`class << some_instance` opens the eigenclass of `some_instance`, sometimes notated as `*some_instance`. The following 3 defs are equivalent, and used as "class methods":
class X
  def X.foo; end
  def self.foo; end
  class << self
    def foo; end
  end
end

"default definee" is the target of the `def` keyword when not specified (using `def target.something`).
It may be different from `self`, like on a class body, where `self` is the class object, but the
default definee is the class method table.
By context:
- in the top level, it's `main`, and self is `main` TODO: check
- in a class body, it's the class method table, and self is the class object TODO: check
- in a method body, ?? TODO
- on #instance_eval:  default definee := *instance; self := instance (so `def self.f` goes to *instance too)
  eg. `C.instance_eval { def f; end; def self.g; end }` defines `C.f` and `C.g`
  eg. `c = C.new; c.instance_eval { def f; end; def self.g; end }` defines `c.f` and `c.g`
- on #class_eval: self and "default definee" are the class TODO: check

Modules are defined with `module <name> ... end`. They're objects of class Module.
They can be reopened and modified at any time.

methods defined in modules are instance methods, and are only available when the module is included in a class.
inclusion works by linking the module's method table in the class's ancestors chain, just before the class itself,
so the search path for methods (and the #ancestors list) is:
  (*class,) class, included modules, parent, included modules in parent, ..., Object, Kernel (module included in Object), BasicObject 

methods defined as `self.<name>` are defined on the module itself (instance of Module), and are available using `Module.name` or when the module is extended in a class

TODO: method_missing, instance_eval, instance_exec, define_method, etc.
TODO: self.included(klass) and self.extended(klass) callbacks
TODO: metaprogramming

### Some important modules

Kernel  # globally available methods
Comparable
Enumerable
Math
Process
Signal
GC

### Some important classes

BasicObject
  Module  # A collection of methods and constants
    Class  # Adds instance creation capabilities to store state
  Object (includes Kernel)  # Instances of classes, storing state
    TrueClass
    FalseClass
    NilClass
    Numeric (includes Comparable)
      Integer
      Float
      Complex
      Rational
    Range (includes Enumerable)
    Symbol
    String (includes Comparable, Enumerable)
    Array (includes Enumerable)
    Hash (includes Enumerable)
    Struct (includes Enumerable)
    Regexp
    Time
    Dir
    IO
      File
    Proc
  Exception
    fatal
    SystemExit
    StandardError
      RuntimeError
      ArgumentError
      IndexError
      NameError
        NoMethodError
      ZeroDivisionError

### Navigating the class hierarchy

```ruby
Integer.superclass == Numeric
Integer.ancestors == [Integer, Numeric, Comparable, Object, Kernel, BasicObject] # In order of inclusion (and member resolution)

Integer.included_modules == [Comparable, Kernel]

# With Activesupport
Numeric.subclasses == [Complex, Rational, Float, Integer]
BasicObject.descendants == [Numeric, Integer, Float, ..., Symbol, String, Array, ..., Exception, ...]
```

### Inspecting objects

```ruby
a = Array.new
a.class == Array
a.methods == [:to_h, :include?, :&, :*, :+, :-, :at, :fetch, :last, :union, :difference, ...]
Array.instance_methods == a.methods
Array.instance_methods(false) == [...]  # Only methods defined in Array, not inherited
Array.instance_methods(false) == Array.instance_methods - Array.superclass.instance_methods
a.object_id  # Unique integer identifier
a.inspect == "[]"  # Can be overridden. Defaults to the class name and id formatted as "#<ClassName:0x007f9d1c0a1b80>"
a.to_s == "[]"  # Can be overridden. Defaults to #inspect
a.method(:each)  # => #<Method: Array#each()>  <-- method is from Array
a.method(:reduce)  # => #<Method: Array(Enumerable)#reduce(*)>  <-- method is from Enumerable
# TODO: respond_to?, method_missing, etc.
```

