!SLIDE

# All about expectations

In specs2, you can define expectations on anything that returns a ``Result``.

- Boolean
- Standard Results
    - ``success``. ``failure``, ``anError``, ``pending``, etc.
- Matcher result - specs2 has built in support for all these and more:
  - Any
  - Option / Either
  - Strings and Numbers
  - Exceptions
  - Iterable and Maps
  - XML and JSON
  - Scalaz
- ScalaCheck property
- Mock expectation
- DataTable
- Forms


# Better error descriptions

!SLIDE

# Iterable matchers

Iterable matchers are improved from specs 1.x:

    val list = List(1, 2, 3)
    list must have size(2)
    list  must contain(3, 2, 1)

Using specs2 ``only`` and ``inOrder`` can now be stated in a single line as:

    List(1, 2, 3) must contain(1, 2, 3).only.inOrder

!SLIDE

# JSON matchers

- ``/(value)`` looks for a value at the root of an Array

    """["name", "Joe" ]""" must /("name")

- ``/(key -> value)`` looks for a pair at the root of a Map

    """{ "name": "Joe" }""" must /("name" -> "Joe")
    """{ "name": "Joe" }""" must not /("name2" -> "Joe")

- ``*/(value)`` looks for a value present anywhere in a document, either
    - as an entry in an Array
    - as the value for a key in a Map
- ``*/(key -> value)`` looks for a pair anywhere in the document

JSON matchers can be chained:

    """{ "person": { "name": "Joe" } }""" must /("person") /("name" -> "Joe")

See [JsonMatchersSpec](https://github.com/etorreborre/specs2/blob/master/src/test/scala/org/specs2/matcher/JsonMatchersSpec.scala)
in the specs2 project specs for more details.

!SLIDE

# Defining your own matchers



<img class="logo" src="/img/novus-logo.gif" />
