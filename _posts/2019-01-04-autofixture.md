---
title: "Using Todoist to manage and prioritize individual work"
published: false
---
I've spend some time during last week playing around with [AutoFixture](https://github.com/AutoFixture/AutoFixture). It's an open-source library with over 3 million nuget downloads that aims to help create "random" data for use in unit tests. It has reasonable defaults for creating data such as integers or strings, and most importantly it will automatically create complex c# objects for you. Despite having high hopes for its usefulness, I found that overall it wasn't as valuable for our code as I hoped, and at present I don't see it bringing enough value to warrant people having to spend time learning its API.

## Pros
Decouples constructor implementation from object creation, which prevents breaking unit tests when changing constructor signature
Helps populate lots of fields that might not be relevant to a test
Integrates with Moq and NUnit well
Object customizations can be packaged into 'modules' and shared/reused easily
Constructs objects that can be difficult to construct due to things like contravariance
## Cons
Documentation isn't the best to read - much of the documentation is written as blog posts
Possibility of flaky tests if randomly created values are not considered (as they are non-deterministic)
After looking through several test projects, I found few places where it seemed more valuable than the existing code without AutoFixture
 

# Example Use Case
As a simple example, say we want to test a component that combines a `Person`'s first and last names into a single string. We would probably do something like this:

```c#
var mockResults = new List<Input> {
  new Input {
    Key = 23,
    Name = 'input 23',
    Value = 4263  
  },
  new Input {
    Key = 52,
    Name = 'result 52',
    Value = 286
  }
```
If we wanted more result, this list would grow longer. In autofixture, we could create something similar via:

```c#
var fixture = new Fixture();
var mockResults = fixture.CreateMany<Input>(2).ToList();
```
This will create a list of 2 results with random-ish data. The defaults are reasonable, and the algorithms for creating the data are customizable and can be overwritten. 
If we were trying to test a method that created a dictionary from input to result based on our autofixture-generated inputs using a regression database, we could easily call the method - but we'd have to programmatically define the expected result, because the inputs are non-deterministic.
"Programmatically defining the expected result" sounds a lot like writing the component being tested to me.

More complicated object construction
Toy examples work very well, but things get a bit more complicated with real examples. In aggregate reporting, I tried using AutoFixture to create an instance of a relatively complicated object, FormattedEventSetDefinitionNaming. The class looks like the following:

public class FormattedCriteriaKey
{
   public string JsLookupKey { get; set; }
   public string Name { get; set; }
}
 
public class FormattedEventSetDefinitionNaming
{
    // eventTagItemId => FormattedCriteriaKey
   public IReadOnlyDictionary<int, FormattedCriteriaKey> EventTagCriteriaKeyMapping { get; set; }
 
 
    // eventSetDefinitionId => JsLookupKey => BreakoutNameAndOrder
   public IReadOnlyDictionary<int, IReadOnlyDictionary<string, BreakoutNameAndOrder>> EventSetDefinitionToCriteriaKeyNamingMapping { get; set; }
}
This object is relatively complex because the two dictionaries are related - the values (formattedCriteriaKeys) in the EventTagCriteriaKeyMapping function as the second key in EventSetDefinitionToCriteriaKeyNamingMapping. So randomly generating data won't work, as that relationship won't be enforced.

However, we could effectively overwrite the default algorithm by doing the following:

var fixture = new Fixture();
var esdNamingFixture = fixture.Create<FormattedEventSetDefinitionNaming>();
var thingIWant = esdNamingFixture.EventSetDefinitionToCriteriaKeyNamingMapping.Values.SelectMany(innerDict =>
      innerDict.Select(kvp => new KeyValuePair<int, FormattedCriteriaKey>(fixture.Create<int>(), new FormattedCriteriaKey {JsLookupKey = kvp.Key, Name = fixture.Create<string>()})))
   .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
esdNamingFixture.EventTagCriteriaKeyMapping = thingIWant;
fixture.Register(() => esdNamingFixture);
What I've done here is build EventTagCriteriaKeyMapping based off the values in EventSetDefinitionIdToCriteriaKeyNamingMapping, and then registered the object with AutoFixture, so that the next time someone calls `fixture.Create<FormattedEventSetDefinitionNaming>()`, they will get a FormattedEventSetDefinitionNaming class with both dictionaries in sync.

Replacing hard-coded values with autofixture-generated data
I believe the biggest problem with AutoFixture's philosophy is that writing test assertions based on non-deterministic input is hard, and can require much more code (and room for error) than simply hard-coding expected input and output values.

For example, if we wanted to test a utility that combined a person's firstName and lastName into a single string, we could hardcode the test to look like the following:

public void TestConcatenateName(){
    var firstName = "John";
    var lastName = "Smith";
    var expected = "Smith, John";
    var result = NameJoiner.join(firstName, lastName);
    Assert.AreEqual(expected, result);
}
If we wanted to write a similar test using AutoFixture-generated data, it would look more like:

public void TestConcatenateName(){
    var fixture = new Fixture();
    var firstName = fixture.Create<string>();
    var lastName = fixture.Create<string>();
    var expected = lastName + ", " + firstName;
    var result = NameJoiner.join(firstName, lastName);
    Assert.AreEqual(expected, result);
}
At this point we have written a test that, in my opinion, has more room for bugs, is less descriptive, and harder to read.

The value AutoFixture might give here would be if we had a `Person` class with many fields that needed to have sensible values that we wanted to pass to the `NameJoiner`. We could have AutoFixture create the `Person`, re-write the properties for `firstName` and `lastName`, and then hard-code out expected return.
