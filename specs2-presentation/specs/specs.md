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
    - Test case: migrating Salat
- Cool new features of specs2
    - JSON matchers
    - Scalacheck
- Specs2 in the wild
- Online Resources

!SLIDE

# The evolution of specs2

specs is a DSL in Scala for doing BDD (Behaviour-Driven Development).
<br/>
<br/>
specs2 is a <b><i>complete rewrite</i></b> of specs 1.x.

## The design principles of specs2
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
    - ``in`` creates an ``Example`` object containing a ``Result``
    - ``should`` creates a group of ``Example`` objects
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

Both types of specifications contain a list of specification fragments provided by the ``is`` method
in the ``SpecificationStructure`` trait.

!SLIDE

# What is a specification fragment?

- Simple text
    - to describe the test case
- Examples
    - a description and executable code that returns a ``Result``, such as
        - a standard result (success, failure)
        - a matcher result
        - a boolean value
- Steps and actions that return success or failure
    - reported only if an exception occurs
- ``SpecStart`` and ``SpecEnd`` delimiters in acceptance specs
- tagging fragments to define which fragments should be included or excluded
- formatting fragments
    -  such as line breaks and tabs to make the test output pleasing to the eye

!SLIDE

# Execution

specs2 executes examples _concurrently_ by default.  You have to explicitly specify when you need
sequential execution.

Fragments are sorted in groups so that all the elements of the group can be executed concurrently.

As each group of ``Example`` fragments runs concurrently, each ``Result`` is collected in a sequence of
 ``ExecutedFragments``, which are then reduced for reporting.

``Step`` can be used to break up the sequences in order to do some intitialisation or cleanup.

!SLIDE

# What is a result?

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
<br/>
<br/>

Q: Will anything aid you with keeping acceptance specification syntax neatly lined up?
<br/>
<br/>

A: Alas, not yet.
<br/>
<br/>


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

# Acceptance specs are functional by default

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

    class NotFunctionalDemoSpec extends org.specs2.mutable.Specification {
      "Mutable specs" should {
        "fail on any bad expectation" in {
          e1   // fails here
          e2   // this is bad too, but we never get here until we fix e1
        }
        "fail on chained bad expectations too" in {
          e3
        }
      }

      def e1 = {
        1 must beGreaterThan(9999) // this MatchResult is NOT discarded
        1 must beLessThanOrEqualTo(1)
      }
      def e2 = {
        1 must beGreaterThan(8888)  // this would fail but we never get here
      }
      def e3 = 1 must beGreaterThan(9999) and beLessThanOrEqualTo(1)
    }

Yields:

    [info] == prasinous.unit.NotFunctionalDemoSpec ==
    [info] Mutable specs should
    [error] x fail on any bad expectation
    [error]     1 is less than 9999 (NotFunctionalDemoSpec.scala:8)
    [error] x fail on chained bad expectations too
    [error]     1 is less than 9999 (NotFunctionalDemoSpec.scala:12)

!SLIDE

# Thrown expectations: a big difference

Acceptance specs matcher behaviour is to return an ``Expectable`` which handles applying the matcher
and returning a ``Result``.

Unit specs throw expectations as soon as they fail, .

If you want acceptance specs to throw expectations like unit specs, mix in ``ThrownExpectations``.

It doesn't change the functional behaviour of acceptance specs (i.e. only the returned ``Result``
will affect the final outcome), but it's useful for interfacing with other test frameworks.


<img class="logo" src="/img/novus-logo.gif" />