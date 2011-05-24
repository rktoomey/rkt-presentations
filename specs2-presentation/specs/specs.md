!SLIDE

<p style="font-size: 70px; color: #16325A;">specs2</p>
<br/>
<br/>
<br/>
<br/>
<p style="font-size: 45px; color: #16325A; font-style: italic;">What's new in the Scala BDD world?</p>
<br/>
<br/>
[http://specs2.org](http://specs2.org)

!SLIDE

# specs2: State of the art executable software specifications

``TODO: still in flux``

- The evolution of specs2
- Migrating to specs2
    - Dependencies
    - Mutable unit specifications: the path of least resistence
    - Acceptance specifications: full-on assimilation
- Cool new features of specs2
    - JSON matchers
    - Scalacheck
    - Generate your user guide from acceptance specs
- How to guides
    - Using contexts
    - Writing your own matchers
    - Running in sequence when sharing mutable state
- Online Resources

!SLIDE

# The evolution of specs2

specs is a DSL in Scala for doing BDD (Behaviour-Driven Development).
<br/>
<br/>
specs2 is a <b><i>complete rewrite</i></b> of specs 1.x.
<br/>
<br/>
The design objectives of specs 1.x were:

- conciseness
- readability
- extensibility
- configuration
- clear implementation
- easy to fix and evolve


!SLIDE

# Recap of specs 1.x

## Conciseness

specs 1.x used the power of Scala implicits to provide an elegant syntax, but:

- implicit methods were inherited from ``Specification``, causing unexpected implicit conversions and namespace pollution
- reserved words like ``should``, ``can`` and ``in`` were not always convenient, and overriding them required ad hoc methods
- ``given ... when ... then`` support required additional methods with no added value but displaying these words


## Readability

- large amounts of code interleaved into small amounts of text devolved into spaghetti
- the [literate specs](http://code.google.com/p/specs/wiki/LiterateSpecifications#A_short_example) prototype was difficult to integrate with the rest of specs 1.x


!SLIDE

# Recap of specs 1.x

## Extensibility

Hindsight: it's very difficult to make something extensible without knowing what the requirements would be.

- some of the matchers and runners written to enhance specs 1.x were worth integrating into specs2
<br/>
<br/>

## Configuration

Customising specs 1.x required a combination of property files, reflection and system properties

- end result: user confusion, frustration and ugly hacks to work around "sensible defaults"

!SLIDE

# Recap of specs 1.x

## Clear implementation

In specs 1.x, the examples executed "on demand" when a runner asked the specification to report its successes and failures

- the examples had to run to know what the outcome was and report it to the specification
- ...but the examples ran without knowing their full execution context (what runs before and after)

!SLIDE

# Recap of specs 1.x

Worse, specs did not <i>actually</i> execute in full isolation...

- a feature intended to be helpful, [the automatic rest of local variables](http://code.google.com/p/specs/wiki/DeclareSpecifications#Execution_model) was an implementation nightmare
- what appeared to be isolation where running one example did not affect surrounding variables was achieved by cloning and carefully copying state around

!SLIDE

# Recap of specs 1.x

## Easy to fix and evolve

The design of specs 1.x with lots of variables and side-effects made being responsive to users harder than
it had to be:

- dianosing bugs and fixing issues takes longer
- adding enhancements became complicated, in some cases nearly impossible

!SLIDE

# A New Deal

specs2 preserves the elegant DSL of specs 1.x but with a simpler, more robust implementation.

## The design principles of specs2
<br/>

<ol>
  <li>Do not use mutable variables</li>
  <li>Use a simple structure</li>
  <li>Control the dependencies (no cycles)</li>
  <li>Control the scope of implicits</li>
</ol>

!SLIDE

# Field Guide to specs2

## Unit specifications

- Extend the ``org.specs2.mutable.Specification`` trait
- are mutable
- Use ``should`` / ``in`` format
<br/>
<br/>

## Acceptance specs

- Extend the ``org.specs2.Specification`` trait
- are functional when extending the default ``org.specs2.Specification`` trait
- Must define a method called ``is`` that takes a ``Fragments`` object, which is composed of:
    - an optional ``SpecStart``
    - a list of ``Fragment`` objects
    - an options ``SpecEnd``
<br/>
<br/>

## What do they both do?

Create a list of _specification fragments_.  But very differently!

!SLIDE

# What is a specification fragment?

- Simple text
    - to describe the test case
- An example
    - a description and executable code that returns a ``Result``, such as
        - a standard result (success, failure)
        - a matcher result
        - a boolean value
- formatting fragments
    -  such as line breaks and tabs to make the test output pleasing to the eye

!SLIDE

# Creating a unit specification

    package prasinous.unit

    import org.specs2.mutable._
    import prasinous._

    class TrivialUnitSpec extends Specification {

      "String reverser" should {
        "reverse String" in {
          StringReverser("hello") must_== "olleh"
          StringReverser("") must beEmpty
          StringReverser(null) must beNull
        }
      }

      "Option string reverser" should  {
        "reverse Option[String]" in {
          OptionStringReverser(Some("hello")) must beSome("olleh")
          // it's in the spec, that means i expected it, right?
          OptionStringReverser(Some(null)) must beSome(null)
          OptionStringReverser(None) must beNone
        }
      }

    }

!SLIDE

# Running a unit specification

Fire up sbt and run

    > test-only prasinous.unit.TrivialUnitSpec

    [info] == prasinous.unit.TrivialUnitSpec ==
    [info] String reverser should
    [info] + reverse String
    [info]
    [info] Option string reverser should
    [info] + reverse Option[String]
    [info]
    [info]
    [info] Total for specification TrivialUnitSpec
    [info] Finished in 72 ms
    [info] 2 examples, 0 failure, 0 error
    [info]
    [info] == prasinous.unit.TrivialUnitSpec ==

!SLIDE

# Creating an acceptance specification

    package prasinous.acceptance

    import org.specs2._
    import prasinous._

    class TrivialAcceptanceSpec extends Specification { def is =

      "This is a specification to check reversing Strings"      ^
                                                                p^
      "StringReverser should"                                   ^
        "reverse a String"                                      ! e1 ^
        "leave an empty String unaffected"                      ! e2 ^
        "not fall down snivelling when someone feeds it null"   ! e3 ^
                                                                p^
      "OptionStringReverser should"                             ^
        "reverse Option[String]"                                ! e4 ^
        "reverse None"                                          ! e5 ^
        "Damn it, will someone please fix the universe?"        ! e6


      def e1 = StringReverser("hello") must_== "olleh"
      def e2 = StringReverser("") must beEmpty
      def e3 = StringReverser(null) must beNull
      def e4 = OptionStringReverser(Some("hello")) must beSome("olleh")
      def e5 = OptionStringReverser(None) must beNone
      def e6 = OptionStringReverser(Some(null)) must beSome(null)
    }

!SLIDE

# Acceptance specification syntax

``def is`` kicks it off -

    class TrivialAcceptanceSpec extends Specification { def is =

``^`` glues things together.  ``p`` is a ``FormattingFragment``.

     "This is a specification to check reversing Strings"      ^
                                                               p^
     "StringReverser should"                                   ^

``"description" ! body`` creates an ``Example`` where ``body`` is a method that returns a ``Result``.

    "reverse a String"                                         ! e1

where

    def e1 = StringReverser("hello") must_== "olleh"

returns ``MatchResult[Any]``

!SLIDE

## Formatting

The specs2 user guide contains a detailed [Layout](http://etorreborre.github.com/specs2/guide/org.specs2.guide.SpecStructure.html#Layout)
section explaining how acceptance specifications are formatted.

Q: Will anything aid you with keeping acceptance specification syntax neatly lined up?

A: Alas, not yet.

Keep an eye on the [mailing list](http://groups.google.com/group/specs2-users) and
[Scalariform](http://mdr.github.com/scalariform/) - someone will surely do something soon.

!SLIDE

# Running an acceptance spec

Fire up sbt and run

    > test-only prasinous.acceptance.TrivialAcceptanceSpec

    [info] == prasinous.acceptance.TrivialAcceptanceSpec ==
    [info] This is a specification to check reversing Strings
    [info]
    [info] StringReverser should
    [info] + reverse a String
    [info] + leave an empty String unaffected
    [info] + not fall down snivelling when someone feeds it null
    [info]
    [info] OptionStringReverser should
    [info] + reverse Option[String]
    [info] + reverse None
    [info] + Damn it, will someone please fix the universe?
    [info]
    [info] Total for specification TrivialAcceptanceSpec
    [info] Finished in 228 ms
    [info] 6 examples, 0 failure, 0 error
    [info]

!SLIDE

# Results

An instance of ``org.specs2.executable.Result`` contains:

- a message describing the outcome
- a message describing the expectation(s)

``StandardResults`` indicate the result of executing an example:

- ``success``
- ``failure``
- ``anError``
- ``pending``
- ``skipped``

A ``MatcherResult`` contains the outcome of an expectation, such as

    "hello" must contain("lo")    // success
    "hello" must contain("lz")    // failure

!SLIDE

# Failure

    > test-only prasinous.unit.FailedExpectationsSpec

    [info] == prasinous.unit.FailedExpectationsSpec ==
    [info] Bad expectations should
    [error] x fail loudly
    [error]     'olleh' is not equal to 'goodbye' (FailedExpectationsSpec.scala:10)
    [info]
    [info]
    [info] Total for specification FailedExpectationsSpec
    [info] Finished in 76 ms
    [info] 1 example, 1 failure, 0 error
    [info]
    [info] == prasinous.unit.FailedExpectationsSpec ==

!SLIDE

# Error

    > test-only prasinous.unit.ErrorDemoSpec

    [info]
    [info] == prasinous.unit.ErrorDemoSpec ==
    [info] My shoddy Fibonacci sequence implementation should
    [error] ! Fragment evaluation error
    [error]     ThrowableException: Bored now. (FutureTask.java:138)
    [error] prasinous.unit.ErrorDemoSpec.fibonacci(ErrorDemoSpec.scala:15)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10$$anonfun$apply$11.apply(ErrorDemoSpec.scala:11)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10$$anonfun$apply$11.apply(ErrorDemoSpec.scala:11)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10.apply(ErrorDemoSpec.scala:11)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10.apply(ErrorDemoSpec.scala:11)
    [error] Bored now.
    [error] prasinous.unit.ErrorDemoSpec.fibonacci(ErrorDemoSpec.scala:15)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10$$anonfun$apply$11.apply(ErrorDemoSpec.scala:11)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10$$anonfun$apply$11.apply(ErrorDemoSpec.scala:11)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10.apply(ErrorDemoSpec.scala:11)
    [error] prasinous.unit.ErrorDemoSpec$$anonfun$1$$anonfun$apply$10.apply(ErrorDemoSpec.scala:11)
    [info]
    [info]
    [info] Total for specification ErrorDemoSpec
    [info] Finished in 41 ms
    [info] 1 example, 0 failure, 1 error
    [info]
    [info] == prasinous.unit.ErrorDemoSpec ==

!SLIDE

# Acceptance specs are functional

    package prasinous.acceptance

    import org.specs2._

    class FunctionalDemoSpec extends Specification { def is =

      "My functional spec demo should"                          ^
        "show that only the last Result is returned"            ! e1 ^
        "fail as expected when results are chained together"    ! e2

      def e1 = {
        1 must beGreaterThan(9999)  // this MatchResult is discarded
        1 must beLessThanOrEqualTo(1)
      }

      // this fails as expected
      def e2 = 1 must beGreaterThan(9999) and beLessThanOrEqualTo(1)

    }

Yields:

    [info] == prasinous.acceptance.FunctionalDemoSpec ==
    [info] My functional spec demo should
    [info] + show that only the last Result is returned
    [error] x fail as expected when results are chained together
    [error]     1 is less than 9999 (FunctionalDemoSpec.scala:10)
    [info]


!SLIDE

# Mutable specs are not

    package prasinous.unit

    import org.specs2.mutable._

    class NotFunctionalDemoSpec extends Specification {
      "Mutable specs" should {
        "fail on any bad expectation" in {
          e1
        }
        "fail on chained bad expectations too" in {
          e2
        }
      }
      def e1 = {
        1 must beGreaterThan(9999) // this MatchResult is NOT discarded
        1 must beLessThanOrEqualTo(1)
      }
      def e2 = 1 must beGreaterThan(9999) and beLessThanOrEqualTo(1)
    }

Yields:

    [info] == prasinous.unit.NotFunctionalDemoSpec ==
    [info] Mutable specs should
    [error] x fail on any bad expectation
    [error]     1 is less than 9999 (NotFunctionalDemoSpec.scala:8)
    [error] x fail on chained bad expectations too
    [error]     1 is less than 9999 (NotFunctionalDemoSpec.scala:12)


!SLIDE

# Migrating to specs2: negating matchers

Matchers now use ``not [matcher]`` format.

Was:
    "My test string" must notContain("bingo")

Now:
    "My test string" must not contain("bingo")

<img class="logo" src="/img/novus-logo.gif" />