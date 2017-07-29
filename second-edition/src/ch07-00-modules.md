# Using Modules to Reuse and Organize Code

> NOTE: The text you are reading is a reimagining of this chapter as if the
> [proposal to revisit Rust's module system](http://aturon.github.io/blog/2017/07/26/revisiting-rusts-modules/)
> is accepted. THIS DOES NOT DESCRIBE CURRENT RUST!
>
> Assumptions I am making about this possible future:
>
> - src/main.rs will still be the default entry point for binary crates
> - src/lib.rs will still be the default entry point for library crates

When you start writing programs in Rust, your code might live solely in the
`main` function. As your code grows, you’ll eventually move functionality into
other functions for reuse and better organization. By splitting your code into
smaller chunks, each chunk is easier to understand on its own. But what happens
if you have too many functions? Rust has a module system that enables the reuse
of code in an organized fashion.

In the same way that you extract lines of code into a function, you can extract
functions (and other code, like structs and enums) into different modules. A
*module* is a namespace that contains definitions of functions or types, and
you can choose whether those definitions are visible outside of the current
crate, or only visible within the current crate. Here’s an overview of how
modules work:

* Inline modules can be declared with the `mod` keyword.
* Creating a new file or directory in the *src* directory creates a new module
  with the name of that file or directory. These modules are public to external
  users of your crate.
* If a file or directory in the *src* directory has a name that starts with an
  underscore, like *_util.rs* or *_util/* for example, this module will only be
  visible within the current crate.
* Files named *lib.rs* do not create a new module. That is, code within
  *src/lib.rs* is at the top level of the namespace hierarchy, and code in a
  file named *src/inner/lib.rs* would be within the `inner` module rather than
  an `inner::lib` module.
* By default, functions, types, and constants within all modules are private.
  The `pub` keyword makes an item public and therefore visible outside its
  namespace.

We’ll look at each of these parts to see how they fit into the whole.
