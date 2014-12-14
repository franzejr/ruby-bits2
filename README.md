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

##### Using multiple lambdas


```ruby
class Library
  attr_accessor :games

  def initialize(games)
    @games = games
  end

  def exec_game(name, action,error_handler)
    game = games.detect { |game| game.name == name }
    begin  
      action.call(game)
    rescue 
      error_handler.call
    end
  end
end
```

##### Block to a Proc and passing it to a function which receives a block

Before

```ruby
library = Library.new(GAMES)
library.each { |game| puts "#{game.name} (#{game.system}) - #{game.year}" }
```

After

```ruby
my_code = Proc.new {
  |game| puts "#{game.name} (#{game.system}) - #{game.year}" 
  
  }

library = Library.new(GAMES)

library.each(&my_code)
```

##### Capturing blocks

Before

```ruby
class Library
  attr_accessor :games

  def initialize(games)
    @games = games
  end

  def each
    games.each do |game|
      yield game
    end
  end
end
```

After

```ruby
class Library
  attr_accessor :games

  def initialize(games)
    @games = games
  end

  def each(&block)
    games.each do |game|
      block.call(game)
    end
  end
end
```


##### Passing blocks

Before

```ruby
class Library
  attr_accessor :games

  def initialize(games)
    @games = games
  end

  def each(&block)
    games.each do |game|
      block.call(game)
    end
  end
end
```

After

```ruby
class Library
  attr_accessor :games

  def initialize(games)
    @games = games
  end

  def each(&block)
    games.each(&block)
  end
end
```

##### Using & in a map function ==> symbol to proc

Before

```ruby
class Library
  attr_accessor :games

  def initialize(games)
    @games = games
  end

  def names
    games.map { |game| game.name }
  end
end
```

After

```ruby
class Library
  attr_accessor :games

  def initialize(games)
    @games = games
  end

  def names
    games.map(&:name)
  end
end
```

##### Veryfing if a block is given

```ruby
def list
  games.each do |game|
    if block_given?
      puts yield game
    else
      puts game.name
    end
  end
end
```

#### Dynamic Classes and Methods
