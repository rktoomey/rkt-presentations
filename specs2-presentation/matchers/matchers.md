!SLIDE

# Matchers: making expectations easy

In specs2, you can define expectations on anything that returns a ``Result``.

- Boolean
- Standard Results
    - ``success``, ``failure``, ``anError``, ``pending``, etc.
- Matcher result - specs2 has built in support for all these and more:
    - Any
    - Option / Either
    - Strings and Numbers
    - Exceptions
    - Iterable and Maps
    - XML and JSON
    - Scalaz
    - Parser Combinator matchers
- ScalaCheck property
- Mock expectation
- DataTable
- Forms

!SLIDE

# Where to find information about matchers

- [Matchers Guide](http://etorreborre.github.com/specs2/guide/org.specs2.guide.Matchers.html)
- [specs2 matcher code](https://github.com/etorreborre/specs2/tree/master/src/main/scala/org/specs2/matcher)
- [specs2 matcher specs](https://github.com/etorreborre/specs2/tree/master/src/test/scala/org/specs2/matcher)

!SLIDE

# Iterable matchers

## specs 1.x:

    val list = List(1, 2, 3)
    list must have size(3)
    list must containInOrder(1, 2, 3)

## specs2

Using ``only`` and ``inOrder`` we can state this in one shot:

    List(1, 2, 3) must contain(1, 2, 3).only.inOrder

!SLIDE

# JSON matchers

<span class="clarification">The JSON matchers rely on the standard Scala libs.</span>
<br/>
<br/>

``/(value)`` looks for a value at the root of an Array

    """["name", "Joe" ]""" must /("name")

``/(key -> value)`` looks for a pair at the root of a Map

    """{ "name": "Joe" }""" must /("name" -> "Joe")
    """{ "name": "Joe" }""" must not /("name2" -> "Joe")
<br/>

``*/(value)`` looks for a value present anywhere in a document, either as an entry in an Array
or as the value for a key in a Map

``*/(key -> value)`` looks for a pair anywhere in the document
<br/>
<br/>

JSON matchers can be chained:

    """{ "person": { "name": "Joe" } }""" must /("person") /("name" -> "Joe")
<br/>
See [JsonMatchersSpec](https://github.com/etorreborre/specs2/blob/master/src/test/scala/org/specs2/matcher/JsonMatchersSpec.scala)
in the specs2 project specs for more details.

!SLIDE

# specs2 matchers in the wild

- Eric Torreborre's fork of ``lift/framework`` is using the new ``ParserMatchers`` trait to specify the parsers helpers
    - [CombParserHelpersSpec](http://bit.ly/idczY2) using specs2
    - compare with original [CombParserHelpersSpec](https://github.com/lift/framework/blob/master/core/util/src/test/scala/net/liftweb/util/CombParserHelpersSpec.scala) using specs 1.x

- [Casbah](https://github.com/mongodb/casbah) is now using specs2
    - shiny new custom ``DBObject`` matchers in [DBObjectMatchers](https://github.com/mongodb/casbah/blob/master/casbah-commons/src/main/scala/test/MongoDBSpecification.scala)
    - usage examples in [DSLCoreOperatorsSpec](https://github.com/mongodb/casbah/blob/master/casbah-query/src/test/scala/DSLCoreOperatorsSpec.scala)
    and [BarewordOperatorsSpec](https://github.com/mongodb/casbah/blob/master/casbah-query/src/test/scala/BarewordOperatorsSpec.scala)

- TDD on Android using Robolectric with Specs2
    - [https://github.com/jbrechtel/robospecs](https://github.com/jbrechtel/robospecs)

- [Salat-Avro](https://github.com/T8Webware/salat-avro) has more examples of JSON matchers




<img class="logo" src="/img/novus-logo.gif" />
