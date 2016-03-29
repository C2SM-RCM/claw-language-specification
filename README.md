# CLAW language specification

#### Versions
##### Stable
The current stable version is the version 0.1 and can be find in the branch
`v0.1` or as a packaged release in the releases tab.

##### On going
Currently designing version 0.2 of the language specification. This work is done
in the `master` branch.

##### History
* **Iteration 0.1**:
  * Define low-level block transformation.
    * Loop transformations.
    * Remove transformation.
* **Iteration 0.2**:
  * Refine low-level transformation from 0.1 if needed.
    * Add `collapse` clause to loop-fusion directive.
  * Add missing low-level transformation from the requirements.  
    * Add loop transformation `loop-hoist`.
    * Add array notation transformation `array-transform`.
    * Add claw transformation `kcache` for column caching.
    * Add claw transformation `` for on the fly computation.
  * Start to abstract low-level transformation.
* **Next iterations**:
  * Refine previous iterations
  * Getting higher abstraction

#### General information about the CLAW language
The directives are either local or global.

* Local directive: those directives have a limited impact on a local block of
code (for example, only in a subroutine)
* Global directive: those directives can have an impact on the whole
application.


This language is separated in the followings sections:
* [CLAW abstraction](./definition/claw-abstraction.md)
  (specific abstraction for climate system modeling build on the top of other
  directives)
  * k caching (column caching)
* [Loop transformation](./definition/loop-transform.md)
  * loop fusion
  * loop interchange/reordering
  * loop extraction
  * loop hoisting
* [OpenACC abstractions/helpers](./definition/openacc-abstraction.md)
  * * array notation to do statement  
* [On the fly computation](./definition/on-the-fly.md)
* [Utilities](./definition/utilities.md)
  * remove

##### Line continuation
CLAW directives can be defined on several line. The syntax is described in the
listing below:

```Fortran
!$claw directive options &
!$claw options
```


##### Interpretation order of the CLAW directives
The claw directives can be combined together. For example, loop-fusion and
loop-interchange can be used together in a group of nested loops.

The interpretation order of the directives is the following:

1. remove
2. array-transform
3. loop-extract
4. loop-fusion
5. loop-hoist
6. loop-interchange
7. formatting transformation (internal transformation only)

Users must be aware that directives transformation are applied sequentially and
therefore, a transformation can be performed on already transformed code.
