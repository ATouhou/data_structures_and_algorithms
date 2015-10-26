## Hash Tables (aka Hash Maps)

A Hash Map is one of the most fundamental and commonly-used
collection data structures. An implementation appears in most
modern programming languages, and its versatility and performance
characteristics make it something of a "swiss army knife" of
data structures.

So why is a Hash Map (this, by the way, is where Ruby gets the name for its `Hash` class)
so powerful? As we will see, a Hash Map has 2 essential properties:

1. Ability to associate between arbitrary keys and arbitrary values
2. Ability to insert and retrieve values in constant time

### Associativity

So what do we mean by "associate"? We say a data structure
is associative when it allows us to define a relationship
between 2 values and retrieve one in response to the other.

We've actually already worked frequently with one fundamental
associative data structure: the __Array__.

Arrays define associative relationships between numeric indices and values contained in
the array -- if we want to look something up in an array,
we need to either know its numeric index so that we can go
directly to that position in the array, __or__ we have to
iterate through every element in the array and look at
each one to see if it's the element we want.

This is great if our data is ordered and if we are able to
consistently retrieve it by going directly to its numeric
index, but that isn't always the case.

I often want to associate between _arbitrary_ keys
and values (strings, objects, other arrays, etc)? For example, I want
to associate the string "pizza" with the value "awesome",
and I don't want to have to define an explicit ordering
in the process. Additionally, I want to be able to add a whole lot of keys and
values into the map and maintain a speedy lookup time.

This is where a Hash Table comes in.

### Hash Table Structure

To implement this data structure, we'll rely on a few key
tools:

1. A Hashing Algorithm for uniquely differentiating pieces of data.
Many languages already provide this -- in this example we'll look at
using the SHA-1 implementation included with Ruby's Digest library.
2. An internal array which we'll ultimately use to store data
3. An additional abstraction for handling "collisions" between
hashcodes within the data structure.

To some extent, a Hash Table is a bit of Data Structure "sleight of hand"
-- it allows us to translate arbitrary data (i.e. hash keys) into
numeric array indices, and thus take advantage of the speedy
index lookups we get out of the box with an array.

### Hash Function Basics

This ability to translate arbitrary data to numeric array
indices is the fundamental operation of a hash table, and
the Hashing function is what makes it possible.

Hash functions are a special type of algorithm designed
to turn arbitrarily long data into consistent-length
hash "digests" that represent that data.

Importantly, a good hash function will do several things:

1. Be fast (this is important to help us maintain our hash table performance)
2. Be 1-way (given the input you can generate the digest, but with the digest
there is no way to guess the input)
3. Be collision-resistant (it should be highly unlikely to find 2 inputs that
generate the same digest value)
4. Be consistent -- hashing the same value should always yield the same digest

Let's look at a ruby example using SHA-1:

```ruby
[13] pry(main)> require "digest"
=> true
[14] pry(main)> Digest::SHA1.hexdigest("pizza")
=> "1f6ccd2be75f1cc94a22a773eea8f8aeb5c68217"
```

Here I used SHA1 to hash the string "pizza" into a 40-character hexadecimal
digest number. One thing to note is that a SHA1 hash will always be 40 hexadecimal
digits (160 bits) -- this is true whether I hash the string "pizza" or the
entire contents of _War and Peace_.

Additionally keep in mind that the digest produced is actually a _number_. Ruby
happens to represent it for us in this example as a string of hexadecimal
characters, but we can convert it to its actual numeric value like so:

```
[14] pry(main)> Digest::SHA1.hexdigest("pizza")
=> "1f6ccd2be75f1cc94a22a773eea8f8aeb5c68217"
[15] pry(main)> Digest::SHA1.hexdigest("pizza").to_i(16)
=> 975987071262755080377722350727279193143145743181
```

We'll exploit this property of the hash digests in the next step.

### Hash Table Algorithm

So how does it work? In short, when creating a new Hash Table, we'll
allocate an internal array to actually store our data. Later on
we may introduce an additional abstraction around Nodes or Elements,
but the array is going to be our fundamental storage mechanism
for the data.

When asked to insert a Key/Value pair into the array, we need to
map it to a numeric index within our internal array. Here's where
we will leverage the hash function -- it produces a numeric
representation of the data in question (the digest). Our hash
table will use this numeric digest, modulo the length of our
array, to determine which spot the element should be inserted into.

Let's look at a rough example in ruby:

```ruby
require "digest"

table = Array.new(10) # 10-element hash table to start
key = "pizza"
value = "awesome"

digest = Digest::SHA1.hexdigest(key).to_i(16)
position = digest % table.length

table[position] = value
```
