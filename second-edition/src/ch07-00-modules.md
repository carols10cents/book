# Using Modules to Reuse and Organize Code

When you start writing programs in Rust, your code might live solely in the
`main` function. As your code grows, you’ll eventually move functionality into
other functions for reuse and better organization. By splitting your code into
smaller chunks, each chunk is easier to understand on its own. But what happens
if you have too many functions? Rust has a module system that enables the reuse
of code in an organized fashion.

In the same way that you extract lines of code into a function, you can extract
functions (and other code, like structs and enums) into different modules. A
*module* is a namespace that contains definitions of functions or types, and
you can choose whether those definitions are part of your external API
(exported) visible to the rest of the crate outside of their module (public) or
neither (private). Here’s an overview of how modules work:

* The `mod` keyword declares a new inline module. Code within the module appears
  immediately following this declaration within curly braces.
* Modules can be defined in a separate file by naming the file with the name of
  the module and putting the definitions in that file.
* By default, modules are public to the rest of your crate.
* By default, functions, types, and constants are private to their module. The
  `pub` keyword makes an item public and therefore visible to the rest of the
  crate.
* The `export` keyword makes items part of your public API accessible by crates
  that depend on your crate. To export an item within a module, the module must
  be exported as well.
* The `use` keyword brings definitions inside modules into scope so it’s easier
  to refer to them.

We’ll look at each of these parts to see how they fit into the whole.
