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

Before

```ruby
class Game
  SYSTEMS = ['SNES', 'PS1', 'Genesis']

  attr_accessor :name, :year, :system

  def runs_on_snes?
    self.system == 'SNES'
  end

  def runs_on_ps1?
    self.system == 'PS1'
  end

  def runs_on_genesis?
    self.system == 'Genesis'
  end
end
```

After

```ruby
class Game
  SYSTEMS = ['SNES', 'PS1', 'Genesis']

  attr_accessor :name, :year, :system
  
  SYSTEMS.each do |var|
    define_method "runs_on_#{var.downcase}?" do
      system == var
    end
  end

end

```

Before
```ruby
class Library
  attr_accessor :games

  def each(&block)
    games.each(&block)
  end

  def map(&block)
    games.map(&block)
  end

  def select(&block)
    games.select(&block)
  end
end
```

After
```ruby
class Library
  attr_accessor :games
 
  [:each, :map, :select].each do |iterator|
    define_method iterator do |&block|
      games.send iterator, &block
    end
  end
end
```

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

##### The method method

Methods are also objects.So, to get this object, we can use the method method.

```ruby
tweets = ["First Tweet","Second Tweet"]
timeline = Timeline.new(tweets)

#Getting the method
content_method = timeline.method(:contents)
#Invoking
content_method.call
```

#### Understanding and using self


###### Class Eval
Sets self to the given class and executes a block

```ruby
Game.class_eval do
  def self.find_by_owner
  end
end
```

Using class_eval and defining a method in execution time

```ruby
class LibraryManager
  def self.make_available(klass, user)
    klass.class_eval do
      define_method "lend_to_#{user}" do
        
      end
    end
  end
end
```

##### Instance_eval

Sets self to the given instance and executes a block

```ruby
tweet = Tweet.new
tweet.instance_eval do
  self.status = "Changing the tweet's status"
end
```
Inside the block self is the Tweet instance

```ruby
contra_game = Game.new('Contra')
contra_game.instance_eval do
  self.owner = 'Alice'
end
```

```ruby
class Game
  def initialize(&block)
    instance_eval(&block) if block_given?
  end

  def owner(name=nil)
    if name
      @owner = name
    else
      @owner
    end
  end
end
```


#### Method missing

```ruby
class Library
  def method_missing(method_name,*args)
    puts method_name    
  end

end
```

Sending to other class

```ruby
class Library
  def initialize(console)
    @manager = console
  end

  def method_missing(name, *args)
    @manager.send(name,*args)
  end
end
```

##### Delegating for other class if contains a substring in the method name

```ruby
class Library
  def initialize(console)
    @manager = console
  end

  def method_missing(name, *args)
    match = name.to_s.include?("atari")
    if match
    	@manager.send(name, *args)
    else
     super
    end
  end
end
```


##### Define method revisited

```ruby
class Library
  SYSTEMS = ['arcade', 'atari', 'pc']

  attr_accessor :games

  def method_missing(name, *args)
    system = name.to_s
    if SYSTEMS.include?(system)
      self.class.class_eval do
        define_method(system) do
          find_by_system(system)
        end
      end
      else
        super
    end
  end

  private

  def find_by_system(system)
    games.select { |game| game.system == system }
  end
end
```

##### Domain Specific Language

A DSL is a language that has terminology and constructs designed for a specific domain. If you're familiar with rails, ActiveRecord is a good example of a DSL.
