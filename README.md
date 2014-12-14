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

##### Struct

```ruby
class Tweet
  attr_accessor :user, :status
  def initialize(user, status)
    @user, @status = user, status
  end 
end
```

Using Struct to do the same thing

```ruby
Tweet = Struct.new(:user, :status)
```

We'll get the same result:

```ruby
tweet = Tweet.new('Gregg', 'compiling!')
tweet.user # => Gregg
tweet.status # => compiling!
```

##### Struct extra methods

```ruby
Tweet = Struct.new(:user, :status) do 
  def to_s
    "#{user}: #{status}"
  end
end
```

##### Alias method

```ruby
class Timeline
  def initialize(tweets = [])
    @tweets = tweets
  end
  
  def tweets
    @tweets
  end
  
  def contents
    @tweets
  end 
end
```

We've two methods with the same implementation. So, we can use the alias_method.

```ruby
class Timeline
  def initialize(tweets = [])
    @tweets = tweets
  end
  def tweets
    @tweets
end
  alias_method :contents, :tweets
end
```

##### Define Method

We can also define method dynamically.


###### The send method

We've the following ruby class:

```ruby
class Timeline
  
  def initialize(tweets)
    @tweets = tweets
  end
  
  def contents
    @tweets
  end 
private
  def direct_messages
  end 
end
```
We can use the send method to run methods:

```ruby
tweets = ["First Tweet","Second Tweet"]
timeline = Timeline.new(tweets)

#Normal way
timeline.contents

#Using send method
timeline.send(:contents)
timeline.send("contents")
```
Send can also runs private or protected methods. To avoid it, we can use public_send. It prevents running private or protected methods.

