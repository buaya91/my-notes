## Lesson learn around type safety

Typesafety in Scala does not play well with overloaded methods, because overloading is a runtime feature (likely implemented as runtime virtual dispatch?), thus generics will not be enforce due to type erasure.

  
