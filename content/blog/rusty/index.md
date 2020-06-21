---
title: Rust fun and learnings
date: "2020-06-20T00:00:00.000Z"
---
As part of a new service that needs to be performant, I got a chance to play with Rust which I have been meaning to try for sometime. Rust thought or rather showed couple of things in action. 

### Stack > Heap

All through college and interview preperation we are ingrained with the concept of stack level objects are performant than heap level access. I worked 3 years on JVM related lanaguages and never had to bother doing these optimizations. Compiler makes those decisions for you. While working on the service, we had a common object that needs to be shared across various threads. Think of a connection pool object that needs to be shared between workers so that you can share connections among threads. There are two ways to do this:

- Using [Arc](https://doc.rust-lang.org/std/sync/struct.Arc.html) to keep count of references, allocating object during compile time onto stack and sharing it between workers/threads during runtime.
- Another way of doing the same is, using [lazy static](https://github.com/rust-lang-nursery/lazy-static.rs) and initializing object during runtime to share between threads. This way the object gets allocated onto heap, theoretically compiler does not know the size of object during compile time. There might be a way to force this onto stack, but I am not aware of it.

The point I was trying to get at is, when we profiled both approaches stack based approach is a good 50% more efficient than heap based approach. It's a good practical learning of one of those theoretical things. Rust forces you to make lot of those decisions via pointers (more on it in a section below).

### Higher level > Lower level

The way I understand rust is a systems programming language which builds on concepts of c/c++ while also making it not easy to make mistakes in this systems languages. The point I am trying to make here is not that working in rust is better than these other languages. It's more of a generic most of the times you are better off using abstractions rather than trying and squeezing performance. While building service, one of my goals was to make as performant as possible. I went down the path of taking the leanest framework possible in hopes of deriving maximum performance. I coded my way to parsing requests, serializing/deserializing and managing state of objects between workers. Although I learnt a lot in the process about rust which I do not regret a bit, the moment I switched to a higher level framework I ended up gaining at-least a good 25% performance gain. I would recommend the same for new programmers of the language. Try with basics, but when it comes time to move to production be sure to benchmark against higher level frameworks. It might be easier this way than trying other way around i.e. working in higher level frameworks and moving down.

### Functional Languages are fun

I sorely miss working in Scala. It's a heavy language combined with a compiler that frustrates you, in-spite of that there are lot of niceties. Functional paradigms that help you keep grounded and prevents lot mistakes making you defensively. There are lot of libraries that help you get into flamewars time to time e.t.c.. Some functional paradigms I got to play with while working on the service - 

- Pattern matching on errors and futures
- Closures on functions
- Error mapping

I will not bore much with details in this section. [Rust official book](https://doc.rust-lang.org/book/ch13-00-functional-features.html) is a good place to explore more about these features.

### Pointers don't have to be pita

Rust makes you to choose between pass by reference and pass by value. And have been teached that almost always good to use the method that is least privileged from the point of view of function that is using it. It is also good in most of the other languages, rust provides ways to enforce it via clippy and a community who is passionate about advocating the right ways. This also keeps the use of `null` to minimum. You are almost always encouraged to pass by reference. In this context, I would also mention borrow checker which is a great tool that helps in making sure the variables you are using are always valid. In case you are wondering, [borrow checker](https://doc.rust-lang.org/1.1.0/book/references-and-borrowing.html) is the tool that obviates the need for garbage collector.

### Compilers in other languages can be way more better

It's a welcome change to work in a compiler that care's for programmer. I have worked in atleast 5 different languages. None of them makes working with errors as fun as rust. Although it had its own quirks like for that time when I am trying to work around borrow's, an encounter with rust compiler is a chance to learn new things. Add the nicety that clippy adds to the mix, it's a joy working with rust compiler. I have been productive within couple of days and in most cases happy with my code.

### Bonus learning

You should definitely read about [musl](https://www.musl-libc.org/intro.html), we used it to build static binary that is small and packaged that binary into an alpine docker image to keep our docker image size close to 10MB in comparison to a docker image built via python for similar application is about 200MB.