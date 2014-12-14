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

###### Proc.new
```ruby
my_proc = Proc.new  { puts "test" }
my_proc.call  # => test
```

###### Lambda
```ruby
my_proc = lambda { puts "test" }
my_proc.call  # => test
```

###### Block to Lambda

Block

```ruby
tweet = Tweet.new('Ruby Bits!')
tweet.post { puts "Sent!" }
```

```ruby
class Tweet
  def post
    if authenticate?(@user, @password)
      # submit the tweet
      yield
    else
      raise 'Auth Error'
end end
end
```


Lambda

```ruby
tweet = Tweet.new('Ruby Bits!') 
success = -> { puts "Sent!" } 
tweet.post(success)
```

```ruby
class Tweet
  def post(success)
    if authenticate?(@user, @password)
      # submit the tweet
      success.call
    else
      raise 'Auth Error'
end end
end
```

###### Lambda to block

```ruby
tweets = ["First tweet", "Second tweet"] 
tweets.each do |tweet|
  puts tweet
end
```

```ruby
tweets = ["First tweet", "Second tweet"]
printer = lambda { |tweet| puts tweet }
tweets.each(&printer)
```

```
'&' turns proc into block
```

##### Changing Proc into lambda

Proc

```ruby
library = Library.new(GAMES)
print_details = Proc.new do |game|
  puts "#{game.name} (#{game.system}) - #{game.year}"
end
library.exec_game('Contra', print_details)
```

Lambda

```ruby
library = Library.new(GAMES)
print_details = lambda { |game|
  puts "#{game.name} (#{game.system}) - #{game.year}"
  }
library.exec_game('Contra', print_details)
```
