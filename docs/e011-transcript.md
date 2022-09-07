# New Rustacean 11 Transcript by @jeremywsherman

hello, I'm Chris Krycho, and this is the New Rustacean podcast, a fifteen to twenty-minute show about learning the Rust
programming language. This is episode 11: Once upon a…type?

The last month has been an unusual mix for the show, with a bonus episode about building community, and then a two-part
interview with Sean Griffin, which I hope you all enjoyed. I got several requests for further interviews, and I already
have a few in mind. My goal will be to release one every other month or so.

A bit of news from the Rust community: Rust 1.7 came out this week. It included a bunch of standard library
improvements, as well as the warnings for some type system soundness changes that were introduced back in 1.4 becoming
errors. Keep an eye out for that if you're upgrading. Many people will not be affected, but depending on the
particulars of your implementation details, you might.

One of the highlighted standard library changes was a pretty substantial improvement to the way hash maps in the
language behave, which can nicely result in a serious speedup in a number of scenarios. The actual change underlying
that was that you can now drop in a substitute algorithm for hashing, indicated to the compiler simply by specifying a
specific type on the generic.

As it turns out, this leads right into our discussion for today. We're diving back into the more standard fare for the
show with an in-depth discussion of one of the things that Sean and I talked about a fair bit in the interview over the
last couple episodes, and frankly which has come up in the show from the beginning: Rust's type system as a type
system.

Before we can talk about Rust's type system in particular, though, we have to answer a much more general question: What
is a type system, and what do we mean when we talk about the different kinds of type systems that [2:00] programming
languages have. We can come at this from two directions: One is in terms of the mechanics of programming and generating
some runnable output from the computer. In that sense, our type system is the way a programming language has for
representing data and functions to the compiler. We might distinguish between different type systems, then, in the
extent to which they provide more or less information, both to the computer and to the programmer in that process.

At the most basic level, there are bits, of course, ones and zeros down at the machine. But we need a way to specify
what those bits represent when we're running a program, and we need ways to tell the compiler or interpreter how the
should think about those bits.

So it may actually help to come at that same question in more abstract terms. A paper I read, which was a distillation
of an opening lecture to some PhD students, put it this way: A type system is "a system describing what things can be
constructed." I think that's helpful: It explains what we mean when we start talking to the compiler, as we said above,
we're specifying what to construct and how to construct it. For example, we might be constructing a number, or a
string, or a linked list, or a hash map. We also might be constructing a complex data structure with other data
structures embedded in it, and a hash map is just such an example ofttimes. But we also might be constructing
operations on those data types. We might, for example, be building a function which takes in two number and does
something with them, or one that takes in any type which is iterable and iterates over all its elements and prints
them; we might build a function which takes an iterable and a function and calls the function argument on every item of
that iterable, and so on. [4:00] We tell the computer what we want to construct and how to construct it.

So type systems let us specify intent about our programs. They let us tell the compiler what we mean.

Now, there is enormous variation in the kinds of type systems out there. We might think first about two major axes of
type systems, the ones that come up most commonly in discussion I think: Static versus dynamic and strong versus weak.

By "static versus dynamic" what we really mean is whether the types are known at the time of some compilation step, in
which case they're static, or only at runtime, in which case they're dynamic.

By the strong versus weak comparison we mean, how easy is it to transform, to transmute, one type into another. In a
strong type system, you have strong guarantees that an item with a given type will continue to be handled as that same
type. In a weak type system, by contrast, the language might perform implicit conversions between types.

On both of these axes, there are trade-offs. And, of course, even though these two axes aren't directly connected,
their combinations make for trade-offs, too.

Let's make this concrete in terms of languages you're probably familiar with, at least in principal.

PHP and JavaScript are both dynamically and weakly typed. Types are not known 'til runtime, and they can and will be
implicitly converted. If, for example, you add an integer to a string, the language will just transform them under the
covers so the operation makes sense and can be carried out. Whether that's exactly what you had in mind or not, that's
what will happen.

Python is dynamically and strongly typed. Again types are not known 'til runtime, but types are not implicitly
converted. If you try to add an integer and a string together in Python, you will get a type error, and the interpreter
will tell you [6:00] that addition isn't supported for the two different types. But again, that won't happen 'til
runtime, it will not happen in some compilation step.

C is statically typed but weakly typed. Its types, that is, are known at compile time, but it's trivial to coerce from
one type to another. If you declare an integer and a character array string and add them together, you'll get nonsense,
but you won't get a compiler error. Now, granted, modern compilers will usually warn you about that kind of thing, but
strictly speaking, that's not part of C, that's tooling we've put around C because C's weak typing is known to be a
problem.

Getting back to home as it were, Rust is statically and strongly typed. Its types are known at compile time, and there
is no implicit conversion between types. You can of course convert between types, but you have to explicitly define the
underlying transformation yourself, and, in most cases, you have to explicitly call that transformation in your code,
or it won't compile at all.

Thus, in the integer and string addition example, if you don't have a defined transformation from integer to string for
concatenation, or from a string to an integer for addition, Rust won't even compile it, and Rust won't compile it if
you don't make that conversion explicit.

So far so good: Type systems give us the ability to specify our intent to the computer and have that intent enforced
more or less rigorously.

Dynamically typed systems are often more convenient, especially for rapid development, as they let you just write the
code without needing to specify everything about the types upfront. In practice, of course, once you're out of the
rapid prototyping phase, you usually end up specifying something about the types, but that usually ends up in
documentation, because if, for example, your function needs an iterable, and you hand it a non-iterable item, you will
get a runtime error, [9:00] and that's a problem.

That is the attraction of the static type system, which can give you that information at compile time. But of course it
comes at the cost of making you spell everything out, which can slow you down considerably, and therefore can make
rapid prototyping much harder.

