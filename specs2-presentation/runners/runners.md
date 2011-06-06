!SLIDE

# Contexts: making things happen when you need them

<span class="new">
The specs2 user guide has been updated to provide increased coverage on how to use
<a href="http://etorreborre.github.com/specs2/guide/org.specs2.guide.SpecStructure.html#Contexts">
Contexts</a> - go see that immediately.
<br/>
Also, see this helpful <a href="https://gist.github.com/997433">gist</a> showing how to use implicit contexts
to reduce duplication.
</span>

When you want to make sure that something happens for every example, use any combination of these traits:

- ``Before``
- ``After``
- ``Around``
- ``Outside``

Contexts provide an ``apply`` method which can be applied to the body of an example so that your code
is executed when you need it relative to the example code.
<br/>
<br/>
Contexts of the same type can be composed and/or sequenced.  <span class="clarafication">Contexts can also extend each
other to provide more specific setups.</span>
<br/>
<br/>
<span class="clarification"><code>Context</code> now extends <code>Scope</code>.</span>
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

The simplest way to equip your examples with the state they expect: use ``Scope`` to create a reusable
state sandbox and rebase the existing expectations to work with the new setup.

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

# This old spec: what about context instead of scope?

Now, imagine instead of a set, we were using shared mutable state like a database table.
<br/>
<br/>
Since I didn't want to introduce deps into my test code, pretend ``Before`` is actually populating
a database table with known test data.  Now using a ``Context`` instead of a ``Spec`` makes sense.

    object setupData extends Before {
      var betterIdea = Set.empty[Int]
      def before {
        betterIdea ++= (1 to 100).toSet
        betterIdea must have size(100)
      }
    }

    "or do it with a context instead" >> {
      "now my example will be executed inside the setupData context" >> setupData {
        // sub in database access for "setupData.betterIdea"
        val newSet = setupData.betterIdea ++ (101 to 200).toSet
        newSet must have size (200)
      }
    }


<img class="logo" src="/img/novus-logo.gif" />