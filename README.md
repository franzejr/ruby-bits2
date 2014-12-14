ruby-bits2
==========


#### Blocks, Procs and Lambdas

```ruby
def call_this_block_twice
  yield
  yield
end
```

```ruby
call_this_block_twice {puts "Hello World!"}
```

What if we want to store this block for execute it later?

Two ways:
- Proc
- Lambda

Proc.new
my_proc = Proc.new  { puts "test" }
my_proc.call  # => test

Lambda
my_proc = lambda { puts "test" }
my_proc.call  # => test
