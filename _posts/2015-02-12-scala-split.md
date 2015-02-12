---
layout: page
title: Using Scala's split to parse a TextLine
---

I was recently writing a Scalding job where I was reading a delimited string via TextLine and relying on split 
to eventually manipulate these fields. Everything looked fine in my unit tests but when I ran the 
job on the cluster, I was finding an error like:

`cascading.tuple.TupleException: operation added the wrong number of fields, expected: ['field1', 'field2', 'field3']` 

This was because I had some data where I was expecting fields at the end of the delimited string might be empty. 
Then, I was using split and discarding them all. You can see this in the Scala console.

```scala
scala> val mysample = "a,b,c,,,,,"
mysample: String = a,b,c,,,,,

scala> val asList = mysample.split(",")
asList: Array[String] = Array(a, b, c)
```

In this case, after weighing a few options I decided on appending a field and removing in my job later:

```scala
scala> val mysample = "a,b,c,,,,," + "END"
mysample: String = a,b,c,,,,,END

scala> val asList = mysample.split(",")
asList: Array[String] = Array(a, b, c, "", "", "", "", END)
```