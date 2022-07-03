# The D Programming Language Vision Document

This document describes the high-level goals for the D programming language and its current
maintenance and development priorities according to the D Language Foundation. Those tasked with
directing specific projects in the D ecosystem can use this document as an aid for establishing task
lists and milestones. Community members interested in contributing to the D programming language can
use this document to determine the areas in which their contributions may have the greatest impact
on advancing the foundation's priorities.

This is a living document and can be expected to evolve with new details and additional priorities
over time.

## High-Level Vision

Following are high-level goals that serve to provide direction for lower-level priorities and tasks.
The items in this section are neither concrete tasks nor specific directives. They are broad goals
that the foundation has identified as general targets toward which development in the D ecosystem
should be directed. Though we welcome contributions that are not specifically aimed at furthering
these goals, new ecosystem project ideas and new language feature proposals that serve to further
these goals are of higher priority and will have a higher chance of support and/or acceptance from
the maintainers.

Note that the ordering of the goals in this list has no special significance.

### Simplification

The D programming language and its standard library have grown in size and complexity throughout
their lifetime. Both have accumulated cruft, and unwanted vestiges of D1 can still be found. This
goal is aimed at reducing complexity, eliminating cruft, and simplifying the user experience.

The following guidelines can assist in identifying tasks to make progress on this goal.

* Identify features that have negative consequences that outweigh their benefit. Such features are
  candidates for removal. (E.g., removing the comma operator clears the way for native tuples.)
* Identify features that are underused, or that have proven to provide little benefit in the general
  case. The removal of these features may be controversial, but once identified we can list them in
  an Appendix of the spec as features that should be avoided in new code. Such features are
  candidates for future deprecation and removal.
* Be wary of adding complex features, or any features that increase complexity without substantial
  benefit.
* Streamline the syntax where possible, e.g., reduce or eliminate the verbosity resulting from the
  use of several function attributes, commonly referred to as "attribute soup".

This goal extends beyond language features to encompass tooling and the ecosystem at large. The
out-of-the-box experience is critical, and the following guidelines are intended to achieve as
smooth an experience as possible.

* Users must not be required to install more than the minimal set of tools necessary to use a D
  compiler beyond the compiler's installation package.
* Language features, tools, and libraries should "just work" without requiring users to jump through
  hoops.
* The D website should leave no one behind, catering to programmers of varying skill and knowledge
  levels.
* Third-party tools (e.g., IDE plugins, linters) and libraries should be easily accessible and
  well-documented, and their maintainers should have access to any available support from the D
  Language Foundation.

D users should have the best out-of-the-box user experience we can provide them. Inspiration should
be found by examining what works and what doesn't with other programming languages: strong IDE
integration, bundled tools like linters and code formatters, file monitoring, etc. Ideas and
initiatives that improve D's user experience are welcome.

### Stronger integration with other languages

Stronger integration with C and other languages is a must. Multi-language projects, or multiple
interrelated projects using multiple languages, are more common in modern software development than
they once were. The ability to painlessly integrate D into existing code bases and toolchains is
critical to increasing adoption.

Though this does increase complexity in terms of implementation, it also helps to simplify the user
experience. That's important when one must integrate foreign language libraries into D. One should
not be required to be an expert in C to use a C API from D or to compile and link C libraries with D
applications.

This applies not just to the language, but also to tooling. It must be possible to easily set up a
build system that is flexible enough to handle foreign language projects. Package management must
account for foreign language dependencies in some form.

This is tightly coupled with the goal of simplicity: foreign language integration should "just work"
to the extent that it is technically possible.

### Memory safety

