## Some of the mistakes I've made

### Code related: Misusing trait mixin to share behaviours

It started as a way to share data in test (aka test fixture), all data are
immutable so we dont think it is a problem, then inevitably
the need to share behaviours arise, it was alright at the beginning because
behaviours were stateless, but soon we find many of the behaviours depends on
some shared resource manager, the main offender being ActorSystem, which
requires manual shutdown.

ActorSystem cannot be shared as a global singleton as that would causes
more coupling and it's challenging to terminate it, eg. how do you know if
other part of the code still needs it.

Given every piece of shared code is a trait, the only way to share
actor system is via overriding on each call site, the whole mixin stack
become too coupled and fragile, to make thing worse, trait initialize
to null in scala (2.12.1), this creates runtime NPE.

The solution for this specific situation is to use class to contain
variable that need to be initialized (ActorSystem), this does not solve
the problem of too much coupling/sharing due to traits, but it solves the
trait variables initialize problem, and is less costly to implement.

Lesson: Only use trait mixin for dead-simple cases, `prefer composition` by
passing instances instead.

### Code related: Introduce unintentional side-effects that might cause data corruption

I accidentally wrote some code that trigger some terrible side-effect.
Several mistake happened in this process

1. Me, as the developer, did not notice that side-effect is arbritrary code
that can talk to database.
2. Given I didn't think it as an arbritrary code runner, I did not add
test to safe-guard around it.
3. We dont have test for this kind of edge case, ie. to test if side-effect
happens once and only once.

Lesson: I need a checklist whenever doing commit, the checklist should always
question myself about

1. usage of side-effects, do you really need to do it?
2. implication, what are the side-effects that's gonna happen
3. Have you tested the side-effectful code?

### Comm related: I failed to stay cool when I am in tricky situation

Reason 1: I am too eager to prove myself, and this often backfire

I need to learn that slowing down is beneficial for me, and also my team.

### Fixing issue: I failed to understand the problem at hand, and gave a wrong responses

My first reaction should always be reproduce the error, find out the root cause,
verify if the root cause is correct by trial and error.

I should not try to make guess and shoot into the dark.

### Whenever proposing non-trivial change, always prepare a comprehensive argument and tradeoff

### Stop cracking jokes that's borderline offensive, you can't handle that
