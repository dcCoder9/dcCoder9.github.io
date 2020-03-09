---
title: "Automating testing in C# with AutoFixture"
published: true
---
I spent some time last week playing around with [AutoFixture](https://github.com/AutoFixture/AutoFixture). It's an open-source library with over 3 million nuget downloads that aims to help create "random" data for use in unit tests. It has reasonable defaults for creating data such as integers or strings, and most importantly it will automatically create complex c# objects for you. Despite having high hopes for its usefulness, I found that overall it wasn't as valuable for our code as I hoped, and at present I don't see it bringing enough value to warrant people having to spend time learning its API.

## Pros
- Decouples constructor implementation from object creation, which prevents breaking unit tests when changing constructor signature
- Helps populate lots of fields that might not be relevant to a test
- Integrates with Moq and NUnit well
- Object customizations can be packaged into 'modules' and shared/reused easily
Constructs objects that can be difficult to construct due to things like contravariance

## Cons
- Documentation isn't the best to read - much of the documentation is written as blog posts
- Possibility of flaky tests if randomly created values are not considered (as they are non-deterministic)
- After looking through several test projects, **I found few places where it seemed more valuable than the existing code without AutoFixture**

# Example Use Case
As a simple example, say we want to test a component that combines a `Person`'s first and last names into a single string. We would probably do something like this:

```c#
public void TestConcatenateName(){
   var people = new List<Person> {
     new Person {
       Id = 234,
       FirstName = "Joe",
       LastName = "Smith"
     },
     new Input {
       Id = 617
       FirstName = "Jane",
       LastName = "Doe"
     }};

    var expectedValues = new List<string> {
      "Smith, Joe", "Doe, Jane"
    }

    foreach (var i = 0; i < people.Count(); ++i){
      var result = NameJoiner.getFullName(p);
      Assert.AreEqual(expected, result);
    }
}

```
If we wanted more result, this list would grow longer (and take more time and space to write). In autofixture, we could create our input objects like this:

```c#
  var fixture = new Fixture();
  var mockResults = fixture.CreateMany<Person>(2).ToList();
```
This will create a list of 2 `Person`s with random data for all their fields. The defaults are reasonable, and the algorithms for creating the data are customizable and can be overwritten.
If we were trying to test the same method using our autofixture-generated inputs, we could easily call the method - but we'd have to programmatically define the expected result, because the inputs are non-deterministic.

"Programmatically defining the expected result" sounds a lot like re-writing the component being tested to me. I believe the biggest problem with AutoFixture's philosophy is that writing test assertions based on non-deterministic input is hard, and can require much more code (and room for error) than simply hard-coding expected input and output values. If your test ends up being complicated, then who's testing your test?

A potential value-add for AutoFixture might be if our `Person` class had a lot of other fields that were irrelevant. We could have AutoFixture create the `Person`, overwrite the properties for `firstName` and `lastName`, and then hard-code out expected return like we did before.
