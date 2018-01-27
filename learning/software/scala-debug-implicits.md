## How to debug when generic derivation is not working?

typically generic derivation work by resolving implicits in multiple layers

1. Find out what is required by top layer
2. Provide all implicit type class instance 1 by 1 by some dummy implementations, eg
   ```
   def dummy[T]: T = ???
   implicit val needed: Codec[A] = dummy
   ```
3. Compile and try if it fix the issue

