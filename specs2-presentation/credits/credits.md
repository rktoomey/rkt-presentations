!SLIDE

# Online resources

- [http://specs2.org](http://specs2.org)
    - [Quick Start](http://etorreborre.github.com/specs2/guide/org.specs2.guide.QuickStart.html)
    - <span class="clarification">Updated</span> [User Guide](http://etorreborre.github.com/specs2/guide/org.specs2.UserGuide.html)
    - [Sample project](https://github.com/etorreborre/specs2-test)
    - [User Group](http://groups.google.com/group/specs2-users)
    - [@spec2.org]()
<br/>

- Eric Torreborre
    - [@etorreborre](http://twitter.com/#!/etorreborre)
    - [Blog](http://etorreborre.blogspot.com/)
    - <span class="new">NEW</span> [specs2 migration guide](http://etorreborre.blogspot.com/2011/05/specs2-migration-guide.html)
<br/>

- Other projects mentioned in these slides
    - [Scalaz](https://github.com/scalaz/scalaz)
    - [Scalacheck](http://code.google.com/p/scalacheck/)
    - [Mockito](http://mockito.org/)
    - [Fitnesse](http://fitnesse.org/)

!SLIDE

# Sample code

- [Official specs2 examples](https://github.com/etorreborre/specs2/tree/1.3/src/test/scala/org/specs2/examples)
- [My sample code for this specs2 presentation](https://github.com/rktoomey/specs2-examples)
- [Salat](https://github.com/novus/salat) uses specs2
    - [Main conversion changeset](https://github.com/novus/salat/commit/493247fb8d9d5eeafb50f44fbceab1a0e6107af5)
    - [Cleanup work](https://github.com/novus/salat/commit/8903b1cf53c059208edff21d0b1737c0a40ad845)
    - [SalatDAOSpec](https://github.com/novus/salat/blob/master/salat-core/src/test/scala/com/novus/salat/test/dao/SalatDAOSpec.scala) showing using contexts and demonstrating how to run a test sequentially
    - [SalatSpec](https://github.com/novus/salat/blob/master/salat-core/src/test/scala/com/novus/salat/test/SalatSpec.scala) - a mutable ``Specification`` trait that has before and after steps

# Useful things to think about

- [specs2 Philosophy](http://etorreborre.github.com/specs2/guide/org.specs2.guide.Philosophy.html)
- Josh Suereth's NEScala symposium talk, [Implicits without the import tax](http://bit.ly/ev3qeS)

!SLIDE

# Thank you

- [@etorreborre](http://twitter.com/#!/etorreborre) for writing and beautifully documenting specs2, and so much more:
    - for assisting at every stage in the preparation and remix of this presentation
    - for clarifying the user guide in response to some of the questions from this presentation
    - for being so responsive at more hours of the day and night than I could possibly expect  (especially given the time
    difference between New York and Sydney - it's clear he's someone who cares a lot about helping people to understand
    and use specs2)
- [@softprops](http://twitter.com/softprops) for [picture-show](https://github.com/softprops/picture-show)
- [Novus Partners](http://www.novus.com) for hosting this [ny-scala](http://www.meetup.com/ny-scala/) meetup
- [@n8han](http://twitter.com/#!/n8han) for filming this talk
- all of the attendees at the presentation for displaying such interest and asking so many questions!
- everyone who has re-tweeted this presentation, for helping to create a vital and interesting discussion around BDD testing
frameworks in Scala

<img class="logo" src="/img/novus-logo.gif" />
