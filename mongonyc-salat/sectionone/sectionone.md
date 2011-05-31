!SLIDE

<p style="font-size: 60px; color: 16325a;">Salat</p>
<br/>
<br/>
<br/>
<br/>
<p style="font-size: 45px; color: 16325a; font-style: italic;">Simple Serialization with MongoDB and Scala</p>
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

    val salat = "com.novus" %% "salat-core" % "0.0.8-SNAPSHOT"

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
    - how enums are handled (_by value_ or _by id_) - default is _by value_
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

# There and back again: more detail

    scala> val dbo = grater[Alpha].asDBObject(a)

Our method call to ``grater[Alpha]`` made the ``Context`` either find or create a ``Grater`` for ``Alpha``.

The ``Grater`` finds the pickled Scala signature for ``Alpha`` and uses it to identify the:

- constructor, including:
    - default arguments for constructor params
- indexed fields, including:
    - type of each field
    - key overrides at the field (``@Key``)), class or global context level
- companion objects

Once a ``Grater`` for ``Alpha`` exists, we know everything we need to know to do the following two things _without any
runtime reflection_:

 - to serialize an instance of case class ``Alpha`` as a ``DBObject``
 - to deserialize a ``DBObject`` representing an instance of ``Alpha`` into an instance of case class ``Alpha``

!SLIDE

# There and back again: more detail

    scala> val dbo = grater[Alpha].asDBObject(a)
    dbo: com.mongodb.casbah.Imports.DBObject = { "_typeHint" :
        "com.novus.salat.test.model.Alpha" , "x" : "Hello world"}

Calling ``asDBObject`` creates a ``DBObject`` representation of ``Alpha``.

## What about the type hint?

The type hint, ``_typeHint``, is not essential for deserializing ``Alpha`` and can be omitted.  We'll talk more about that later.

!SLIDE

# There and back again: more detail

    scala> val a_* = grater[Alpha].asObject(dbo)
    a_*: com.novus.salat.test.model.Alpha = Alpha(Hello world)

Turning the ``DBObject`` representation of an instance of ``Alpha`` back into an object is as easy as calling ``asObject``.

Note that we didn't use ``_typeHint`` here - we already told the ``Grater`` we were expecting ``Alpha``.

    scala> val dbo = grater[Alpha].asDBObject(a)
    dbo: com.mongodb.casbah.Imports.DBObject = { "_typeHint" :
        "com.novus.salat.test.model.Alpha" , "x" : "Hello world"}
    scala>


!SLIDE

# What Scala types can Salat handle?

- case classes
    - embedded case classes
- embedded case classes typed to a trait or abstract superclass annotated with ``@Salat``
- Scala enums
- Options
- collections



<img class="logo" src="/img/novus-logo.gif" />