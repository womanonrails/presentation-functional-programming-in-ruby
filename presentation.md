class: center, middle, inverse

# Functional Programming in Ruby
Agnieszka Matysek

---

class: middle

# Agenda
- Introduction
- Procs
- Lambdas
- Closures
- Summary
- Example


---

class: center, middle, inverse

# Simple block

---

class: middle

# Block

```ruby
[1, 2, 3, 4].each do |item|
  p item
end
```

```ruby
1
2
3
4
 => [1, 2, 3, 4]
```

???

- .strong[block] - nameless function in Ruby
- only one per method
- as last argument
- .strong[p] - return text from ""
- .strong[puts] - return nil

---

class: middle

# Define our block

```ruby
def my_own_block
  p 'before'
  yield
  p 'after'
end
```

---

class: middle

# How can we run our block?

```ruby
my_own_block
"before"
LocalJumpError: no block given (yield)
  from (irb):6:in `my_own_block'
  from (irb):9
  from /home/agnieszka/.rvm/rubies/ruby-2.1.3/bin/irb:11:in `<main>'
```

## Simple fix


```ruby
def my_own_block
  p 'before'
  yield if block_given?
  p 'after'
end
```

```ruby
"before"
"after"
 => "after"
```

---

class: middle

# How put something to the block?

```ruby
my_own_block(5)
ArgumentError: wrong number of arguments (1 for 0)
  from (irb):4:in `my_own_block'
  from (irb):12
  from /home/agnieszka/.rvm/rubies/ruby-2.1.3/bin/irb:11:in `<main>'
```

## Simple fix

```ruby
my_own_block { 5 }
"before"
"after"
 => "after"
```

```ruby
my_own_block { p 5 }
"before"
5
"after"
 => "after"

```

???

- parameter is not a argument
- do block but we don't see
- when we want to see we can put print into
- we can use .strong[do end] instead .strong[{}]
- block always return last line!

---

class: center, middle, inverse

# Proc object

???

- we check class of block

---

class: middle

# Block class

```ruby
def my_block(&block)
  p 'before'
  p block.class
  block.call
  p 'after'
end
```

## Execute

```ruby
my_block { p 4 }
"before"
Proc
4
"after"
 => "after"
```

---

class: middle

# Many Procs in one function

```ruby
def run_proc(first, last)
  first.call
  last.call
end

proc1 = Proc.new { p 'first' }
proc2 = Proc.new { p 'last' }
```

## Execute

```ruby
run_proc proc1, proc2
"first"
"last"
 => "last"
```

---

class: middle

# Other declaration

```ruby
def run_proc
  p 'before'
  my_proc = Proc.new
  my_proc.call
  p 'after'
end
```

## Execute

```ruby
run_proc { p 6 }
"before"
6
"after"
 => "after"
```

???

- How its work?
- If .strong[Proc.new] don't have a block of code
- it look for Proc in current context and use it
- its implisyty work

---

class: center, middle, inverse

# How call Proc object?

---

class: middle

# Proc with parameter

```ruby
my_proc = Proc.new do |item|
  p "Text: #{item}"
end
```

## Execute

```ruby
my_proc.call 10
my_proc.(20)
my_proc[30]
my_proc === 40
```

???

- if we call Proc like this .strong[my_proc.(20)] we must use ()

---

class: middle

# Proc tricks

```ruby
proc1 = Proc.new { |number| number % 3 == 0 }
proc2 = Proc.new { |number| number % 3 == 1 }
```

```ruby
case 3
when proc1 then p 'proc1'
when proc2 then p 'proc2'
else
  p 'not a proc'
end
```

## Realy tricky

```ruby
(4..7) === 5 => true
```

???

- case use === comarision so we can use .strong[Proc] in .strong[case]
- case use also .strong[Range] so we can do tricks in Ruby
---

class: center, middle, inverse

# Lambdas

???

- almost like Proc, but have some different behaviors
- in Ruby 1.8 .strong[proc] is lambda but in 1.9 proc is Proc object

---

class: middle

## 1. Arguments controll

```ruby
my_proc = Proc.new { |item|  p "==#{item}==" }
my_lambda = lambda { |item|  p "==#{item}==" }
```

#### Arity
```ruby
my_proc.arity
 => 1
my_lambda.arity
 => 1
```

#### Execute
```ruby
my_proc.call
"===="
 => "===="

my_lambda.call
ArgumentError: wrong number of arguments (0 for 1)
  from (irb):64:in `block in irb_binding'
  from (irb):69:in `call'
  from (irb):69
  from /home/agnieszka/.rvm/rubies/ruby-2.1.3/bin/irb:11:in `<main>'
```

???

- in .strong[lambda] is like a method

---

class: middle

### Array of arguments

```ruby
lambda { |*items| }.arity
 => -1
