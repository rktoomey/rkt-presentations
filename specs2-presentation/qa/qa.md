!SLIDE

<br/>
<br/>
<br/>
<br/>
<br/>
<span class="big_new">User Q & A</span>

!SLIDE

## Question

Can specs2 be used with ScalaTest?  and for what purpose?

## Answer

<span class="eric">
<b>Eric:</b>
<br/>
<br/>
The specs2 matchers should be usable in ScalaTest (and reciprocally).  You could use specs2 Json matchers for example
while keeping the ScalaTest reporters and infrastructure.
<br/>
<br/>
Possibly the ScalaCheck API or the DataTables as well but I haven't tried it.
<br/>
<br/>
Those objects traits can also be reused in JUnit and TestNG modulo some adapation of the ThrownExpectations trait.
See for example what is done here:
<a href="https://github.com/etorreborre/specs2/blob/1.4/src/main/scala/org/specs2/matcher/JUnitMatchers.scala">JUnitMatchers.scala</a>
<span>

!SLIDE

## Question

Can use cases written by a business user be parsed into a sample acceptance spec with default methods e1, e2, etc.
populated?

## Answer

<span class="eric">
<b>Eric:</b>
<br/>
<br/>
Possibly, yes. However my view on the subject is that business users can not conveniently do this. Note that the big
difficulty is not so much in writing the first spec but more in maintaining the whole lot.
<br/>
<br/>
The long-term answer is that I would like to write a GUI client to support this case: with version control integration,
iterative development,...
<br/>
<br/>
Actually ThoughtWorks is selling a tool that makes most of this:
<br/>
<a href="http://www.thoughtworks-studios.com/agile-test-automation">http://www.thoughtworks-studios.com/agile-test-automation</a>
</span>

!SLIDE

## Question

Did `Given` - `When` - `Then` come over from Ruby/Cuke?

## Answer

<span class="eric">
<b>Eric:</b>
<br/>
<br/>
Absolutely. My take on it is:
<br/>
* you don't have to use the given-when-then keywords but you can use whatever text you feel is natural
<br/>
* the G-W-T sequence is statically typed: well as much as possible, it is still possible to fail when extracting
  values from the text
</span>

!SLIDE

## Question

What else can `Given` - `When` - `Then` be used for?

## Answer

<span class="eric">
<b>Eric:</b>
<br/>
<br/>
I just see it as a guideline to write clear specifications along the "Arrange-act-assert" paradigm:
<br/>
<a href="http://c2.com/cgi/wiki?ArrangeActAssert">http://c2.com/cgi/wiki?ArrangeActAssert</a>
<span>

!SLIDE

## Question

Is it possible to define the body inline when using `"description" ! body` format?

## Answer

<span class="eric"><b>Eric:</b> Yes.<span>

    class InlineFunctionalDemoSpec extends Specification { def is =

      "My functional spec demo should"                          ^
        "show that only the last Result is returned"            ! {
          1 must beGreaterThan(9999)  // this MatchResult is discarded
          1 must beLessThanOrEqualTo(1) } ^
        "fail as expected when results are chained together"    ! { 1 must beGreaterThan(9999) and beLessThanOrEqualTo(1) }

    }

!SLIDE

## Question

What is AutoExample for?

## Answer

<span class="eric">
<b>Eric:</b>
<br/>
<br/>
To avoid repetition between the example description and the example code when the code says it all. For example I used
that a lot to specify matchers:
<br/>
<br/>
<a href="https://github.com/etorreborre/specs2/blob/1.3/src/test/scala/org/specs2/matcher/AnyMatchersSpec.scala">
https://github.com/etorreborre/specs2/blob/1.3/src/test/scala/org/specs2/matcher/AnyMatchersSpec.scala</a>
<br/>
<br/>
It's also used to enable the "backtick" notation:
<a href="http://etorreborre.github.com/specs2/guide/org.specs2.guide.SpecStructure.html#Acceptance+specification">http://etorreborre.github.com/specs2/guide/org.specs2.guide.SpecStructure.html#Acceptance+specification</a>
(see "you can even push this idea further by writing:").
<span>

!SLIDE

## Question

What's the rationale behind acceptance specs?  Isn't the way unit specs mingle text and code a benefit?

## Answer

<span class="eric">
<b>Eric:</b>
<br/>
<br/>
My own rationale is the following. By being able to read the whole spec *text* with one glance I can better think about
what I expect from my system. So when I implement something new, I usually spend some time adding the examples which
make sense before implementing them. I also found my specs easier to understand when revisiting them after a few weeks.
<span>

!SLIDE

## Discussion

What is the actual process of putting together an acceptance spec?

## Writing process

You might begin by just stubbing out expectations with inline results:

    class AcceptanceProcessDraft extends Specification { def is =
      "first example" ! pending ^
      "second example" ! pending
    }

Once the examples are clear, extract the results out into methods and begin populating them:

    class AcceptanceProcessSecondDraft extends Specification { def is =
      "first example"    ! e1 ^
      "second example"   ! e2 ^
      "third example"    ! e3

      def e1 = pending
      def e2 = pending
      def e3 = pending
    }

!SLIDE

## Discussion

What is the actual process of putting together an acceptance spec?

## Notes from Eric

There have been numerous discussion of acceptance spec style and purpose on the specs2 mailing list.
<br/>
<br/>
<span class="eric">
<b>Eric:</b>
<br/>
<br/>
One thing I noticed is that I often split examples in two:
<br/>
<code>
  "this should do that" ! e1^
</code>
<br/>
becomes
<pre><code>
  "this should do that"  ^
     "in this case"         ! e1^
     "in that case"         ! e2^
</code></pre>
<span>

!SLIDE

## Discussion

Compare and contrast specs/specs2 with ScalaTest.

## My personal experience

Speaking personally, I started out with ScalaTest at work but quickly found specs to be a better tool for me.
<br/>
<br/>
Although ScalaTest appeared more direct to me at first, coming from a JUnit/TestNG background, over a period of two months
I became dissatisfied:

- the errors when tests failed were difficult to work with.  When I was just starting out in Scala, trying to figure out where
in an N-deep level nest of anonymous inner classes some expectation failed was very daunting to me.
- ScalaTest dynamic matchers seemed initially appealing when I wanted to check values within an object graph but turned
into a maintenance nightmare when I refactored model objects.
<br/>
<br/>
I saw that some open source projects I liked were using specs, so I gave it a try.  I appreciated the clean syntax of specs,
and I liked the matcher syntax more than ScalaTest's.
<br/>
<br/>
In simple testing setups ScalaTest and specs appear quite similar, so my transition from ScalaTest to specs was quite rapid.
When my test cases failed, the errors made it extremely easy to target the failing lines.

!SLIDE

## Discussion

Compare and contrast specs/specs2 with ScalaTest.

## My personal experience, continued

Although I came to specs for its simplicity, in the long run what kept me using specs was its power.  So it was a natural
for me to migrate Salat to specs2 when it was released.  specs2, although a complete rewrite, had everything I liked about
specs 1.x plus many new powerful features I could use to isolate mutability in my specs.
<br/>
<br/>
That said, Akka uses ScalaTest, and I quite admire the style and thoroughness of their tests as a model for anyone who
wants to use ScalaTest.
<br/>
<br/>
There have been numerous discussions online.  Here's a good StackOverflow thread with a comparison from Bill Venners:

- [What's the difference between ScalaTest and Scala Specs unit test frameworks?](http://stackoverflow.com/questions/2220815/whats-the-difference-between-scalatest-and-scala-specs-unit-test-frameworks)