Now, historically, most statically typed languages forced you to declare types fully whenever declaring an instance of
the type: An integer i equals five, or my type, some instance of that type, equals new my type, and so on. And all that
ceremony can get old.

In languages descended from or inspired by ML, however, including OCaml, Haskell, Swift, F#, and, to our concern, Rust,
a fair bit of the pain involved with specifying types ahead of time is removed because the compiler can often infer the
types, and in far more sophisticated ways than you get even with C++'s auto or C#'s var keywords. I'm not going to
cover all the details of the Hindley–Milner type inference algorithm used by Haskell, F#, and Rust; I don't understand
it well enough for one thing, and for another, it would take quite a while even if I did. But suffice it to say it's
enormously capable. As a result, we can have strong, static typing with at least a little less of the pain we might
expect coming from other strongly, statically typed languages.

So that gives us a pretty good idea of the static versus dynamic and weak versus strong axes. There is another axis
though, and that's what we might call expressiveness: What kinds of concepts is your type system capable of
communicating? While Rust is only a little stronger than Java and probably about the same strength or, again, only a
little stronger than C#, it's more expressive than either: It can communicate things that they can't. Likewise,
Haskell's type system is even more expressive than Rust's: It has concepts like type constructors, or higher-kinded
types, which let you [10:00] do generic things with types themselves. Beyond Haskell, there are yet further things you
can express in type systems, things like so-called dependent types, where the type of something can even include the
specific values involved. And if you're not tracking with that, it's okay; suffice it to say that there's an enormous
range of expressivity, and that Rust falls toward the more expressive end of it, but not all the way at that end of it.

But of course there are trade-offs here, too: The more your type system is capable of expressing, the more
possibilities there are for getting things wrong in a variety of ways, and the more work you have to do to get the
types you actually do want in place. This is why type inference is such a big deal: If you had to fully write out every
type in a language like Rust, much less Haskell, it would be incredibly annoying.

So that's all fine in principal, but what does it look like in practice? What is something you can express in Rust that
you can't express in, say, C, or Python, or even that is just much harder to express there? Here's one example we've
covered in substantial detail already: Rust's enumerated types, which allow you to express concisely and in one place
something which is perhaps possible in C or Python or C# – you can make something that behaves roughly like a
discriminated union in any of those languages, but it is both unnatural and nearly or actually impossible to get the
guarantees around those that Rust gives you out of the box. This is why error handling in those languages ranges from
just returning an error code or using a goto in C to exception-based handling in both Python and C#. None of those
languages have integrated language-level support for returning a type which is always either a usable result or an
error with the details of the result or the error immediately available there.

Likewise, in none of them can you force the caller to deal with the possibility of an error at that site. [12:00] This
is something that is both straightforward to express in Rust, and difficult to express and impossible to enforce in
those languages. That doesn't make them bad languages; each is very good in its own way in fact. The point here is
simply that Rust's type system allows you to express a concept – the result of some operation – in the form of a type:
The Result type. Likewise, in Rust, you can tell the compiler not only that you expect a function as an argument to
another function, but what kind of function you will take as an argument. You can say, this function accepts functions
which operate on all iterable, addable types, for example, or further, this function accepts another function which
itself takes a function acting on a given type as an argument.

This kind of thing hurt my head at first, truly, but it's incredibly powerful, and unlike in, say, JavaScript, where I
first ran into these ideas, and where they're also incredibly common, here in Rust, we can know ahead of time whether
we're meeting the required interface. We can, through the type system, express our intent to the compiler, and the
compiler can then respond if we try to pass in, say, a function which doesn't correspond to the relevant type
requirements.

For one last example, one of the things we haven't talked about on the show yet, lifetimes, is both a part of Rust's
type system, and something that it is really nice that the compiler can usually infer for us because they're hard.
Having them available is an important, essential even, part of Rust's memory safety guarantees. Knowing how long some
piece of data lives lets the compiler, and specifically the borrow checker, make the right decisions about whether a
given piece of data is available or not. But if we had to specify the lifetime of every single type every place we used
it, that would make our code much noisier and much harder to write, not just because it would involve more typing, but
because we'd have to be thinking [14:00] about it constantly. I'd go so far as to say that, if we had to write
lifetimes everywhere, the benefit we get from tracking them probably wouldn't be worth the cost of using them in the
vast majority of programs. But lifetimes are also something you can express in Rust that you literally cannot express
in many other languages. They exist in those other languages, of course, but you can't necessarily inform the compiler
of your intent about them.

This is directly analogous to the way that arguments to function exist in Python. At runtime, some argument to a
function will be a string or an integer or some type you've constructed, but you have no way to tell Python that the
type must be a string or an integer or some other type you've constructed.

In a language like C++, you have to reason about lifetimes, because, like Rust, it does some deallocation of data
behind the scenes for you when it can, but in Rust you basically never have to call a destructor explicitly in code
that just uses a type, because all of those details are handled for you by lifetimes, and you can as necessary specify
the lifetimes yourself. And the compiler will tell you when there's something wrong with the lifetimes, it will just
say, hey, you need a lifetime marker on this particular type. Don't get me wrong: As we'll cover in the next few
episodes, reasoning about and informing the compiler about lifetimes is still hard. But in Rust, at least we can tell
the compiler about it, and any given piece of data in the language has a lifetime associated with it, so we can now
express, in the types themselves, something we have to deal with in C++ separate from the type system, by calling
destructors or so on.

Coming back around to our opening then, what does all of this have to do with those changes to the hash map? Well,
simply put, the type of a hash map implementation is generic, so you can specify in the type system itself how a hash
map behaves, so long as you supply it a hashing algorithm that conforms [16:00] to the specified bounds on that
generic.
