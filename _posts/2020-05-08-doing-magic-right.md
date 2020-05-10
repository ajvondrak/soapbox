---
layout: post
title: On RSpec, Minitest, and Doing Magic Right
---

I started programming Ruby professionally in 2014, which meant I had to learn [RSpec](http://rspec.info/). To this day, it remains the de facto standard for writing tests. And to this day, I still don't really get it.

<!-- more -->

Back then, I didn't get how to use it: contexts, hooks, expectations, mocks. It's a lot to learn, especially for inexperienced programmers. Still, I persisted. I fumbled my way through the syntax. Resources like the [RSpec Style Guide](https://rspec.rubystyle.guide/) taught me some conventions. I wrote test suites large enough to leverage features like [shared examples](https://relishapp.com/rspec/rspec-core/v/3-9/docs/example-groups/shared-examples) and [custom matchers](https://relishapp.com/rspec/rspec-expectations/docs/custom-matchers).

Nowadays, I don't get why to use RSpec in the first place. Is it because the DSL reads vaguely like English? Is it because the output is human readable? Is it because, in the [words of one of its developers](https://stackoverflow.com/a/12480737), "it reifies so many testing concepts into first class objects"?

Those are the things RSpec says on the tin, anyway. I don't fully get why they're virtues. And to the extent that they are, I don't see how RSpec delivers *well* on any of them.

## The DSL

The Ruby community at large is enamored with [DSLs](https://en.wikipedia.org/wiki/Domain-specific_language), be it [Sinatra](http://sinatrarb.com/), [Whenever](https://github.com/javan/whenever), [Rake](https://github.com/ruby/rake), [Chef](https://docs.chef.io/recipes/), or things that probably [don't even count](https://www.infoq.com/news/2007/06/dsl-or-not/). One of the most common criticisms of DSLs is that they're ["magic"](https://en.wikipedia.org/wiki/Magic_%28programming%29).

But I think the problem with RSpec is exactly the opposite: [it's not actually magic](https://www.youtube.com/watch?v=Libc0-0TRg4). If it were magic, I would be able to read it and *not* know what it's doing underneath. But in order to make sense of any of it, I *have* to know how it's implemented.

Take the following example.

```ruby
RSpec.describe Article do
  subject(:article) { described_class.new(title: title) }
  let(:title) { Faker::Lorem.sentence }

  describe '#header' do
    subject(:header) { article.header }

    def words_in(sentence)
      sentence.gsub(/[[:punct:]]/, '').split
    end

    context 'when the title is over 50 characters long' do
      let(:title) { Faker::Lorem.paragraph_by_chars(number: 100) }

      it 'gets truncated' do
        expect(header.length).to be <= 50
        expect(header).to end_with('...')
      end

      it 'keeps words intact' do
        expect(words_in(title)).to match_array(words_in(header))
      end
    end
  end
end
```

Pretty standard RSpec code, right? It's not even doing anything fancy. Yet already I have to grok several things:

* `RSpec.describe` is used at the top level to [avoid namespace pollution](https://relishapp.com/rspec/rspec-core/docs/configuration/global-namespace-dsl).
* Inside the block, you [have to](https://github.com/rspec/rspec-core/blob/9f55fafe62ea367a4801ebed2ca19cf6dfdceb39/lib/rspec/core/example_group.rb#L254-L256) use just `describe`.
* [Example groups](https://relishapp.com/rspec/rspec-core/v/3-9/docs/example-groups) are essentially syntactic sugar for defining classes.
* Knowing this, I remember you can define [arbitrary helper methods](https://relishapp.com/rspec/rspec-core/v/3-9/docs/helper-methods/arbitrary-helper-methods).
* Similarly, I think about how [`subject`](https://relishapp.com/rspec/rspec-core/v/3-9/docs/subject/explicit-subject) and [`let`](https://relishapp.com/rspec/rspec-core/v/3-9/docs/helper-methods/let-and-let) translate down into memoized methods.

And then there are the matchers... When I first started learning RSpec, I always got the syntax wrong. Where do the dots go? the spaces? the underscores? the parentheses? In order to keep any of it straight, I had to know the actual implementation.

I still can't make sense of `expect(header.length).to be <= 50` without the extra mental work of parsing it in my head. I don't see the English, I see Ruby, and my brain has to be the interpreter.

```ruby
expect(header.length).to be <= 50
#=> expect(header.length).to(be.<=(50))

expectation = expect(header.length)
#=> RSpec::Expectations::ExpectationTarget.new(header.length)

matcher = be
#=> RSpec::Matchers::BuiltIn::Be.new

matcher = matcher.<=(50)
#=> RSpec::Matchers::BuiltIn::BeComparedTo.new(50, :<=)

expectation.to(matcher)
#=> ...
```

I can't help but see all the guts. And as used to RSpec as I am, it still trips me up. Or at best, it slows me down:

* I know `end_with` and `match_array` by heart as some of the many [built-in matchers](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/built-in-matchers). But the list is long; I still find myself having to look it up.
* Some matchers have to evaluate an expression multiple times, so you need to wrap things in blocks. Even more punctuation to get in the way of your "English-y" syntax, like `expect { counter.increment }.to change { counter.value }.from(0).to(1)`.
* There are infinitely many [predicate matchers](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/built-in-matchers/predicate-matchers), since they use [`method_missing`](https://ruby-doc.org/core-2.7.1/BasicObject.html#method-i-method_missing) to delegate to the expectation target. Imagine the frustration of junior me, unable to find any documentation on some weird-looking matcher I saw in the wild, only to find it was calling a method dynamically.
* If `be_xxx` wasn't enough, there could be [custom matchers](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/custom-matchers/define-a-custom-matcher), [noun-phrase aliases](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/composing-matchers), or [negated matchers](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/define-negated-matcher) floating around the namespace. I've wasted a not-insignificant amount of time hunting down the names & implementations of matchers littered around the test suite.

So why even bother with this language in the first place? I still have to understand it in terms of the Ruby it seeks to abstract. This isn't magic. It's a [leaky abstraction](https://en.wikipedia.org/wiki/Leaky_abstraction).

## The Documentation

I guess even if I don't find the code readable, the output is pretty legible.

```
Array
  behaves like a collection
    initialized with 3 items
      says it has three items
    #include?
      with an item that is in the collection
        returns true
      with an item that is not in the collection
        returns false
```

Is it really useful, though?

To start, it's barely proper English (even ignoring flagrant misuse of RSpec's syntax, like [`it "uniqueness of title"`](https://github.com/betterspecs/betterspecs/issues/2#issuecomment-9171781)). If you were asked about an array's behavior, would you say "Array behaves like a collection initialized with 3 items says it has three items" out loud? The indentation is what implies any complete sentence structure: "Array behaves like a collection, *so when it's* initialized with 3 items, *it* says it has three items".

The output's structure, of course, comes from the nested `context` blocks + implied `it`s when you read the code. But I know that when I'm looking for a failed example, I'm rarely able to trace the convoluted sentence back through some deep nesting of contexts. I'm just going to rely on the line numbers reported at the end of the run. For that matter, I'm just going to be using the [progress bar format](https://relishapp.com/rspec/rspec-core/v/3-9/docs/command-line/format-option#progress-bar-format-%28default%29) anyway. 🤷

I hear rumors of these [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) teams where specs are shared between technical & nontechnical folk alike, forming an executable set of human-readable requirements that everyone can agree on. It sounds nice in theory. I've never worked on such a team, so I'm speaking out of my depth. But I can't help noticing that RSpec is still used wholesale for everything from acceptance tests down to unit tests. It's even considered *good style* to [keep example descriptions short](https://rspec.rubystyle.guide/#keep-example-descriptions-short) and [use class/method names](https://rspec.rubystyle.guide/#describe-the-methods). Short descriptions are hardly conducive to thorough documentation. And should your nontechnical team *really* be caring about the code on the classes-and-methods level? I highly doubt it.

If you're going to do something, do it right. At least with [Cucumber](https://cucumber.io/) you could write honest-to-god executable documentation in complete sentences and paragraphs. Even [RSpec's docs](https://relishapp.com/rspec/) are generated this way. Again, I've never actually been on a team that *does* this; but RSpec strikes me as a half-measure by comparison.

## The Magic

So if programmers are the only ones who truly care about the tests at the level they're being written with RSpec, why not use something simpler? Well, maybe RSpec is just that gosh dang *powerful*. I mean, the library certainly does a whole lot of stuff.

But then there's [Minitest](https://github.com/seattlerb/minitest), the standard foil to RSpec. It shows how the same testing can still be done in plain old Ruby. RSpec's general syntax can readily be transliterated to classes & methods. That way, you don't have to learn a new language to write your tests.

```ruby
class ArticleTest < Minitest::Test
  def article
    @article ||= Article.new(title: title)
  end

  def title
    @title ||= Fake::Lorem.sentence
  end

  class Header < self
    def header
      @header ||= article.header
    end

    def words_in(sentence)
      sentence.gsub(/[[:punct:]]/, '').split
    end

    class WhenTheTitleIsOver50CharacterLong < self
      def title
        @title ||= Faker::Lorem.paragraph_by_chars(number: 100)
      end

      def test_it_gets_truncated
        # ...
      end

      def test_it_keeps_words_intact
        # ...
      end
    end
  end
end
```

Granted, the nesting is so awkward that you usually won't do this in Minitest. A more faithful translation would probably be flatter, with simpler method names. (This is an advantage, if you ask me.) Then, instead of using the expectations syntax, `Minitest::Test` defines [assertion methods](http://docs.seattlerb.org/minitest/Minitest/Assertions.html) that your class will inherit.

```ruby
class ArticleTest < Minitest::Test
  def words_in(sentence)
    sentence.gsub(/[[:punct:]]/, '').split
  end

  def test_long_headers_get_truncated
    article = Article.new(title: Faker::Lorem.paragraph_by_chars(number: 100))
    assert_operator article.header.length, :<=, 50
    assert article.header.end_with?('...')
  end

  def test_header_keeps_words_intact
    title = Faker::Lorem.paragraph_by_chars(number: 100)
    words = words_in(title)
    article = Article.new(title: title)
    words_in(article.header).each do |word|
      assert_includes words, word, <<~MSG
        #{word.inspect} is in the header #{article.header.inspect}
        but not in the title #{article.title.inspect}
      MSG
    end
  end
end
```

The "first class features" of matchers are generally "first class features" of Minitest&mdash;or indeed, the Ruby programming language.

* [Negation](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/define-negated-matcher) is done with the `refute_*` counterparts to `assert_*`.
* [Customized messages](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/customized-message) can be given as the last argument to any Minitest assertion (except [`assert_output`](http://docs.seattlerb.org/minitest/Minitest/Assertions.html#method-i-assert_output)), as demonstrated above.
* [Custom matchers](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/custom-matchers) simply become Ruby methods that delegate to the Minitest `assert_*` primitives.
* [Compound expectations](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/compound-expectations) could just as well be separate assertions for conjunction (`assert one; assert two`) or plain boolean operators for disjunction (`assert one || two, 'neither one nor two'`).

Similarly, RSpec hooks that Minitest [excludes](https://github.com/seattlerb/minitest/issues/61) can be accomplished with [plain Ruby](https://github.com/seattlerb/minitest/#label-How+to+run+code+before+a+group+of+tests-3F). Shared contexts & example groups can be arranged as [modules](https://github.com/seattlerb/minitest/#label-How+to+share+code+across+test+classes-3F) and [classes](http://wojtekmach.pl/blog/2012/07/17/liskov-principle-and-minitest/). [RSpec mocks](https://relishapp.com/rspec/rspec-mocks/docs) are more or less comparable to Minitest's [equivalents](https://github.com/seattlerb/minitest#label-Mocks). By and large, Minitest doesn't reinvent Ruby's wheel.

Furthermore, almost just to prove that it's possible, Minitest implements essentially a [subset of RSpec's DSL](https://github.com/seattlerb/minitest#label-Specs). But no matter how small its implementation (a dubious virtue), I have the same distaste for it as I do for RSpec. Why bother in the first place?

The one thing I'll give RSpec is that strings are easier to type out than coming up with `test_*` method names. But you don't need a whole framework to achieve this. [`ActiveSupport::TestCase`](https://guides.rubyonrails.org/testing.html#rails-meets-minitest) has a [lightweight `test` method](https://github.com/rails/rails/blob/0e35c10659b4dba65d580b76df267be1eef0c5dd/activesupport/lib/active_support/testing/declarative.rb#L13-L24) that's almost literally `def` with a string.

```ruby
class ArticleTest < ActiveSupport::TestCase
  def words_in(sentence)
    # ...
  end

  test 'long headers get truncated' do
    # ...
  end

  test 'header keeps words intact' do
    # ...
  end
end

ArticleTest.instance_methods - ActiveSupport::TestCase.instance_methods #=> [
#   :test_long_headers_get_truncated,
#   :test_header_keeps_words_intact,
#   :words_in,
# ]
```

The times I *do* need to group together tests, I admittedly miss having `context`. For that, you could use something as beautifully small as [minitest-context](https://github.com/arenaflowers/minitest-context) or as relatively big as [shoulda-context](https://github.com/thoughtbot/shoulda-context).

Going further, Minitest plugins fill in various gaps. [minitest-around](https://github.com/splattael/minitest-around) gives you [`around` hooks](https://relishapp.com/rspec/rspec-core/v/3-9/docs/hooks/around-hooks). [minitest-metadata](https://github.com/wojtekmach/minitest-metadata) gives you [metadata](https://relishapp.com/rspec/rspec-core/docs/metadata/user-defined-metadata) (which I've personally never actually needed). [minitest-reporters](https://github.com/kern/minitest-reporters) gives you nicer output akin to [RSpec's formatters](https://relishapp.com/rspec/rspec-core/v/3-9/docs/command-line/format-option). And so on.

I agree that all this patchwork isn't a great experience out of the box. Even basic CLI tooling is hand-rolled. So you aren't really comparing RSpec to *just* Minitest; you're comparing it to Minitest and an ecosystem of plugins & boilerplate. Be that as it may, RSpec's killer features are met by this ecosystem&mdash;plus a dose of basic Ruby programming skills. Perhaps it's less cohesive, but I don't think you're throwing the baby out with the bathwater by ditching RSpec.

## Magic Done Wrong

Complex problems often demand complex solutions. But RSpec is the sort of complexity that *increases* cognitive load. If you're going to lean into the so-called magic, why would you use it to make your job harder?

I believe it's possible to use magic for good. To do something complex in order to make something simple.

I don't even think Minitest accomplishes this. If you were to ask around, it'd certainly be perceived as less "magic", and it *is* less complicated than a spec-style DSL. But you still carry a lot of mental weight:

* `assert_same` uses `equal?`, but `assert_equal` uses `==`, and `assert_nil` has to be used explicitly [come Minitest 6](https://github.com/seattlerb/minitest/commit/922bc9151a622cb3ef0b9f170aa09c3bb72c7eb8).
* Remember to use the order `assert_equal expect, actual` for nicer failure messages.
* And it's `assert_includes collection, object`, not `assert_includes object, collection`.
* Oh yeah, `assert_predicate` exists. Maybe I should use that for better failure messages instead of `assert object.predicate?`.
* Does anyone else always have to think about what `assert_in_epsilon` means, or is it just me?
* I've made the mistake before of thinking `assert_raises "message"` does an `assert_equal "message", e.message`, but it's just the Minitest failure message. Oops. 😅
* Stubbing is (purposefully) painful. I definitely avoid it&mdash;I've been burned enough times by bad stubs. But sometimes I legitimately need it, and making the process more restrictive doesn't help.

Much like I have to open up RSpec's docs to figure out how to use it, I regularly crack open Minitest's source code to remember how a certain method works (usually the stubbing). Minitest is a breath of fresh air compared to RSpec, but I have to admit the mental drag is still there years after adopting it.

## Magic Done Right

So what *does* qualify as good magic?

It wasn't until I used [pytest](https://docs.pytest.org/) in a Python project that it clicked. Specifically, all of its [assertions](https://docs.pytest.org/en/latest/assert.html) are written plainly as Python [`assert` statements](https://docs.python.org/3/reference/simple_stmts.html#assert). (Although its [fixtures](https://docs.pytest.org/en/latest/fixture.html) are pretty fuckin' cool, too.)

We could do the same in Minitest, saying `assert a == b` instead of `assert_equal a, b`. But the whole problem is that we won't get useful output when something fails. The plain `assert` is being passed in a boolean, so its failure would look like

```
Minitest::Assertion: Expected false to be truthy.
```

whereas the `assert_equal` could say

```
Minitest::Assertion: Expected: 1
  Actual: 2
```

So what pytest does is [rewrite the whole damn program](http://pybites.blogspot.com/2011/07/behind-scenes-of-pytests-new-assertion.html) behind the scenes to decorate every `assert` with rich diffs. You, the tester, needn't be any the wiser. If your test fails, you just get a useful error message. You don't have to remember the difference between `assert_same` and `assert_equal`, or the order of arguments to `assert_includes`. Its complexity is entirely hidden from you, provides concrete value, and it doesn't ask anything special in return. How's *that* for magic?

The closest Ruby version I've seen is [Power Assert](https://github.com/ruby/power_assert), which combs over a Ruby expression so you get thorough output like

```
Failure:
   assert { 3.times.to_a.include?(3) }
              |     |    |
              |     |    false
              |     [0, 1, 2]
              #<Enumerator: 3:times>
```

I haven't had a chance to convert my existing Minitest suites over to see how well this works in practice. The project comes with some strong [caveats](https://github.com/ruby/power_assert#label-Known+Limitations) that make me hesitant to try, though.

Programming language & library designers should really be taking heed. All these [RSpec vs Minitest](https://www.google.com/search?q=RSpec%20vs%20Minitest) debates are tired&mdash;they've [been raging](http://www.rubyinside.com/dhh-offended-by-rspec-debate-4610.html) *years* longer than I've even been programming Ruby. Power Assert & pytest have to go through great lengths to get something resembling good output without burdening the programmer. A good language design can make this much easier.

One example is [Elixir](https://elixir-lang.org/). The [ExUnit](https://hexdocs.pm/ex_unit/ExUnit.html) framework is simple but powerful, and its implementation is relatively straightforward due to Elixir's [macros](https://elixir-lang.org/getting-started/meta/macros.html). [`ExUnit.Case`](https://hexdocs.pm/ex_unit/ExUnit.Case.html) gives you the `describe`/`test` pair&mdash;a light dusting of DSL, like `ActiveSupport::TestCase` but with more functionality. Then [`ExUnit.Assertions`](https://hexdocs.pm/ex_unit/ExUnit.Assertions.html) defines the `assert` macro, which introspects its arguments at compile-time so you get better failure reporting. The fact that it can even do this is a testament to the language's design.

It's still arguably complex. After all, someone had to implement everything that goes into supporting macros. Maybe programming is just [magic all the way down](https://en.wikipedia.org/wiki/Turtles_all_the_way_down). I only ask that *your* magic doesn't shift the burden onto *me*.
