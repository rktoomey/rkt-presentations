!SLIDE

<p style="font-size: 70px; color: #16325A;">specs2</p>
<br/>
<br/>
<br/>
<br/>
<p style="font-size: 45px; color: 16325a; font-style: italic;">To the barricades with executable software specifications</p>
<br/>
<br/>
[http://specs2.org](http://specs2.org)

!SLIDE

# specs2: State of the art executable software specifications

- The evolution of specs2
- How does it work?
- Migrating to specs2
    - Dependencies
    - Mutable unit specifications: the path of least resistence
    - Acceptance specifications: full-on assimilation
- Cool new features of specs2
    - JSON matchers
    - Scalacheck
    - Generate your user guide from acceptance specs
- How to guides
    - Using contexts
    - Writing your own matchers
    - Running in sequence when sharing mutable state
- Online Resources

!SLIDE

# The evolution of specs2

specs is a DSL in Scala for doing BDD (Behaviour-Driven Development).
<br/>
<br/>
specs2 is a <b><i>complete rewrite</i></b> of specs 1.x.
<br/>
<br/>
The design objectives of specs 1.x were:

- conciseness
- readability
- extensibility
- configuration
- clear implementation
- easy to fix and evolve


!SLIDE

# Recap of specs 1.x

## Conciseness

specs 1.x used the power of Scala implicits to provide an elegant syntax, but:

- reserved words like ``should``, ``can`` and ``in`` were not always convenient, and overriding them required ad hoc methods
- implicit methods were inherited from ``Specification``, causing unexpected implicit conversions and namespace pollution
- ``given ... when ... then`` support required additional methods with no added value but displaying these words


## Readability

- large amounts of code interleaved into small amounts of text devolved into spaghetti
- the [literate specs](http://code.google.com/p/specs/wiki/LiterateSpecifications#A_short_example) prototype was difficult to integrate with the rest of specs 1.x


!SLIDE

# Recap of specs 1.x

## Extensibility

Hindsight: it's very difficult to make something extensible without knowing what the requirements would be.

- some of the matchers and runners written to enhance specs 1.x were worth integrating into specs2
<br/>
<br/>

## Configuration

Customising specs 1.x required a combination of property files, reflection and system properties

- end result: user confusion, frustration and ugly hacks to work around "sensible defaults"

!SLIDE

# Recap of specs 1.x

## Clear implementation

In specs 1.x, the examples executed "on demand" when a runner asked the specification to report its successes and failures

- the examples had to run to know what the outcome was and report it to the specification
- ...but the examples ran without knowing their full execution context (what runs before and after)

!SLIDE

# Recap of specs 1.x

Worse, specs did not <i>actually</i> execute in full isolation...

- a feature intended to be helpful, [the automatic rest of local variables](http://code.google.com/p/specs/wiki/DeclareSpecifications#Execution_model) was an implementation nightmare
- what appeared to be isolation where running one example did not affect surrounding variables was achieved by cloning and carefully copying state around

!SLIDE

# Recap of specs 1.x

## Easy to fix and evolve

Although user support was always responsive, the design of specs 1.x with lots of variables and side-effects made
things harder than they had to be:

- dianosing bugs and fixing issues
- adding enhancements

!SLIDE

# A New Deal

specs2 preserves the elegant DSL of specs 1.x but with a simpler, more robust implementation.

## The design principles of specs2
<br/>

<ol>
  <li>Do not use mutable variables</li>
  <li>Use a simple structure</li>
  <li>Control the dependencies (no cycles)</li>
  <li>Control the scope of implicits</li>
</ol>



!SLIDE

# Online resources

- [http://specs2.org](http://specs2.org)
    - [Quick Start](http://etorreborre.github.com/specs2/guide/org.specs2.guide.QuickStart.html)
    - [User Guide](http://etorreborre.github.com/specs2/guide/org.specs2.UserGuide.html)
    - [Philosophy](http://etorreborre.github.com/specs2/guide/org.specs2.guide.Philosophy.html)
    - [Sample project](https://github.com/etorreborre/specs2-test)
    - [User Group](http://groups.google.com/group/specs2-users)
<br/>

- Eric Torreborre
    - [@etorreborre](http://twitter.com/#!/etorreborre)
    - [Blog](http://etorreborre.blogspot.com/)
<br/>

- Other projects mentioned in these slides
    - [Scalaz](https://github.com/scalaz/scalaz)
    - [Scalacheck](http://code.google.com/p/scalacheck/)
    - [Mockito](http://mockito.org/)
    - [Fitnesse](http://fitnesse.org/)


!SLIDE

# Thank you

- [@etorreborre](http://twitter.com/#!/etorreborre) for a project of enormous scope
- [@softprops](http://twitter.com/softprops) for [picture-show](https://github.com/softprops/picture-show)

<img class="logo" src="/specs/novus-logo.gif" />
