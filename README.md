# Ruby Objects: The "Has Many" Relationship

## Objectives

1. Describe the "has many" relationship between Ruby objects.
2. Build classes that produce objects with a belongs to and has many relationship.
3. Explain why we need to associate objects in this way.

## Introduction

We know that the programs we write are meant to model real-world environments. This is because the programs we write are designed to carry out real-world jobs and solve real-world problems. Whether you're creating an app that connects users around the world in some kind of social network or writing a program for a major university that manages their course offerings and students, your code will need to be able to realistically map the relationships between different entities.

We already know about the "belongs to" relationship. Let's say we have a `Song` class that produces individual song objects. Each song belongs to the artist that wrote it. We can build that relationship by creating an `attr_accessor` in the `Song` class for `artist`:

```ruby
class Song
  attr_accessor :artist, :name

  def initialize(name)
    @name = name
  end
end
```

If we also have an `Artist` class that looks like this:

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
  end
end
```

We can set an individual instance of `Song` equal to an instance of the `Artist` class like this:

```ruby
ninetynine_problems = Song.new("99 Problems")
jay_z = Artist.new("Jay-Z")

ninetynine_problems.artist = jay_z

ninetynine_problems.artist.name
  # => "Jay-Z"
```

The benefit here is that in setting the `artist=` method equal to a real instance of the `Artist` class, instead of equal to a simple string, we are associating our song to a robust object that has its own attributes and behaviors.

For example, in the code above, we are calling the `#name` method on the artist of `ninetynine_problems`. With method chaining like this, we can do even more with our code.

The inverse of the "belongs to" relationship is the "has many" relationship. If a song belongs to an artist, then an artist should be able to have many songs. This makes sense in the real-world––most musical artists have authored and performed many more than one song.

Let's take a closer look.

## The "has many" Relationship

How can we represent an object's "having many" of something? One option is to store every song an artist has within an array. It would look something like this:

```ruby
class Artist
  attr_accessor :songs, :name

  def initialize(name)
    @name = name
    @songs = []
  end

end
```

Now when we initialize a new artist, they start off with an empty collection of songs. The issue with this method though is that whenever you add an artist to a song, you _also_ have to go and duplicate this information by adding a song to its artist like this:

```ruby
ninetynine_problems = Song.new("99 Problems")
jay_z = Artist.new("Jay-Z")

ninetynine_problems.artist = jay_z
jay_z.songs << ninetynine_problems
```

So now we end up with duplicate data. Every time an artist gets a new song, we have to write this information in two places, on the song AND on the artist. Think about it, now if you want to remove the song `ninetynine_problems` from your library, you have to _also_ know to go remove it from Jay Z's `@songs` instance variable. There must be a better way!

I know! An artist could _search_ through all the songs to find the songs that belong to them! This way, we would only have to store each song with an instance variable `@artist` set equal to the artist object.

