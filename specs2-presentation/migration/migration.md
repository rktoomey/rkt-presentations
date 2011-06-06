!SLIDE

# Migrating to specs2

<span class="eric">
<b>Eric:</b> The best way to introduce concurrency into examples is to isolate the mutable variables.
</span>
<br/>
<br/>
<span class="new">NEW</span> Eric posted about [specs2 migration](http://etorreborre.blogspot.com/2011/05/specs2-migration-guide.html) on his blog.

- unit specs - the path of least resistance
- acceptance specs - requires complete restructuring, but in exchange for substantial benefits
<br/>

!SLIDE

# Migrating to specs2

## Use case: migrating my own project

On 9 March, I migrated [Salat](https://github.com/novus/salat/) from specs 1.6.7 to specs2 1.0.1.  It took
about two hours, and it was easy.
<br/>
<br/>
My strategy was as follows:

- update the specs2 dependencies
- switch my base testing trait, ``SalatSpec`` from using ``org.specs.Specification`` to using ``org.specs2.mutable.Specification``
- address minor syntax changes to the matcher syntax
- handle cases where I needed to do something before and/or after a unit test

!SLIDE

# Updating the dependencies

Drop in place and go:

    val specs2 = "org.specs2" %% "specs2" % "1.3"
    val scalaz = "org.specs2" %% "specs2-scalaz-core" % "6.0.RC2"

    def specs2Framework = new TestFramework("org.specs2.runner.SpecsFramework")
    override def testFrameworks = super.testFrameworks ++ Seq(specs2Framework)

    val snapshots = "snapshots" at "http://scala-tools.org/repo-snapshots"
    val releases  = "releases" at "http://scala-tools.org/repo-releases"

!SLIDE

# Migrating to specs2: restructuring my test trait

## Before

    trait SalatSpec extends Specification with PendingUntilFixed with Logging {
      val SalatSpecDb = "test_salat"
      detailedDiffs()
      doBeforeSpec {
        com.mongodb.casbah.commons.conversions.scala.RegisterConversionHelpers()
        com.mongodb.casbah.commons.conversions.scala.RegisterJodaTimeConversionHelpers()
      }
      doAfterSpec {
        MongoConnection().dropDatabase(SalatSpecDb)
      }

    }

!SLIDE

# Migrating to specs2: restructuring my test trait

## After

    trait SalatSpec extends Specification with Logging {
      val SalatSpecDb = "test_salat"
      override def is =
        Step {
          com.mongodb.casbah.commons.conversions.scala.RegisterConversionHelpers()
          com.mongodb.casbah.commons.conversions.scala.RegisterJodaTimeConversionHelpers()
        } ^
          super.is ^
          Step {
            MongoConnection().dropDatabase(SalatSpecDb)
          }

    }

- No more ``detailedDiffs()`` - the default settings were good enough:
- No more ``PendingUntilFixed`` - in specs2 this is now part of the common specification features
- ``doBeforeSpec`` has been replaced by overriding ``is`` with a ``Step`` to register Casbah's conversion helpers
- ``doAfterSpec`` has been replaced by using ``^`` to glue a final step onto the supertrait's ``is`` method

!SLIDE

# Migrating to specs2: changes to matchers

The following matcher forms are now preferred:

    a must matcher(b)
    a must not matcher(c)

## Before

    "My test string" must notContain("bingo")
    dbo must notHaveKey("aa")

## After

    "My test string" must not contain("bingo")
    dbo must not have key("aa")

!SLIDE

# Migrating to specs2: making things run sequentially

For the most part, the specs in Salat can run concurrently.

However, some examples for ``SalatDAO`` required sequential access to shared mutable state in a single
MongoDB collection.

    class SalatDAOSpec extends SalatSpec {

      // which most specs can execute concurrently, this particular spec needs to execute sequentially
      // to avoid mutating shared state: namely, the MongoDB collection referenced by the AlphaDAO

      override def is = args(sequential = true) ^ super.is


!SLIDE

# Migrating to specs2: using scopes to set up data

Unit specs have ``Scope``, a simple way of creating a new scope with variables that can be re-used in any example.
<br/>
<br/>
I used it to isolate the tedium of data setup so that my examples could focus on what I was really trying to achieve.

    trait xiScope extends Scope {
      log.debug("before: dropping %s", XiDAO.collection.getFullName())
      XiDAO.collection.drop()
      XiDAO.collection.count must_== 0L

      val xi1 = Xi(x = "x1", y = Some("y1"))
      val xi2 = Xi(x = "x2", y = Some("y2"))
      val xi3 = Xi(x = "x3", y = Some("y3"))
      val xi4 = Xi(x = "x4", y = Some("y4"))
      val xi5 = Xi(x = "x5", y = None)
      val _ids = XiDAO.insert(xi1, xi2, xi3, xi4, xi5)
      _ids must contain(Option(xi1.id), Option(xi2.id), Option(xi3.id), Option(xi4.id), Option(xi5.id))
      XiDAO.collection.count must_== 5L
    }

!SLIDE

# Migrating to specs2: using a scope

My new ``xiScope`` can be used as easily as:

    "support using a projection on an Option field to filter out Nones" in new xiScope {
      // a projection on a findOne that matches xi1
      XiDAO.primitiveProjection[String](MongoDBObject("x" -> "x1"), "y") must beSome("y1")
      // a projection on a findOne that brings nothing back
      XiDAO.primitiveProjection[String](MongoDBObject("x" -> "x99"), "y") must beNone

      val projList = XiDAO.primitiveProjections[String](MongoDBObject(), "y")
      projList must haveSize(4)
      projList must contain("y1", "y2", "y3", "y4") // xi5 has a null value for y, not in the list
    }

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


<img class="logo" src="/img/novus-logo.gif" />