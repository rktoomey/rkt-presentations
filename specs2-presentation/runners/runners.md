!SLIDE

# Control execution and reporting

Use [arguments](http://etorreborre.github.com/specs2/guide/org.specs2.guide.Runners.html#Arguments).    It's that easy.

Inside a spec, pass them in to ``is``.  For instance, let's say you are working inside a web framework and
you want to filter stacktraces to show only your own code.

    def is = args(traceFilter = includeTrace("com.foo.confabulator"))

In sbt, you can pass in arguments: the example shown below will output to both console and html.

    > test-only com.foo.confabulator.test.TryHarderSpec -- html console

!SLIDE

# JUnit Integratation

    class WithJUnitSpec extends SpecificationWithJUnit {
      "My spec" should {
        "run in JUnit too" in {
          success
        }
      }
    }

For IDE support, you can still use ``@Runner``:

    import org.junit.runner._
    import runner._

    @RunWith(classOf[JUnitRunner])
    class WithJUnitSpec extends Specification {
      "My spec" should {
        "run in JUnit too" in {
          success
        }
      }
    }

!SLIDE

# Contexts: making things happen when you need them

When you want to make sure that something happens for every examples, use any combination of these traits:

- ``Before``
- ``After``
- ``Around``
- ``Outside``

Contexts provide an ``apply`` method which can be applied to the body of an example so that your code
is executed when you need it relative to the example code.

Contexts of the same type can be composed and/or sequenced.

Whereas ``Scope`` brings what's inside into context with a mutable spec, a Context is about having
the appropriate method being executed when and where you need it (http connection, database operations).

!SLIDE

# Demonstration of Around (unit spec)

Courtesy of a [gist](https://gist.github.com/904913) from Eric Torreborre.  ``>>`` appends an
``Example`` to the unit spec's list of fragments.

    class UnitSpec extends org.specs2.mutable.Specification {

      "This specification has examples which must be executed inside an http session" >> {
        "Example 1 is executed inside the session" >> http {
          success
        }
        "Example 2 is also executed inside the session" >> http {
          success
        }
      }

      object http extends Around {
        def around[T <% Result](t: =>T) = openHttpSession("test") {
          t // execute t inside a http session
        }
      }
    }

Eric also has a [gist](https://gist.github.com/905789) sketching out how to use ``Outside`` and ``AroundOutside``.

!SLIDE

# This old spec: specs 1.x

There's always something special in a dusty corner...  Now it sees the light of day.


    class BadIdea extends Specification {
      var badIdea = MSet.empty[Int]
      "My badly set up spec" should {
        shareVariables()  // now we're in for it...
        doFirst {
          println("setUp(): hey, who got rid of JUnit?")
          badIdea = MHashSet.empty[Int] ++= (1 to 100).toSet
          badIdea must notBeEmpty
        }
        "make use of shared mutable state" in {
          badIdea ++= (101 to 200).toSet
          badIdea must have size (200)
        }
        "have expectations based on previous sequential manipulation of shared mutable state" in {
          badIdea = badIdea.filter(_ % 2 == 0)
          badIdea must have size (100)
        }
        // MOAR...
        doLast {
          println("tearDown(): success!")
          badIdea.clear()
        }
      }
    }

!SLIDE

# This old spec: hauled into specs2 with complete violation of intent

    import scala.collection.mutable.{Set => MSet, HashSet => MHashSet}

    class BadIdea extends org.specs2.mutable.Specification {
      var badIdea = MSet.empty[Int]

      override def is = args(sequential=true)^  Step {
          println("Doing something beforehand!")
          badIdea = MHashSet.empty[Int] ++= (1 to 100).toSet
          badIdea must not be empty
        } ^
          super.is ^ Step {
              println("Doing something afterwards!")
              badIdea.clear()
            }

      "My badly set up spec" should {
        "make use of shared mutable state" in {
          badIdea ++= (101 to 200).toSet
          badIdea must have size(200)
        }
        "have expectations based on previous sequential manipulation of shared mutable state" in {
          badIdea = badIdea.filter(_ % 2 == 0)
          badIdea must have size(100)
        }
        // Send help...
      }
    }

!SLIDE

# This old spec: renovated

The simplest way to make something happen beforehand: use ``Scope`` to leverage implicits
that convert

Stupidity contained, probably even compatible with specs3.  Yes!

    class BetterIdea extends Specification {

      // happens BEFORE each use case
      trait testData extends Scope {
        println("Setting up data JUST for you and your little dog, my pretty.")
        val betterIdea = (1 to 100).toSet
        betterIdea must have size(100)
      }

      "My better spec using contexts" should {
        "force each use case to have its own immutable data" in new testData {
          val newSet = betterIdea ++ (101 to 200).toSet
          newSet must have size(200)
        }
        "get rid of expectations that depend on shared state" in new testData {
          val anotherNewSet = betterIdea.filter(_ % 2 == 0)
          anotherNewSet must have size(50)  // changed expectation no longer depends on shared state!
        }
      }
    }

!SLIDE

# This old spec: renovated, but with a context this time

Now, imagine instead of a set, we were using shared mutable state like a database table.  Since I didn't
want to introduce deps into my test code, pretend our ``Set[Int]`` is actually a database table.  Now
using a ``Context`` instead of a ``Spec`` makes sense.

    object setupData extends Before {
      var betterIdea = Set.empty[Int]
      def before {
        betterIdea ++= (1 to 100).toSet
        betterIdea must have size(100)
      }
    }

    "or do it with a context instead" >> {
      "now my example will be executed inside the setupData context" >> setupData {
        val newSet = setupData.betterIdea ++ (101 to 200).toSet
        newSet must have size (200)
      }
    }


<img class="logo" src="/img/novus-logo.gif" />