![seraching](http://i.giphy.com/3orif4omu38hZIdEGY.gif "Searching through a collection")

To do this of course, we'll need to have all of our songs in one place so artists can search them later. First let's set up our Song class with an `@@all` class variable and a getter method for for that array to accommodate that:

```ruby
class Song
  attr_accessor :artist, :name

  @@all = []

  def self.all
    @@all
  end

  def initialize(name)
    @name = name
    @@all << self
  end
end
```

Now every time we create a new song, it will get added to the class variable `@@all` array so ALL songs are now stored in one _searchable_ place. If an artist wants to find their songs, they would just have to search through all the songs to find the ones that belong to them. So if I'm searching through a collection and I want to return an array of all objects matching a certain rule (`song.artist == my_artist`), then I think `#select` is the enumerator for us to use! Let's see what that would look like:

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def songs
    Song.all.select { | song | song.artist == self}
  end
end
```

Now we can execute the following lines of code:

```ruby
jay_z = Artist.new("Jay Z")
aesop_rock = Artist.new("Aesop Rock")
empire_state_of_mind = Song.new("Empire State of Mind")
empire_state_of_mind.artist = jay_z
lotta_years = Song.new("Lotta Years")
lotta_years.artist = aesop_rock
none_shall_pass = Song.new("None Shall Pass")
none_shall_pass.artist = aesop_rock

jay_z.songs[0].name
#=> "Empire State of Mind"
aesop_rock.songs.count
#=> 2
```

So just by adding our artist object to our songs, our artists can now find all of their songs!

### Relating Objects with "belongs to" and "has many"

So now if a song has an artist object stored in their `@artist` instance variable, it's really easy for us to follow this association from both directions:

```ruby
# given an artist, we can find all their songs
jay_z.songs
#=> [#<Song:0x007ffc02133368 @name="Empire State of Mind", @genre="rap", @artist=#<Artist:0x007ffc01847268 @name="Jay Z">>]

# given a song, we can find it's artist
lotta_years.artist
#=> #<Artist:0x007ffc01836288 @name="Aesop Rock">

# and finally, given a song, we can find the artist and then find all that artists other songs!

lotta_years.artist.songs
#=>  => [#<Song:0x007ffc02140040 @name="Lotta Years", @artist=#<Artist:0x007ffc01836288 @name="Aesop Rock">>, #<Song:0x007ffc030283f0 @name="None Shall Pass", @artist=#<Artist:0x007ffc01836288 @name="Aesop Rock">>]
```

And to do all of this, we never have to store the song data in the artist object!

Now what would this look like if we want to add Genres into the mix as well? Well first we'll have to define the relationship between songs, genres, and artists. For the purpose of this readme, let's say that a song belongs to a genre and an artist has many genres _through_ it's songs. A Genre has many songs and has many artists. This feels like it makes sense because if an artist doesn't have any songs then they can't have any genres. And if Aesop Rock has a song in the 'rap' genre and a song in the 'pop' genre, then he would have the genres 'rap' and 'pop'.

So lets just start simple first. We need a Genre class that has a name attribute. Because a Genre "has many" songs and a song "belongs to" a genre, we'll store the genre on the song class just like we did with artists and songs. The Genre class will also need a way to find all it's songs just like Artist had.

```ruby
class Genre
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def songs
    Song.all.select { | song | song.artist == self}
  end
end
```

And now we'll have to update our Song class to have a `@genre` as well:

```ruby
class Song
  attr_accessor :artist, :name, :genre

  ...
end
```

Now that we have Genres added to the mix, we can do some other useful things with this collection of real song objects, such as iterate over them and collect their genres:

```ruby
aesop_rock.songs.collect do |song|
  song.genre.name
end
  # => ["rap", "pop"]
```

So we could turn the code snippet above into a method on artist called `#genres` so that an artist could easily call all of their genres!

## Extending the Association and Cleaning up our Code

The code we have so far is pretty good. The best thing about it though, is that it accommodates future change. We've built solid associations between our `Artist`, `Genre` and `Song` class via our has many/belongs to code. With this foundation we can make our code even better in the following ways:

## Allow Initialization of a Song with an Artist and Genre

It's kind of a pain to have to create a song and then add the artist and the genre to it. It kind of feels like this could be done all in one step. Let's add some arguments to our `#initialize` method to make this work:

```ruby
class Song
  ...

  def initialize(name, artist, genre)
    @name = name
    @artist = artist
    @genre = genre
    @@all << self
  end
end
```

Now we can easily create our song with all the associated objects! Let's see what that looks like:

```ruby
jay_z = Artist.new("Jay Z")
aesop_rock = Artist.new("Aesop Rock")

rap = Genre.new("rap")
pop = Genre.new("pop")

empire_state_of_mind = Song.new("Empire State of Mind", jay_z, rap)
lotta_years = Song.new("Lotta Years", aesop_rock, pop)
none_shall_pass = Song.new("None Shall Pass", aesop_rock, rap)
```

This is a bit easier now! But I just want to see if we can make it a bit better. What if we still want the option to be able to create a song with just passing in a name as a strong? So we can change our code just a bit to accommodate this by adding some default arguments for `artist` and `genre`. Now I don't think we want to have a default artist object or a default genre object, instead let's just set these attributes to `nil`. This way it will be clear that they are not yet set:

```ruby
class Song
  ...

  def initialize(name, artist=nil, genre=nil)
    @name = name
    @artist = artist
    @genre = genre
    @@all << self
  end
end
```

Perfect. Now the code snippet we wrote earlier works, and so will this one!

### The `#add_song_by_name` Method

As we can see, it can become somewhat tedious to keep having to call `Song.new` and pass in both our new song's name as a string and our new songs artist as an artist object. It would be really nice if our artist could just create the song. Maybe something like an `#add_song_by_name` method?

```ruby
class Artist
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def add_song_by_name(song_name)
    Song.new(song_name, self)
  end

  def songs
    Song.all.select { | song | song.artist == self}
  end
end
```

Ahh, much better! Now our artists can create their own songs with just a string for a name.

### The `#artist_name` Method

Since we've already set up these great associations between instances of the `Song` and `Artist` class, we can use them to build other helpful methods.

Currently, to access the name of a given song's artist, we have to chain our methods like this:

```ruby
empire_state_of_mind.artist.name
  # => "Jay-Z"
```

That's not very elegant. Wouldn't it be nice if we have one simple and descriptive method that could return the name of a given song's artist? Let's build one!

```ruby
class Song
  ...

  def artist_name
    self.artist.name
  end
```

Now we can call:

```ruby
empire_state_of_mind.artist_name
  # => "Jay-Z"
```

Much better. Notice that we used the `self` keyword inside the `#artist_name` method to refer to the instance of `Song` on which the method is being called. Then we call `#artist` on that song instance. This would return the `Artist` instance associated to the song. Chaining a call to `#name` after that is equivalent to saying: call `#name` on the return value of `self.artist`, i.e. call `#name` on the artist of this song.

These are only a few of the ways in which you can extend, or build on, the foundational has many and belongs to associations.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/ruby-objects-has-many-readme' title='Ruby Objects: The "Has Many" Relationship'>Ruby Objects: The "Has Many" Relationship</a> on Learn.co and start learning to code for free.</p>
