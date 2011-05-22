!SLIDE

<p style="font-size: 60px; color: 16325a;">Salat in four minutes</p> 
<br/>
<br/>
<br/>
<br/>
<p style="font-size: 45px; color: 16325a; font-style: italic;">...or less</p>
<br/>
<br/>
[https://github.com/novus/salat](https://github.com/novus/salat)

!SLIDE

# Salat in four minutes... or less

- What is Salat?
    - Getting started
- Demonstration
- How does it work?
    - Moving parts
    - What Scala types can it handle?
- SalatDAO
- Advanced Usage
    - Trait or abstract class with @Salat
    - @Key
    - @Persist
    - Roll your own context
- More information


!SLIDE

# What is Salat?

Salat is a bi-directional Scala case class serialization library that leverages MongoDB's DBObject (which uses BSON underneath) as its target format. 
<br/>
<br/>
Salat has dependencies on the latest releases of:

- scalap
- casbah-core
- mongo-java-driver

!SLIDE

# Getting started

### Add the Novus repos and the salat-core dependency to your sbt project

    val novusRepo = "Novus Release Repository" at "http://repo.novus.com/releases/"
    val novusSnapsRepo = "Novus Snapshots Repository" at "http://repo.novus.com/snapshots/"

    val salat = "com.novus" %% "salat-core" % "0.0.7-SNAPSHOT"

<br/>

### Import Salat implicits and default context

    import com.novus.salat._
    import com.novus.salat.annotations._
    import com.novus.salat.global._

!SLIDE

# Demonstration: there and back again


    case class Alpha(x: String)

    scala> val a = Alpha(x = "Hello world")
    a: com.novus.salat.test.model.Alpha = Alpha(Hello world)

    scala> val dbo = grater[Alpha].asDBObject(a)
    dbo: com.mongodb.casbah.Imports.DBObject = { "_typeHint" : 
        "com.novus.salat.test.model.Alpha" , "x" : "Hello world"}

    scala> val a_* = grater[Alpha].asObject(dbo)
    a_*: com.novus.salat.test.model.Alpha = Alpha(Hello world)

    scala> a == a_*
    res0: Boolean = true

!SLIDE

# How does it work?

A case class instance extends Scala's Product trait, which provides a product 
iterator over its elements.
<br/>
<br/>
Salat used pickled Scala signatures to turn case classes into indexed fields with 
associated type information.
<br/>
<br/>
These fields are then serialized or deserialized using the memoized indexed fields 
with type information.
<br/>
<br/>
For more information about pickled Scala signatures, see
<br/>
`scala.tools.scalap.scalax.rules.scalasig.ScalaSigParser`
<br/>
<br/>
In addition, refer to this brief paper:
<br/>
[SID # 10 (draft) - Storage of pickled Scala signatures in class files](http://www.scala-lang.org/sid/10)
 
!SLIDE

# Moving parts

- a `Context` has global serialization behavior including:
    - how type hinting is handled (always, when necessary or never) - default is always
    - what the type hint is - default is `_typeHint`
    - how enums are handled (by value or by id) - default is by value
    - math context used for deserializing BigDecimal (default precision is 17)

- a `Grater` can serialize and deserialize an individual case class

!SLIDE

# Keeping things in scope

The context is an implicit supplied by importing Salat's `global` package object 
(or your own package object).
<br/>
<br/>

    import com.novus.salat.global._
<br/>

Graters are created on first request.  Use the `grater` method supplied in 
Salat's top level package object:
<br/>
<br/>

    import com.novus.salat._        
    
    grater[Alpha].asObject(dbo)

!SLIDE

# What Scala types can it handle?

- Case classes
- Case classes typed to a trait or an abstract superclass (requires @Salat)
<br/>
<br/>

### Inside a case class constructor
- Immutable collections: lists, seqs, maps whose key is a String
- Options
- any type handled by BSON encoding hooks

!SLIDE

# What can Casbah's BSON encoding hooks handle?
- org.joda.time.DateTime
- BSON types like ObjectId
<br/>
<br/>

For more information on how to write and use BSON encoding hooks, see the Casbah API docs: 
<br/>
[Briefly: Automatic Type Conversions](http://api.mongodb.org/scala/casbah/2.0.2/tutorial.html#briefly-automatic-type-conversions)
<br/>
<br/>

    com.mongodb.casbah.commons.conversions.scala.RegisterConversionHelpers()
    com.mongodb.casbah.commons.conversions.scala.RegisterJodaTimeConversionHelpers()

!SLIDE

# What Scala types does it not handle?
- classes
- case classes nested inside an enclosing class or trait (Cake pattern - coming soon)
- collection support needs to be improved - no Set, Array, mutable Map, etc.

!SLIDE

# SalatDAO: just add water

`SalatDAO` makes it simple to start working with your case class objects.  Use it 
as is or as the basis for your own DAO implementation.
<br/>
<br/>

By extending `SalatDAO`, you can do the following out of box:

- insert and get back an Option with the id
- findOne and get back an Option typed to your case class
- find and get back a Mongo cursor typed to your class
  - iterate, limit, skip and sort 
- update with a query and a case class
- save and remove case classes 

!SLIDE

# SalatDAO: getting started

    import com.novus.salat._
    import com.novus.salat.global._

    case class Omega(_id: ObjectId = new ObjectId, z: String, y: Boolean)

    object OmegaDAO extends SalatDAO[Omega, ObjectId](collection = MongoConnection()("quick-salat")("omega")) 

!SLIDE

# SalatDAO: insert and find

    scala> val o = Omega(z = "something", y = false)
    o: com.novus.salat.test.model.Omega = Omega(4dac7b3e75e1b63949139c91,
        something,false)

    scala> val _id = OmegaDAO.insert(o)
    _id: Option[com.mongodb.casbah.Imports.ObjectId] = Some(4dac7b3e75e1b63949139c91)

    scala> val o_* = OmegaDAO.findOne(MongoDBObject("z" -> "something"))
    o_*: Option[com.novus.salat.test.model.Omega] = Some(Omega(4dac7b3e75e1b63949139c91,
        something,false))

!SLIDE

# SalatDAO: update

    scala> val toUpdate = o.copy(z = "something else")
    toUpdate: com.novus.salat.test.model.Omega = Omega(4dac7b3e75e1b63949139c91,
        something else,false)

    scala> OmegaDAO.update(MongoDBObject("z" -> "something"), toUpdate) 
    com.mongodb.CommandResult = { "updatedExisting" : true , "n" : 1 , 
        "connectionId" : 255 , "err" :  null  , "ok" : 1.0}

    scala> val o_** = OmegaDAO.findOneByID(new ObjectId("4dac7b3e75e1b63949139c91"))
    o_**: Option[com.novus.salat.test.model.Omega] = Some(Omega(4dac7b3e75e1b63949139c91,
        something else,false))
<br/>
<br/>

# SalatDAO: remove

    scala> OmegaDAO.remove(updated)
    res1: com.mongodb.CommandResult = { "n" : 1 , "connectionId" : 255 , 
        "err" :  null  , "ok" : 1.0}


!SLIDE

# Trait or abstract class with @Salat

When using a case class typed to a trait or abstract superclass, mark it with the `@Salat` annotation:

    @Salat
    trait Zeta {
      val x: String
    }
    case class Eta(x: String) extends Zeta
    case class Iota(z: Zeta)

    scala> val i = Iota(z = Eta("eta"))       
    i: com.novus.salat.test.model.Iota = Iota(Eta(eta))

    scala> val dbo = grater[Iota].asDBObject(i)
    dbo: com.mongodb.casbah.Imports.DBObject = { "_typeHint" : 
        "com.novus.salat.test.model.Iota" , "z" : 
        { "_typeHint" : "com.novus.salat.test.model.Eta" , "x" : "eta"}}

    scala> val i_* = grater[Iota].asObject(dbo)
    i_*: com.novus.salat.test.model.Iota = Iota(Eta(eta))

!SLIDE

# Using @Key to change a key name

    case class Omicron(@Key("_id") id: ObjectId = new ObjectId,
                       @Key("saluations") x: String)

    scala> val o = Omicron(x = "ave")             
    o: com.novus.salat.test.model.Omicron = Omicron(4dac7fe775e101ed63792313,ave)

    scala> val dbo = grater[Omicron].asDBObject(o)
    dbo: com.mongodb.casbah.Imports.DBObject = { "_typeHint" : "com.novus.salat.test.model.Omicron" , 
        "_id" : { "$oid" : "4dac7fe775e101ed63792313"} , "saluations" : "ave"}

    scala> val o_* = grater[Omicron].asObject(dbo)
    o_*: com.novus.salat.test.model.Omicron = Omicron(4dac7fe775e101ed63792313,ave)

!SLIDE

# Using @Persist to persist a value not in case class constructor

Values marked with `@Persist` will be serialized to DBO and then discarded when deserialized back to the case class.

    case class Psi(x: String) {
      @Persist val reversed = x.reverse
    }

    scala> val p = Psi(x = "persist me")
    p: com.novus.salat.test.model.Psi = Psi(persist me)

    scala> p.reversed                   
    res0: String = em tsisrep

    scala> val dbo = grater[Psi].asDBObject(p)
    dbo: com.mongodb.casbah.Imports.DBObject = { "_typeHint" : "com.novus.salat.test.model.Psi" , 
        "x" : "persist me" , "reversed" : "em tsisrep"}

    scala> val p_* = grater[Psi].asObject(dbo)
    p_*: com.novus.salat.test.model.Psi = Psi(persist me)

!SLIDE

# Roll your own context

Switch to only using type hints "when necessary":

    package object when_necessary {
      implicit val ctx = new Context {
        val name = Some("TestContext-WhenNecessary")
        override val typeHintStrategy = TypeHintStrategy(when = 
        TypeHintFrequency.WhenNecessary, typeHint = TypeHint)
      }
    }
<br/>
<br/>
Or choose a different type hint:

    package object custom_type_hint {
      val CustomTypeHint = "_t"
      implicit val ctx = new Context {
        val name = Some("TestContext-Always-Custom-TypeHint")
        override val typeHintStrategy = TypeHintStrategy(when = TypeHintFrequency.Always, 
            typeHint = CustomTypeHint)
      }
    }

!SLIDE

# Finding out more

The specs in the Salat source code provide many usage examples.
<br/>
<br/>
Github
<br/>
[https://github.com/novus/salat](https://github.com/novus/salat)
<br/>
<br/>
Quick Start Guide
<br/>
[https://github.com/novus/salat/wiki/Quick-start](https://github.com/novus/salat/wiki/Quick-start)
<br/>
<br/>
Mailing List
<br/>
[http://groups.google.com/group/scala-salat](http://groups.google.com/group/scala-salat)
<br/>
<br/>
Twitter
<br/>
[@prasinous](http://twitter.com/prasinous)
<br/>
<br/>
Is your project using Salat?  Let us know about it!

!SLIDE

# Thank you

* [Novus](http://www.novus.com) supports the development of Salat
* [@softprops](http://twitter.com/softprops) for [picture-show](https://github.com/softprops/picture-show)

<img src="/sectionone/novus-logo.gif" />

