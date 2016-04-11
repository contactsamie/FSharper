# FSharper

Directly from http://www.mindscapehq.com/blog/index.php/2012/03/27/5-12-f-features-every-c-programmer-should-lust-after/

1. The Option type
Tony Hoare, the inventor of the null reference, called it his ‘billion dollar mistake.’ Null is a disaster because it means that every reference type has a magic value that will destroy your program. For value types, we now have ‘normal’ value types and nullable value types to express the difference between, say, ‘this function returns an integer’ and ‘this function might return an integer, or it might not.’ But we don’t have that for reference types.
F# provides the Option type to distinguish between ‘there definitely is a value’ and ‘there might or might not be a value.’ F#’s Option is a bit like Nullable but can be used with any type, value or reference, and comes equipped with helper functions and pattern matching support. Here’s an example of Option in use:
let names = [ "alice"; "bob"; "carol"]
let name = List.tryFind (fun s -> length s < 4) names  // returns an Option
match name with
| Some n -> printfn "%s has a short name" n
| None   -> printfn "They all have long names"
Notice that if we erroneously try to use the option returned from List.tryFind as if it were a real string, the error is caught at compile time:
let names = [ "alice"; "bob"; "carol"]
let name = List.tryFind (fun s -> length s < 4) names  // returns an Option
printfn "%s has a short name" name  // Error! Expected string but got string option
In C#, the ‘not found’ case would be represented by null, and the compiler would not be able to catch the potential error.
The FSharpx project provides some helpers to make it easy to use the F# Option type from C#, though it’s of limited use except in interop because the C# compiler still won’t stop you using nulls even in non-optional situations.
2. Immutable record types
Immutable types are a great way to improve the correctness of programs. Any type that represents a value, rather than an entity which needs to preserve identity as its attributes change, should be immutable. Even quite complex objects, such as expression trees in LINQ or syntax trees in Roslyn, benefit from immutability. Unfortunately, C# makes it far more effort to write immutable types than mutable ones. Here’s two ways to implement a 2D Point type in C#, one mutable (wrong!) and one immutable (right!):
public class MutablePoint {
  public double X { get; set; }
  public double Y { get; set; }
}
 
public class ImmutablePoint {
  private readonly double _x;
  private readonly double _x;
 
  public ImmutablePoint(double x, double y) {
    _x = x;
    _y = y;
  }
 
  public double X { get { return _x; } }
  public double Y { get { return _y; } }
}
The safe, immutable type is more than twice as long!
Contrast this with F#:
type Point = { X : float; Y : float }
It’s short, it’s immutable and as a special wonder bonus it’s got proper value equality built in for free. And I want it in C#, pronto.
3. Object expressions
Object expressions are a great feature for when you want to implement an interface or override a class member without going to the trouble of spinning out an entire class or subclass for it. This can dramatically reduce the amount of code particularly when you need to capture local variables for your implementation or override. (If you’re familiar with Java’s local classes, object expressions are a lot like that.) Here’s an example using the LightSpeed IAuditInfoStrategy interface:
// blame takes a string and returns an IAuditInfoStrategy
let blame x = {
  new IAuditInfoStrategy with
    member this.Mode = AuditInfoMode.Custom
    member this.GetCurrentUser() = x }
In C#, I would have had to write out a ScapegoatingAuditInfoStrategy class with a field to hold the blamee and a constructor to pass the blamee into the class. In F#, I just told the compiler to make me an IAuditInfoStrategy that always blamed x, and it did. I often end up writing lots of little classes to capture small nuggets of code or fragments of state, and it would be great if C# had something like object expressions to make it easier.
More about object expressions here.
4. Partial application
I actually like C# lambda syntax more than F#. This is just as well, because I have to write a lot more lambdas in C# than I do in F#. The reason is that in F# I can use a trick called partial application to reuse a ‘normal’ function just with particular arguments. Let’s see an example.
Suppose I want to double every element in a sequence. I can do that using the LINQ Select operator, or its F# equivalent Seq.map:
// C#
var doubled = values.Select(v => v * 2);
// F#
let doubled = Seq.map (fun v -> v * 2) values
C# code is more concise than F# code, right? Not so fast! In F# we can partially apply the * function and save ourselves writing the lambda!
let doubled = Seq.map ((*) 2) values
Of course you can do this with your own functions too:
let isDivisibleBy x y = y % x = 0
let evens = List.filter (isDivisibleBy 2) values  // rather than (fun v -> isDivisibleBy 2 v)
Partial application reduces the amount of lambda noise in the code, and can also be useful in creating new functions by specialising existing ones. You can read more about partial application in this series and (more concisely) in this article.
5. Pattern matching
I’ve written extensively about pattern matching elsewhere, and it’s too big a topic to explain thoroughly here. Pattern matching allows you to combine programming by cases (like a switch statement) with custom logic (like an if statement) and decomposing composites to get the bits you’re interested in (like a series of property accesses or collection operations). It’s also extensible using active patterns, which means you can build classifiers independently of the routines that use the classification. And patterns compose far more conveniently than imperative code.
One often-overlooked but very powerful feature of pattern matching is that it can be used in a let statement. This allows you to effectively return multiple values from a function without having to create a XxxResult type or explicitly picking apart a tuple. Combined with the F# compiler being able to silently translate C# out-parameters into tupled return values, this creates some neat, readable idioms:
let dict = new Dictionary<string, string>()
let (found, result) = dict.TryGetValue("alice")
 
// C# equivalent
var dict = new Dictionary<string, string>();
string result = null;
bool found = dict.TryGetResult("alice", out result);
There’s much more to pattern matching than this, including recursing over lists, visiting over class hierarchies (discriminated unions), working safely with options, type-checking and casting, and, well, the list goes on. I use pattern matching all the time in F# and I always miss it when I have to come back to C# and write imperative (if-style) code to distinguish between cases.
5 1/2. Async
F#’s async workflows make it easy to write asynchronous code in a readable, easy to understand, top-to-bottom way. This is compelling — so compelling, in fact, that a similar feature is going to be adopted into C#. F# still has a bunch of async features that haven’t made it into C#, notably async agents, but for the core scenario C# programmers no longer need to be envious!
And there’s more
There’s so much in F# that would be a huge boon for C# that I could easily have made this a top 10 list. Some of F#’s features can be used from C# (read Mauricio Scheffer’s great post on ’10 reasons to use the F# runtime in your C# app’) but a lot of the features I miss are part of the language. For example, I’d love to see F#-level type inference in C#. After all, which one of these declarations do you find easier to read?
// C#
public static IEnumerable<TResult> Map<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector) { ... }
// F#
let map selector source = ...  // F# infers types and generic type parameters from implementation
Or how about recursive iterators? Or an interactive prompt in Visual Studio, so you could try out snippets or run demos without having to build a console or NUnit project? (Roslyn has a C# Interactive window, but Roslyn’s still some way off!) And looking to the future, F# type providers open up a whole new order of expressiveness beyond standard code generation techniques.
