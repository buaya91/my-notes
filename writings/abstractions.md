## Abstractions, DRY, and naming stuff

To DRY or not to DRY is always a constant debate in software development

Name is a good proxy for abstraction, if you cannot name something
properly, it's either

1. It's a poor abstraction
2. It's an unknown abstraction that you just discovered (maybe you want
publish it with well-defined behaviour)

Poorly name abstraction is simply poor abstraction, because without an
appropriate name, chances are people will change it's behaviour into something
totally unexpected by the original author, so the abstraction might change
semantic meaning over time because people's interpretation to it's name
changes.

