## Lesson learnt about typeclass pattern

1. Typeclass pattern can normally replace interface inheritance
2. It is more powerful as it can defer the decision of
implementation as last as possible, with the cost of having typeclass
throughout the call chain
3. Sometime all you need is an ADT with runtime pattern matching to
perform logic lookup, which can be simpler and with less restriction
4. An example where typeclass does not work as well is when you need to
cross network boundary, conceptually typeclass does not go well
with serialization as generic does not work well with serialization