```

### New lambda declaration

```ruby
my_lambda = -> { p 'Is it still work?' }
my_lambda.call
"Is it still work?"
 => "Is it still work?"
```

### But ...

```ruby
my_lambda.class
 => Proc

my_lambda.lambda?
 => true
```

???

- .strong[lambda] class is always .strong[Proc]

---

class: middle

### 2. Using with return

```ruby
def run(proc)
  p 'before'
  proc.call
  p 'after'
end
```
#### Execute lambda

```ruby
run lambda { p 'In'; return }
"before"
"In"
"after"
 => "after"
```
#### Execute proc

```ruby
run proc { p 'In'; return }
"before"
"In"
LocalJumpError: unexpected return
  from (irb):77:in `block in irb_binding'
  from (irb):73:in `call'
  from (irb):73:in `run'
  from (irb):77
  from /home/agnieszka/.rvm/rubies/ruby-2.1.3/bin/irb:11:in `<main>'
```

???

- for .strong[lambda] everythink is ok, finish lambda and go back to method run
- for .strong[proc] we have error, stop on "In"
- .strong[proc] don't return current context
- .strong[proc] want to return context where he was defined (in this case it is main context)

---

class: middle

### Change the conetxt

```ruby
def change_context
  run lambda { p 'In'; return }
  run proc { p 'In'; return }
end
```

#### Execute first lambda

```ruby
"before"
"In"
"after"
"before"
"In"
 => nil
```

#### Execute first proc

```ruby
"before"
"In"
 => nil
```

???

- in this case context for .strong[proc] is .strong[change_context] method
- .strong[proc] return this context

---

class: middle

### 3. Lambda can return value even the creation context is gone

```ruby
def first(closure)
  p 'start first'
  p "closure: #{closure.call}"
  p 'end first'
end

def second
  p 'start second'
  p "first return: #{first(lambda { return 'lambda' })}"
  p 'end second'
end
```

#### Execute

```ruby
second
"start second"
"start first"
"closure: lambda"
"end first"
"first return: end first"
"end second"
 => "end second"
```

---

class: middle

### 3. Proc after return, finish all methods (first and second)

```ruby
def first(closure)
  p 'start first'
  p "closure: #{closure.call}"
  p 'end first'
end

def second
  p 'start second'
  p "first return: #{first(proc { return 'proc' })}"
  p 'end second'
end
```

#### Execute

```ruby
second
"start second"
"start first"
 => "proc"
```

???

- using exception with proc is unsafe

---

class: center, middle, inverse

# Closures

---

class: middle

# Magic start

```ruby
def run(proc)
  proc.call
end
```
## Execute

```ruby
name = 'Agnieszka'
my_proc = proc { p name }
run my_proc
"Agnieszka"
 => "Agnieszka"
```

???

- Before proc context we declare .strong[name]
- when we run our proc he see the name varialbe

---

## What happend when variable is not created before?

```ruby
my_proc = proc { p last_name }
run my_proc
NameError: undefined local variable or method `last_name' for main:Object
  from (irb):121:in `block in irb_binding'
  from (irb):116:in `call'
  from (irb):116:in `run'
  from (irb):122
  from /home/agnieszka/.rvm/rubies/ruby-2.1.3/bin/irb:11:in `<main>'
```

#### Second try

```ruby
last_name = 'Matysek'
run my_proc
NameError: undefined local variable or method `last_name' for main:Object
  from (irb):121:in `block in irb_binding'
  from (irb):116:in `call'
  from (irb):116:in `run'
  from (irb):124
  from /home/agnieszka/.rvm/rubies/ruby-2.1.3/bin/irb:11:in `<main>'
```

???

- .strong[last_name] variable don't exist when proc is create so they can not use it

---

class: middle

# What's going on here?

```ruby
name = 'Agnieszka'
my_proc = proc { p name }
name = 'Ula'
run my_proc
=> ?
```

---

class: middle

# Magic

```ruby
name = 'Agnieszka'
my_proc = proc { p name }
name = 'Ula'
run my_proc
"Ula"
 => "Ula"
```

???

- .strong[proc] remember reference to this variable not a content

---

class: middle

# Where can we use this?
#### Defining function which create other function

```ruby
def multiple(m)
  lambda { |n| n * m }
end

double = multiple(2)
triple = multiple(3)
```

#### Execute
```ruby
double[5] => 10
triple[5] => 15
```
???

- .strong[m] comes in and exist
- .strong[multiple] method lambda see .strong[m] (2 or 3)
- lambda has an argument .strong[n] so we put 5 in this case

---

class: center, middle, inverse

# Summary

---

class: middle

# Types of closures

- block + yield
- block + &b
- Proc.new
- proc
- lambda
- method

???

- I'll back to method in a minute

---

class: middle

# How we call yield_run or block_run?

```ruby
def yield_run
  yield
