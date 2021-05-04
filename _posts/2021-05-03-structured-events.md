---
layout: post
title: "Writing a Tracing Framework, Part 1: Structured Events"
series: minitrace
---

_Disclaimer: this blog and the [minitrace](https://github.com/ajvondrak/minitrace/) gem are my personal works. They are not affiliated with my employer, [Honeycomb](https://honeycomb.io/). Views expressed are my own._

A while back, I wrote a talk about [the path from logs to traces](https://ajvondrak.github.io/soapbox/2021/02/25/the-path-from-logs-to-traces/) explaining how flat, human-readable log files can be transformed into rich, machine-parseable data structures. It's an important conceptual overview, but leveraging traces requires the proper tooling. So I want to go one step further by building a tracing library from the ground up.

Specifically, we'll look at the development of [minitrace](https://github.com/ajvondrak/minitrace), a minimalist tracing framework written as a Ruby gem. The aim isn't to be minimal in *functionality*, but in implementation. We'll get to the same features found in "big" libraries like the [Honeycomb Beeline](https://github.com/honeycombio/beeline-ruby) or [OpenTelemetry](https://github.com/open-telemetry/opentelemetry-ruby), but we'll do it with less code and make it easier to understand. Don't worry too much about knowing Ruby; the lessons are broadly applicable to most major object-oriented programming languages.

<!-- more -->

## Events

As covered in the talk, the basic unit of data we work with is a *structured event*. At its core, an event is composed of *fields* that map arbitrary string names to arbitrarily complex values. Developers are probably most familiar with this structure in the form of JSON, but in principle we could serialize an event any number of ways: XML, Protobuf, URI query strings, whatever. As long as we can map names to values, we're said to have "structure". In contrast, log files are composed of lines that have no inherent structure, since a line is just a single arbitrary string&mdash;usually a message in a natural language like English.

The idea is that your running code will dynamically add fields to an ongoing event. The game of tracing, then, is to develop conventions around which fields we add to which events.

To represent events in Ruby, we'll make a class named `Minitrace::Event` that maintains a good ol' fashioned hash table. It's Ruby, so types are dynamic, but it's understood that the hash is from `String` (the field name) to `Object` (i.e., any type). While you could manipulate this hash of fields directly, it's good practice to define a standard interface. So to start, we have the following class definition:

```ruby
class Minitrace::Event
  attr_reader :fields

  def initialize
    @fields = {}
  end

  def add_field(field, value)
    @fields[field] = value
    self
  end

  def add_fields(fields)
    @fields.merge!(fields)
    self
  end
end
```

An event object is initialized with an empty hash table of fields, which are exposed using the `#fields` attribute:

```ruby
event = Minitrace::Event.new
event.fields #=> {}
```

Fields are added one at a time by `#add_field`:

```ruby
event.add_field("name", "value")
event.fields #=> { "name" => "value" }
```

The method returns the current `Minitrace::Event` instance so that you can easily chain calls:

```ruby
event = Minitrace::Event.new
event.add_field("a", 1).add_field("b", 2)
event.fields #=> { "a" => 1, "b" => 2 }
```

You can also merge in another hash of name-value pairs in one shot with `#add_fields`:

```ruby
event = Minitrace::Event.new
event.add_fields("a" => 1, "b" => 2)
event.fields #=> { "a" => 1, "b" => 2 }
```

## Backends

Of course, if all we needed was a hash table, we would have little use for the `Minitrace::Event` class. The point of events is to *fire* them to some backend once they're finished. The most useful sort of backend will process each event's data and store it for later querying. Such backends include services like [Honeycomb](https://honeycomb.io/), [LightStep](https://lightstep.com), [AWS X-Ray](https://aws.amazon.com/xray/), and [several others](https://opentelemetry.io/vendors/).

But the mechanics of firing events will differ between vendors, protocols, etc. For a generic tracing library, we don't want to be tied down to anything specific. So we'll define an abstract base class called `Minitrace::Backend`:

```ruby
class Minitrace::Backend
  def process(event)
    raise NotImplementedError
  end
end
```

Its sole interface is the `#process` method, which takes in a `Minitrace::Event` instance and does something (anything!) with it. The base class doesn't actually implement anything&mdash;it just raises an error. Concrete backends inherit from the base class and provide their own implementations of the `#process` method (which they have to do, or else they'll inherit the error-raising version).

For now, I don't want to worry about sending data to a service. However, we need a concrete backend, and it'll be useful for unit tests to "spy" on the events we try to process. So let's add a backend that just keeps an array of events it has already seen:

```ruby
class Minitrace::Backends::Spy < Minitrace::Backend
  def processed
    @processed ||= []
  end

  def process(event)
    processed << event
  end
end
```

Here, the `#processed` method returns the value of an instance variable, which is initially an empty array (using `||=` like this is a common Rubyism). The `#process` implementation just pushes the event onto the end of that array.

We'll then configure the minitrace library with a global backend:

```ruby
module Minitrace
  class << self
    attr_accessor :backend
  end
end
```

This defines the `Minitrace.backend` attribute, which we can assign to any instance of a `Minitrace::Backend`. Since we only have the one concrete class right now, we'd go with this:

```ruby
Minitrace.backend = Minitrace::Backends::Spy.new
```

Finally, we can hook events up to the backend by way of a `Minitrace::Event#fire` method:

```ruby
class Minitrace::Event
  def fire
    return if @fired
    Minitrace.backend.process(self)
    @fired = true
  end
end
```

This takes the current event instance and processes it with the global minitrace backend. For our spy backend, this just means the event gets pushed onto the `#processed` array:

```ruby
event = Minitrace::Event.new
event.add_field("test", "event")
event.fire

Minitrace.backend.processed
#=> [#<Minitrace::Event:0x00007fe951969118 @fields={"test"=>"event"}, @fired=true>]
```

Also notice that there's an extra guard in the `#fire` method to prevent double-firing. This keeps our implementation from accidentally processing the same event twice:

```ruby
100.times { event.fire }
Minitrace.backend.processed.count #=> 1
```

If necessary, individual backends might incorporate their own retry logic to send the event data again. But this would be a specialized case handled outside the general event-firing interface.

## Up Next

In the future, we'll build more elaborate backends and get our events into a service like Honeycomb (with which I'm most familiar). But before we get to that, we have enough here to start fleshing out how to turn plain events into rich traces. In the next installment, we'll devise a friendlier API for creating & manipulating events.

You can find the code from this post&mdash;including tests&mdash;in the following commits on GitHub:
* <samp><a href="https://github.com/ajvondrak/minitrace/commit/a578df8a05ea39a005ad4db6a1ea9db2a346d248">a578df8</a> Add Minitrace::Event</samp>
* <samp><a href="https://github.com/ajvondrak/minitrace/commit/478dab8038f2f724f16f8510516bd49290db0680">478dab8</a> Sketch out the structure for backends</samp>
* <samp><a href="https://github.com/ajvondrak/minitrace/commit/924c0725f9d56093a781527cfebc807a628c4a5a">924c072</a> Fire events to backend</samp>