The language maintainers do not see memory safety as a fad, nor is their focus on its implementation
in the D programming language a form of "chasing" other languages. They see memory safety as a
critical component of modern programming languages going forward, as Walter Bright described [in
this panel exchange at DConf 2017](https://youtu.be/GcFuAptUExE?t=1424):

> I believe memory safety will kill C... People are tired of those expensive disasters they have
   when they have memory corruption bugs, and malware gets in and wrecks their system and, you know,
   destroys their customer trust in their products. They're just not going to put up with it
   anymore.

From the beginning, D has had memory safety features akin to those of Java (e.g., garbage collected
memory, array bounds checking, arrays that are fat pointers), but work remains on more advanced
memory safety features.

Fixing memory safety issues large and small is always a top goal. Beyond that, the following are the
current high-level goals for memory safety.

* Allow the continued use of garbage collection as the default memory management strategy without
  impact. The GC is one of D's strengths, and we should not "throw the baby out with the bath
  water".
* Allow `@safe` to be the default without it getting in the way. DIP 1000 is crucial to this because
  it eliminates most of the reasons why D code written as simply as possible is not `@safe`.
* Eliminate undefined behavior in `@safe` code.
* Find a way to allow library writers to write `@safe` containers or smart pointers. This is not
  possible in the current language due to aliasing issues. A good example: it's not possible to
  write a vector type where the following code is `@safe`.
    ```d
    auto v = vector(1, 2, 3);
    v ~= 4;
    ```
  Appending to a vector can trigger a reallocation, and therefore can only be memory safe if no
  other aliases exist anywhere in the program to the block of memory the vector uses before the
  append.
* Implement proper move semantics. This means priority should be given to known issues with the
  implementation, finishing up the move constructor DIP (or something like it), interfacing with
  C++, etc.

Even as we strive to increase memory safety in D, we must always ensure that programmers who need or
want to eschew memory safety features can do so. And they must be able to do so with minimal
friction.

### Metaprogramming

Metaprogramming is one of D's strongest features. However, there is room for improvement in terms of
the user experience. The following issues especially are of high priority.

* Reduce the impact of templates on compile times.
* Improve template error messages.

### Phobos and DRuntime

Phobos v2 is the future of the D standard library. We need to keep what works, abandon what doesn't,
and repair what's broken. Some of our current goals for Phobos v2 are:

* A versioning scheme that avoids breakage. Phobos v2, v3, v4, etc., should not break code using
  older versions.
* No autodecoding of strings. This has widely been seen as a mistake.
* No `wstring` or `dstring`. Any functions in Phobos v2 that deal with strings should deal
  exclusively with the `string` type. Users can convert from and to the other string types as
  needed.
* "Header only" as much as possible. Phobos v2 modules should minimize the number of link-time
  dependencies they introduce.
* `@nogc` as much as possible.
* It should be possible to easily identify Phobos functions with specific attributes in the
  documentation. E.g., a clickable link that shows all `@nogc` functions.
* Reduce compile-time overhead, e.g., template depth, import hierarchy. This should take precedence
  over the "header only" goal.
* Be open to adding new functions, modules, etc. There is a divide between those who believe in a
  "kitchen sink" standard library and those who support a minimal standard library backed up by a
  large ecosystem. We must find a balance that makes sense for the D programming language.

 DRuntime must be "pay as you go". It should be possible for users to opt-in to DRuntime's
 requirements without paying for the features they might not use. An ideal scenario: a D program
 that does not use the `-betterC` flag but is *effectively* `-betterC` because it uses no features
 from DRuntime should behave as if the flag had been used on purpose. A fully pay-as-you-go runtime
 could make such a scenario a reality.

### Stronger ecosystem

While we work to simplify the D ecosystem by making projects more accessible and easier to use, we
also need to strengthen it in various ways.

* Increase the number of ecosystem contributors. Many projects in the D ecosystem have only one
  maintainer, and the majority of contributions tend to be directed to the most visible or most
  popular projects. We must find ways to provide guidance and direction to contributors and organize
  contributions as much as possible. This includes contributions to the core D repositories.
* Identify mission-critical ecosystem projects. These are projects that D users find invaluable and
  hard to live without. Such projects should receive as much support from the foundation as
  possible.
* Increase the number of third-party libraries. A vast and vibrant ecosystem is key to increasing
  adoption.
* Establish a means for users to identify and distinguish between active projects, stable projects,
  and abandoned projects.
* Provide automatic CI for actively-maintained third-party projects.
* Foster a system that encourages contributors to take over abandoned projects and provide them with
  a path to upgrade to the latest version of D.
* Assist the maintainers of active projects such that they can more easily keep up as the language
  evolves.
* Strengthen editor and IDE integration. We should identify which IDEs and editors are most widely
  used in the D community and support those that are most popular, but we should also support
  projects that are usable across a range of IDE and editor projects, e.g., the implementation of
  the Language Server Protocol for D.
* Tooling for D must be competitive with tools for other languages. We must determine what sort of
  tools are necessary for our ecosystem and actively seek out capable programmers to develop them.

### Community management

As a non-profit organization, the D Language Foundation depends primarily on the work of volunteers.
Any steps we take to bring structure to ecosystem development will be fruitless if there are no
volunteers with the motivation to contribute. As such, we must look for interesting and effective
ways to encourage and motivate potential contributors.

Not all tasks can be accomplished by the efforts of volunteers alone. Some are too complex, or too
time-consuming, to fit within a volunteer's time constraints. We must take steps to increase income
from donations and other sources, allocate more funding for potential contracts, identify tasks that
are more suited to contract work, and seek out contractors to complete them.

All D users and contributors must feel comfortable participating in the D community. We recognize
that the D user base includes people from a variety of backgrounds with different expectations.
Fostering inclusiveness is a top priority. To meet this goal, we will adapt our moderation policy in
the official forums and the language used in our communications via social media as necessary. We
are open to suggestions to help us along the way.

Forum threads and Discourse channels are not the most efficient means of absorbing and responding to
community feedback. We must establish ways for community members to make their voices heard in an
environment that allows for a level of response appropriate to the depth of the feedback, e.g.,
community surveys for general opinions; one-on-one or small group meetings to hash out complex
issues, etc.

The community is the lifeblood of the language. Whether one is a contributor, an active user, or a
part-time lurker, we want all to feel welcome.

### Other

* __New Language Features__: This document should not be seen as a basis for the guaranteed
  acceptance or rejection of any new language feature proposals. The goals outlined here are
  intended to reflect the priorities and intended targets of the language maintainers so that those
  looking to make direct contributions can understand where their efforts will have the most impact.
  Any submitted DIPs that propose features aligned with these goals will still be evaluated on their
  merits. Likewise, any DIPs that are not aligned with these goals. The question should never be
  "why not add this feature", but "why should we add this feature". We should never add features
  just because we can. The maintainers must be convinced there is a compelling reason to add a
  feature.
* __Possible Future Features__: Some features are worth considering for possible inclusion in the
  future. Please note that any DIPs for items on this list are not guaranteed to be accepted. They
  are simply features that one or both of the language maintainers have expressed interest in
  exploring.
  - Pattern matching
  - Tuples
  - Async/await
  - Stronger traits
* __Fixing Fundamental Frontend Bugs__: The D frontend is the foundation upon which all language
  features are built. More important than adding new features is fixing fundamental bugs in the
  frontend. For example, errors caused by the frontend's semantic ordering have been seen in
  production code. Issues like this can potentially become more problematic, or more difficult to
  resolve, with the addition of new features. We are committed to fixing these issues and welcome
  any help in identifying and resolving them.


## Current Priority Tasks

This section will describe the projects and tasks that are the current focus of the language
maintainers.

## Future tasks

This section will describe the projects and tasks the language maintainers intend to prioritize in
the future.

## Wishlist

This section will describe potential projects and tasks the language maintainers would like to
explore.

## Contributing

This section will provide a list of specific projects and tasks, aligned with the general vision,
toward self-motivated contributors can direct their efforts.