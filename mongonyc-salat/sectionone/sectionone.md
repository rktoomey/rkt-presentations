!SLIDE

<p style="font-size: 60px; color: 16325a;">Salat</p>
[https://github.com/novus/salat](https://github.com/novus/salat)
<br/>
<br/>
<br/>
<br/>
<br/>
<p style="font-size: 45px; color: 16325a; font-style: italic;">Simple Serialization with MongoDB and Scala</p>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
[Rose Toomey](http://twitter.com/prasinous), Novus Partners
<br/>
June 2011 @ MongoNYC

!SLIDE

# Salat: Simple Serialization with MongoDB and Scala

- What is Salat?
- Demonstration
- How does it work?
- In detail: more advanced usage
- From zero to DAO in two minutes
- What next?
- Other projects of interest
- More information
- Credits

!SLIDE

# What is Scala?

- [http://www.scala-lang.org](http://www.scala-lang.org)
- [http://typesafe.com/technology/scala](http://typesafe.com/technology/scala)

<br/>
<br/>
Scala is a concise, elegant object-oriented language that runs on the JVM:

- statically typed
- functional
- scalable
- easy to create libraries and DSLs
-

!SLIDE

# What is Casbah?

- [MongoDB - Drivers - Java Language Center](http://www.mongodb.org/display/DOCS/Java+Language+Center)
- [http://api.mongodb.org/scala/casbah/current/](http://api.mongodb.org/scala/casbah/current/)
- [18 Months With Scala: Building a Driver For MongoDB](http://bit.ly/mdOwvl) - Brendan McAdams' presentation to Scala Days 2011 on 3 June
<br/>
<br/>
Casbah is the officially-supported Scala toolkit for MongoDB that provides an intergration layer on top of the offical [mongo-java-driver](http://github.com/mongodb/mongo-java-driver).
<br/>
<br/>
Rather than replacing the official Java driver, Casbah uses implicits, and [Pimp my Library](http://www.artima.com/weblogs/viewpost.jsp?thread=179766)
patterns to enhance the existing Java code.

!SLIDE

# What is Salat?

Salat provides fast, reliable bi-directional serialization between Scala case classes and MongoDB's ``DBObject`` format.

## Fast

Salat mines pickled Scala signatures, introduced with Scala 2.8.0, for hi-fi type information.

## Simple

Salat's design is focused: this is a library for serializing and deserializing Scala case classes.

## No runtime reflection!

"Salat" is a transliteration of the Russian word "салат", for "salad".
<br/>
<br/>
Salat is lightweight and doesn't slow you down through use of runtime reflection.

!SLIDE

# Availability

The latest release, Salat 0.0.7, is available for Scala 2.8.1.
<br/>
<br/>
The latest snapshot, Salat 0.0.8-SNAPSHOT, is available for Scala 2.8.1 and 2.9.0-1.
<br/>
<br/>
Salat is _not_ available for Scala 2.7.7 because pickled Scala signatures were introduced in Scala 2.8.0.
<br/>
<br/>
Salat is _not_ compatible with Java classes for the same reason.

!SLIDE

# Dependencies

Salat has dependencies on the latest releases of:

- scalap, a Scala library that provides functionality for parsing Scala-specific information out of classfiles
- [mongo-java-driver](http://www.mongodb.org/display/DOCS/Java+Language+Center), the official Java driver for MongoDB
- [casbah-core](http://api.mongodb.org/scala/casbah/current/), the official Scala toolkit for MongoDB

!SLIDE

# Getting started

### Add the Novus repos and the salat-core dependency to your sbt project

    val novusRepo = "Novus Release Repository" at "http://repo.novus.com/releases/"
    val novusSnapsRepo = "Novus Snapshots Repository" at "http://repo.novus.com/snapshots/"

    val salat = "com.novus" %% "salat-core" % "0.0.8-SNAPSHOT"

### Import Salat implicits and default context

    import com.novus.salat._
    import com.novus.salat.annotations._
    import com.novus.salat.global._

!SLIDE

# Try it out!

The sample code shown in this presentation is available at [rktoomey/mongonyc2011-salat-examples](https://github.com/rktoomey/mongonyc2011-salat-examples).
<br/>
<br/>
You can build and run the project using [simple-build-tool](http://code.google.com/p/simple-build-tool/).
<br/>
<br/>
The quickest way to get started experimenting is to clone the project and run ``sbt console`` to use a Scala
interpreter with a classpath that includes compiled sources and managed libs:

    ~ $ git://github.com/rktoomey/mongonyc2011-salat-examples.git
    ~ $ cd mongonyc2011-salat-examples
    ~/mongonyc2011-salat-examples $ sbt console

!SLIDE

# How to import what you need

You can try out the sample code shown in this presentation by running ``sbt console`` with these imports:

    Welcome to Scala version 2.9.0.1 (Java HotSpot(TM) 64-Bit Server VM, Java 1.6.0_24).
    Type in expressions to have them evaluated.
    Type :help for more information.

    scala> import com.novus.salat._

    scala> import com.novus.salat.global._

    scala> import com.novus.salat.annotations._

    scala> import com.mongodb.casbah.Imports._

    scala> import prasinous._


!SLIDE

# Demonstration: there and back again

Given a case class:

    package prasinous

    case class Alpha(x: String)


Serializing and deserializing is as simple as using the ``asDBObject`` and ``asObject`` methods:

    scala> val a = Alpha(x = "Hello world")
    a: prasinous.Alpha = Alpha(Hello world)

    scala> val dbo = grater[Alpha].asDBObject(a)
    dbo: com.mongodb.DBObject = { "_typeHint" : "prasinous.Alpha" , "x" : "Hello world"}

    scala> val a_* = grater[Alpha].asObject(dbo)
    a_*: prasinous.Alpha = Alpha(Hello world)

    scala> a == a_*
    res0: Boolean = true

!SLIDE

# How does it work?

A case class instance extends Scala's `Product` trait, which provides a product
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
    - how enums are handled (_by value_ or _by id_) - default is _by value_
    - math context used for deserializing BigDecimal (default precision is 17)

- a `Grater` can serialize and deserialize an individual case class

!SLIDE

# Keeping things in scope

The context is an implicit supplied by importing Salat's `global` package object
(or a custom context defined in your own package object).
<br/>
<br/>

    import com.novus.salat.global._
<br/>

Graters are created on first request.  Use the `grater` method supplied in
Salat's top level package object:
<br/>
<br/>

    import com.novus.salat._
    import com.novus.salat.global._

    grater[Alpha].asObject(dbo)


!SLIDE

# There and back again: more detail

    scala> val dbo = grater[Alpha].asDBObject(a)

Our method call to ``grater[Alpha]`` made the ``Context`` either find or create a ``Grater`` for ``Alpha``.
<br/>
<br/>
The ``Grater`` finds the pickled Scala signature for ``Alpha`` and uses it to identify the constructor, fields and
companion object.
<br/>
<br/>
Once a ``Grater`` for ``Alpha`` exists, we know everything we need to know to do the following two things _without any
runtime reflection_:

 - to serialize an instance of case class ``Alpha`` as a ``DBObject``
 - to deserialize a ``DBObject`` representing an instance of ``Alpha`` into an instance of case class ``Alpha``

!SLIDE

# There and back again: more detail

    scala> val dbo = grater[Alpha].asDBObject(a)
    dbo: com.mongodb.DBObject = { "_typeHint" : "prasinous.Alpha" , "x" : "Hello world"}

Calling ``asDBObject`` creates a ``DBObject`` representation of ``Alpha``.
<br/>
<br/>

## What about the type hint?

The type hint, ``_typeHint``, is not essential for deserializing ``Alpha`` and _could_ be omitted under some circumstances.
<br/>
<br/>
We'll talk more about that later.

!SLIDE

# There and back again: more detail

    scala> val a_* = grater[Alpha].asObject(dbo)
    a_*: prasinous.Alpha = Alpha(Hello world)

Turning the ``DBObject`` representation of an instance of ``Alpha`` back into an object is as easy as calling ``asObject``.
<br/>
<br/>
Note that we didn't use ``_typeHint`` here - we already told the ``Grater`` we were expecting ``Alpha``.

!SLIDE

# So when do we need type hints?

We need type hints to deal with two types of situations:

- case classes typed to a trait or an abstract superclass and annotated with ``@Salat``
- if the context needs to look up a ``Grater`` from a raw ``DBObject``

<br/>
<br/>
For example:

    scala> val dbo = MongoDBObject("_typeHint" -> "prasinous.Alpha", "x" -> "Hello world")
    dbo: com.mongodb.casbah.commons.Imports.DBObject = { "_typeHint" : "prasinous.Alpha" ,
      "x" : "Hello world"}

    scala> ctx.lookup(dbo)
    res1: Option[com.novus.salat.Grater[_ <: com.novus.salat.package.CaseClass]] =
    Some(Grater(class prasinous.Alpha @ com.novus.salat.global.package$$anon$1@1a33c91e))

!SLIDE

# What Scala types can Salat handle?

- case classes
    - embedded case classes
- embedded case classes typed to a trait or abstract superclass annotated with ``@Salat``
- Scala enums
- Options
- collections

### Options

Any supported type is also supported as an ``Option``.

### Collections

Maps are represented as ``DBObject``; all other collections turn into ``DBList``.

!SLIDE

# In detail: Salat collection support

Salat 0.0.7 and below support the following immutable collections:

- Map
- List
- Seq

Salat 0.0.8-SNAPSHOT and above support the following mutable and immutable collections:

- Map
- Lists and linked lists
- Seqs and indexed seqs
- Set
- Buffer
- Vector

!SLIDE

# BSON support

Salat delegates serialization for most common types to Casbah's [BSON](http://www.mongodb.org/display/DOCS/BSON) support:

- String
- Boolean
- Numeric types
    - Int, Double, Long
- ObjectID
- Date

Make sure that Casbah's BSON encoding hooks are in scope:

    com.mongodb.casbah.commons.conversions.scala.RegisterConversionHelpers()

### DateTime support

``org.joda.time.DateTime`` support requires registering Casbah's DateTime encoding hook in addition to Casbah's other conversion helpers:

    com.mongodb.casbah.commons.conversions.scala.RegisterJodaTimeConversionHelpers()

!SLIDE

# Salat extensions to BSON support

Salat provides support for converting the following types to something BSON serializes natively:

- ``Char`` is serialized as ``String``
- ``Float`` is serialized as ``Double``
- ``BigDecimal`` is serialized as ``Double``
    - the precision and rounding mode will be preserved as specified in your ``Context``
- ``BigInt`` is serialized as ``Long``

!SLIDE

# Roll your own: custom BSON encoding hooks

You can support other types by creating custom BSON hooks.  For instance, if you needed to serialize a field typed to
`java.net.URI`, you would need to create a custom BSON hook to handle this type.
<br/>
<br/>
For more information on how to write and use BSON encoding hooks, see the Casbah API docs and source code:

- [Briefly: Automatic Type Conversions](http://api.mongodb.org/scala/casbah/current/tutorial.html#briefly-automatic-type-conversions)
- Refer to ``com.mongodb.casbah.commons.conversions.scala.JodaDateTimeHelpers`` to get started.

<br/>
<br/>
The [Casbah mailing group](http://groups.google.com/group/mongodb-casbah-users) is another valuable resource

!SLIDE

# Unsupported types

Salat can't support any of these types right now:

- Nested inner classes (as used in Cake pattern)
- A class typed at the top-level to a trait or an abstract superclass
- ``com.mongodb.DBRef``

<br/>
<br/>
Salat can't support these types because the mongo-java-driver doesn't support them:

- Any type of Map whose key is not a String
    - any type of map whose key is a String containing ``.`` or ``$``

!SLIDE

# In detail: traits and abstract superclasses

With one easy extra step, Salat can handle fields and collections typed to a trait or an abstract superclass.

### Without traits

    case class Zeta(x: String)
    case class Iota(z: Zeta)

### With a trait

In this example, `Iota`'s `z` field is parameterized to a
trait, namely `trait Zeta`. To avoid performance degradation at
run time, you must annotate `trait Zeta` with the `@Salat`
annotation, as shown below.

    @Salat
    trait Zeta {
      val x: String
    }
    case class Eta(x: String) extends Zeta
    case class Iota(z: Zeta)

!SLIDE

# In detail: traits and abstract superclasses

To deserialize from `DBObject` back to `Iota`, the `_typeHint` field is necessary!

    scala> val i = Iota(z = Eta("eta"))
    i: prasinous.Iota = Iota(Eta(eta))

    scala> val dbo = grater[Iota].asDBObject(i)
    dbo: com.mongodb.DBObject = { "_typeHint" : "prasinous.Iota" ,
      "z" : { "_typeHint" : "prasinous.Eta" , "x" : "eta"}}

    scala> val i_* = grater[Iota].asObject(dbo)
    i_*: prasinous.Iota = Iota(Eta(eta))

!SLIDE

# In detail: key remapping

Use ``@Key`` to perform ad hoc key remapping:

    scala> val o = Omicron(o = "Same old")
    o: prasinous.Omicron = Omicron(4de5df4ce4ffd3ffea79e486,Same old)

    scala> val dbo = grater[Omicron].asDBObject(o)
    dbo: com.mongodb.DBObject = { "_typeHint" : "prasinous.Omicron" ,
      "_id" : { "$oid" : "4de5df4ce4ffd3ffea79e486"} ,
      "o" : "Same old"}

You can also override keys on a per-class basis or globally - see the [custom context](https://github.com/novus/salat/wiki/CustomContext)
wiki page for more information.

!SLIDE

# In detail: serialize a value outside the case class constructor

    case class Psi(x: String) {
      @Persist val reversed = x.reverse
    }

Values marked with `@Persist` will be serialized to DBO and then discarded when deserialized
back to the case class.

    scala> val p = Psi(x = "persist me")
    p: prasinous.Psi = Psi(persist me)

    scala> p.reversed
    res0: String = em tsisrep

    scala> val dbo = grater[Psi].asDBObject(p)
    dbo: com.mongodb.DBObject = { "_typeHint" : "prasinous.Psi" , "x" : "persist me" , "reversed" : "em tsisrep"}

    scala> val p_* = grater[Psi].asObject(dbo)
    p_*: prasinous.Psi = Psi(persist me)

!SLIDE

# From zero to DAO in two minutes

`SalatDAO` makes it simple to start working with your case class objects.  Use it
as is or as the basis for your own DAO implementation.
<br/>
<br/>

By extending `SalatDAO`, you can do the following out of box:

- insert and get back an `Option` with the id
- findOne and get back an `Option` typed to your case class
- find and get back a Mongo cursor typed to your class
  - iterate, limit, skip and sort
- update with a query and a case class
- save and remove case classes
- projections
- built-in support for child collections

!SLIDE

# SalatDAO: getting started

Extend ``SalatDAO``, typing it to your case class and ID, and supply a collection.

    package prasinous

    import com.mongodb.casbah.Imports._
    import com.novus.salat.global._
    import com.novus.salat.dao.SalatDAO

    case class Omega(_id: ObjectId = new ObjectId, y: String, z: Int)

    object OmegaDAO extends SalatDAO[Omega, ObjectId](
      collection = MongoConnection()("mongonyc2011-salat-example")("omega")
    )

!SLIDE

# SalatDAO: insert and find

    scala> val o = Omega(y = "E-123", z = 24)
    o: prasinous.Omega = Omega(4de5c7e7e4ff62f56ace1ccb,E-123,24)

    scala> val _id = OmegaDAO.insert(o)
    _id: Option[com.mongodb.casbah.Imports.ObjectId] = Some(4de5c7e7e4ff62f56ace1ccb)

    scala> val o_* = OmegaDAO.findOneByID(new ObjectId("4de5c7e7e4ff62f56ace1ccb"))
    o_*: Option[prasinous.Omega] = Some(Omega(4de5c7e7e4ff62f56ace1ccb,E-123,24))

    scala> val o_* = OmegaDAO.findOne(MongoDBObject("y" -> "E-123"))
    o_*: Option[prasinous.Omega] = Some(Omega(4de5c7e7e4ff62f56ace1ccb,E-123,24))

    scala> val o_* = OmegaDAO.find(MongoDBObject("y" -> "E-123")).toList
    o_*: List[prasinous.Omega] = List(Omega(4de5c7e7e4ff62f56ace1ccb,E-123,24))

!SLIDE

# SalatDAO: update

You can update using a ``DBObject``:

    scala> OmegaDAO.update(MongoDBObject("_id" -> new ObjectId("4de5c7e7e4ff62f56ace1ccb")), MongoDBObject("y" -> "E-124", "z" -> 25))

    scala> val o_* = OmegaDAO.findOneByID(new ObjectId("4de5c7e7e4ff62f56ace1ccb"))
    o_*: Option[prasinous.Omega] = Some(Omega(4de5c7e7e4ff62f56ace1ccb,E-124,25))

Or a case class, which requires specifying arguments for ``upsert``, ``multi`` and a ``WriteConcern``:

    scala> OmegaDAO.update(q = MongoDBObject("_id" -> new ObjectId("4de5c7e7e4ff62f56ace1ccb")),
    t = o.copy(y = "E-125", z = 26), upsert = false, multi = false, wc = new WriteConcern)

    scala> val o_* = OmegaDAO.findOneByID(new ObjectId("4de5c7e7e4ff62f56ace1ccb"))
    o_*: Option[prasinous.Omega] = Some(Omega(4de5c7e7e4ff62f56ace1ccb,E-125,26))

!SLIDE

# SalatDAO: save

    scala> OmegaDAO.save(o.copy(y = "E-126", z = 27))

    scala> val o_* = OmegaDAO.findOneByID(new ObjectId("4de5c7e7e4ff62f56ace1ccb"))
    o_*: Option[prasinous.Omega] = Some(Omega(4de5c7e7e4ff62f56ace1ccb,E-126,27))

!SLIDE

# SalatDAO: remove

    scala> OmegaDAO.remove(o.copy(y = "E-126", z = 27))

    scala> val o_* = OmegaDAO.findOneByID(new ObjectId("4de5c7e7e4ff62f56ace1ccb"))
    o_*: Option[prasinous.Omega] = None

!SLIDE

# SalatDAO: primitive projections

    case class Theta(_id: ObjectId = new ObjectId, x: String, y: String)

Use projections to bring back a typed list that discards ``null`` or ``None``.

    scala> val _ids = ThetaDAO.insert(Theta(x = "x1", y = "y1"),
        Theta(x = "x2", y = "y2"), Theta(x = "x3", y = "y3"), Theta(x = "x4", y = "y4"),
        Theta(x = "x5", y = null))
    _ids: List[Option[com.mongodb.casbah.Imports.ObjectId]] = List(Some(4de5d418e4ff796559972ad3),
      Some(4de5d418e4ff796559972ad4), Some(4de5d418e4ff796559972ad5), Some(4de5d418e4ff796559972ad6),
      Some(4de5d418e4ff796559972ad7))

    scala> ThetaDAO.primitiveProjections[String](MongoDBObject(), "y")
    res0: List[String] = List(y1, y2, y3, y4)

    scala> ThetaDAO.primitiveProjections[String](MongoDBObject(), "x")
    res1: List[String] = List(x1, x2, x3, x4, x5)

!SLIDE

# SalatDAO: case class projections

    case class Nu(x: String, y: String)
    case class Kappa(@Key("_id") id: ObjectId = new ObjectId, k: String, nu: Nu)

Projections can also handle case classes.

    scala> val _ids = KappaDAO.insert(Kappa(k = "k1", nu = Nu(x = "x1", y = "y1")),
      Kappa(k = "k2", nu = Nu(x = "x2", y = "y2")),
      Kappa(k = "k3", nu = Nu(x = "x3", y = "y3")))
    _ids: List[Option[com.mongodb.casbah.Imports.ObjectId]] = List(Some(4de5d5bbe4ff17cca27b2872),
      Some(4de5d5bbe4ff17cca27b2873), Some(4de5d5bbe4ff17cca27b2874))

    scala> KappaDAO.projection[Nu](MongoDBObject("k" -> "k1"), "nu")
    res0: Option[prasinous.Nu] = Some(Nu(x1,y1))

    scala> KappaDAO.projections[Nu](MongoDBObject("k" -> MongoDBObject("$in" -> List("k2", "k3"))), "nu")
    res1: List[prasinous.Nu] = List(Nu(x2,y2), Nu(x3,y3))

!SLIDE

# SalatDAO: practical concerns

## Write concerns

The following methods take a ``WriteConcern`` parameter that defaults to the collection's own write concern:

- ``insert``
- ``update``
- ``save``
- ``remove``

## What if something goes wrong?

If something goes wrong, these methods will blow up with a detailed runtime exception.


!SLIDE

# Other projects of interest

Salat is not all things to all people.  Some functionality will _always_ be outside of the project vision.
<br/>
<br/>
If you need something that Salat just doesn't do, here's some information about other projects that you might be
interested in.

!SLIDE

# Morphia: more of everything

[Morphia](http://code.google.com/p/morphia/), a type-safe Java ORM for MongoDB, provides:

- Fully featured ORM
    - define embedded and semantic relationships between objects
    - optimistic locking
    - inheritance strategies for mapping model objects to collections
    - lifecycle method annotations like ``@PrePersist``, ``@PostPersist``, ``@PreLoad``, ``@PostLoad``
- type-safe queries
- validation

<br/>
<span class="new">
If you're interested in more information about Morphia, look on
<a href="http://www.10gen.com/conferences/mongonyc2011">MongoNYC 2011</a>
for slides and video from Scott Hernandez' talk on "Morphia: Easy Java Persistence" this morning.
</span>

!SLIDE

# Spring Data

<span class="clarification">The Spring Data lead, Mark Pollack from VMWare, is presenting "MongoDB for Java Devs with Spring and CloudFoundry" in
the Rusack room from 3:00 - 3:30pm</span>
<br/>
<br/>
[Spring Data](http://www.springsource.org/spring-data) is a project to make it easy for Spring
 applications to use non-relational databases, map-reduce frameworks and cloud-based data services.
<br/>
<br/>
The MongoDB support in [spring-data-document](https://github.com/SpringSource/spring-data-document) is currently in beta.
<br/>
<br/>
Examples are available:

- [spring-data-document-examples](https://github.com/SpringSource/spring-data-document-examples)

!SLIDE

# Query DSLs

## Type-safe MongoDB

- [Rogue](https://github.com/foursquare/rogue) - open-sourced by FourSquare, this project provides an internal DSL that
works with the Lift web framework

## Type-safe Google data store

 - [HighChair](https://github.com/chrislewis/highchair) - created by [Chris Lewis](http://twitter.com/#!/chrslws), this toolset for
 developing Google App Engine services and applications in Scala includes a type-safe query DSL that provides a feel intentionally
 similar to [Rogue](https://github.com/foursquare/rogue) but without the Lift dependency

!SLIDE

# Query DSLs

## Type-safe SQL

- [squeryl](http://squeryl.org/) - query DSL and an ORM
- [scala-query](https://github.com/szeiger/scala-query)
- [Circumflex ORM](http://circumflex.ru/projects/orm/index.html)

# Plain SQL but a plusher ride

- [Anorm](http://scala.playframework.org/documentation/scala-0.9/anorm), SQL data access with Play Scala
- [Querulous](https://github.com/twitter/querulous)

!SLIDE

# Who's using Salat?

- [salat-avro](https://github.com/T8Webware/salat-avro), Fast bi-directional Scala case class to Avro serialization from T8Webware and [@rubbish](http://twitter.com/rubbish)
- [smidm](https://github.com/wstrange/smidm), Warren Strange's experimental identity sync manager using Scala and Mongo and the Identity Connector Framework

# Projects that make use of Salat's approach to ScalaSig

- [Jerkson](https://github.com/codahale/jerkson), [@coda](http://twitter.com/coda)'s Scala wrapper for Jackson which brings Scala's ease-of-use to Jackson's features
- [scala-mongo-thingy](https://github.com/havocp/mongo-scala-thingy), a MongoDB -> BSON AST -> (JSON or CaseClass) pipeline from [@havocp](http://twitter.com/havocp)

<br/>
<br/>
Is your project using Salat?  Let us know about it!

!SLIDE

# What happens next?

We're working to make the code in Salat more modular and general purpose.

- our tools for working with pickled Scala signatures will be moved to `salat-util`, a standalone module without dependencies
- the current `salat-core` module will contain
- the Casbah dependencies will be moved out to `salat-casbah` in preparation for adding...
- a new Salat module for using Brendan McAdams' [Hammersmith](https://github.com/bwmcadams/hammersmith) project

!SLIDE

# Briefly: Hammersmith

[Hammersmith](https://github.com/bwmcadams/hammersmith) is a pure asynchronous MongoDB driver for Scala.
<br/>
<br/>
See slides from [Hammersmith: Netty, Scala and MongoDB](http://www.speakerdeck.com/presentations/4dac9ecb5753081d23000002.pdf) - Brendan's presentation
at a recent [ny-scala](http://www.meetup.com/ny-scala/) meetup.

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

!SLIDE

# Thank you

* [Novus](http://www.novus.com) supports the development of Salat
* [10gen](http://www.10gen.com/) for supporting the ongoing development of [Casbah](http://api.mongodb.org/scala/casbah/current/index.html)
* [Eric Torreborre](http://twitter.com/etorreborre) for [specs2](http://specs2.org), which I use to write specifications for Salat
  * see slides for [specs2: What's new in the Scala BDD world?], my recent ny-scala meetup presentation on specs2
* [Brendan McAdams](http://twitter.com/rit) for [Casbah](https://github.com/mongodb/casbah/) and [Hammersmith](https://github.com/bwmcadams/hammersmith)
  * and for being a wellspring of constructive inspiration on how open source projects can make things better...
* [@softprops](http://twitter.com/softprops) for [picture-show](https://github.com/softprops/picture-show)

<img class="logo" src="/img/novus-logo.gif" />
<img class="mongoDBLogo" src="/img/PoweredMongoDBbeige66.png" />