end
```

```ruby
def block_run(&block)
  block.call
end
```

???

- .strong[yield] is not Proc type, it is something like .strong[block.call]

---

class: middle

# The same

```ruby
block_run { 'my proc' }
block_run { p 'my proc' }

block_run do
  'my proc'
end

my_proc = lambda { p 'my proc' }
block_run &my_proc

block_run { my_proc }
block_run { my_proc.call }
```

---

class: middle

# What we get?

```ruby
name = 'Agnieszka'
my_proc = lambda do
  name = 'Maria'
  p name
end
```

```ruby
my_proc.call => ?
name => ?
```

---

class: middle

# Answer

```ruby
name = 'Agnieszka'
my_proc = lambda do
  name = 'Maria'
  p name
end
```

```ruby
my_proc.call
"Maria"
 => "Maria"

name => "Maria"
```

---

class: middle

# What we get?

```ruby
def run
  a = 10
  yield
  yield
  p "#{a}"
end
```

```ruby
a = 5
run { a += 1 } => ?
a => ?
```

---

class: middle

# Answer

```ruby
def run
  a = 10
  yield
  yield
  p "#{a}"
end
```

```ruby
a = 5
run { a += 1 }
"10"
 => "10"

a => 7
```

---

class: middle

# Method as closure

```ruby
def my_method
  p 'Hello word'
end

my_proc = method(:my_method)
my_proc.call
```

???

.strong[TIPS:]
- we can run yield with argument
- we can use proc in Design Paterns (ex. Strategy)
- .strong[.lambda?] - check if something is lambda or not
- Java have also closure as anonymous inner classes

---

class: middle, center, inverse

# Lisp in Ruby ;]

---

class: middle

## Lisp recursive data structure:

```ruby
[1,[2,[3]]]     <--- car=1, cdr=[2,[3]]
   [2,[3]]      <--- car=2, cdr=[3]
      [3]       <--- car=3, cdr=nil
```

---

class: middle

## Lisp list

```ruby
class LispyEnumerable
  include Enumerable

  def initialize(tree)
    @tree = tree
  end

  def each
    while @tree
      car, cdr = @tree
      yield car
      @tree = cdr
    end
  end
end
```

#### Execute

```ruby
list = [1,[2,[3]]]
LispyEnumerable.new(list).each { |item| p item }
1
2
3
 => nil
```

---

class: middle

## Lazy list

```ruby
class LazyLispyEnumerable
  include Enumerable

  def initialize(tree)
    @tree = tree
  end

  def each
    while @tree
      car, cdr = @tree.call
      yield car
      @tree = cdr
    end
  end
end
```

#### Execute

```ruby
list = lambda { [1, lambda { [2, lambda { [3] } ] } ] }
LazyLispyEnumerable.new(list).each { |item| p item }
1
2
3
 => nil
```

???

- what changed?
- They do next part of list when it is call (not earlier)

---

class: middle

## How we can see this?

```ruby
list = lambda do
  p '1 is call'
  [1, lambda do
    p '2 is call'
    [2, lambda { p '3 is call'; [3] }]
  end]
end
```

#### Execute

```ruby
LazyLispyEnumerable.new(list).each { |item| p item }
"1 is call"
1
"2 is call"
2
"3 is call"
3
 => nil
```

---

class: middle

# Fibbonacci:

```ruby
def fibo(a,b)
  lambda { [a, fibo(b, a + b)] }
end
```

#### Execute

```ruby
LazyLispyEnumerable.new(fibo(1,1)).each do |item|
  puts item
  break if item > 100
end
1
1
2
3
5
8
13
21
34
55
89
144
 => nil
```

???
- .strong[fibo] - would go into infinite recursion if it weren't in a lambda
- .strong[fibo] in lambda is lazy
- This is not recursion (in some way - we can call infinite loop)
- .strong[Haskell] - use lazy evaluation for everything, by default

---

class: middle, inverse

# Bibliography

- [Closure in Ruby](http://innig.net/software/ruby/closures-in-ruby)
- [An Introduction to Proc, Lambdas and Closures in Ruby](https://www.youtube.com/watch?v=VBC-G6hahWA)
- [Design Patterns in Ruby - Russ Olsen](http://www.amazon.com/Design-Patterns-Ruby-Russ-Olsen/dp/0321490452)
- [Ruby doc](http://ruby-doc.org/)

---

class: middle, center

.small-image[![Womanonrails](./images/womanonrails.png)]
### Agnieszka Matysek
[@womanonrails](https://twitter.com/womanonrails)

amatysek@fractalsoft.org

