## `mod` and the Filesystem

We’ll start our module example by making a new project with Cargo, but instead
of creating a binary crate, we’ll make a library crate: a project that other
people can pull into their projects as a dependency. For example, the `rand`
crate in Chapter 2 is a library crate that we used as a dependency in the
guessing game project.

We’ll create a skeleton of a library that provides some general networking
functionality; we’ll concentrate on the organization of the modules and
functions but we won’t worry about what code goes in the function bodies. We’ll
call our library `communicator`. By default, Cargo will create a library unless
another type of project is specified: if we omit the `--bin` option that we’ve
been using in all of the chapters preceding this one, our project will be a
library:

```text
$ cargo new communicator
$ cd communicator
```

Notice that Cargo generated *src/lib.rs* instead of *src/main.rs*. Inside
*src/lib.rs* we’ll find the following:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
    }
}
```

Cargo creates an empty test to help us get our library started, rather than the
“Hello, world!” binary that we get when we use the `--bin` option. We’ll look
at the `#[]` and `mod tests` syntax in the “Using `super` to Access a Parent
Module” section later in this chapter, but for now, leave this code at the
bottom of *src/lib.rs*.

Because we don’t have a *src/main.rs* file, there’s nothing for Cargo to
execute with the `cargo run` command. Therefore, we’ll use the `cargo build`
command to compile our library crate’s code.

We’ll look at different options for organizing your library’s code that will be
suitable in a variety of situations, depending on the intent of the code.

### Module Definitions

For our `communicator` networking library, we’ll first define an inline module
named `network` that contains the definition of a function called `connect`.
Inline module definitions are made with the `mod` keyword. Add this code to the
beginning of the *src/lib.rs* file, above the test code:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

After the `mod` keyword, we put the name of the module, `network`, and then a
block of code in curly braces. Everything inside this block is inside the
namespace `network`. In this case, we have a single function, `connect`. If we
wanted to call this function from a script outside the `network` module, we
would need to specify the module and use the namespace syntax `::`, like so:
`network::connect()` rather than just `connect()`.

We can also have multiple modules, side by side, in the same *src/lib.rs* file.
For example, to also have a `client` module that has a function named `connect`
as well, we can add it as shown in Listing 7-1:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

<span class="caption">Listing 7-1: The `network` module and the `client` module
defined side by side in *src/lib.rs*</span>

Now we have a `network::connect` function and a `client::connect` function.
These can have completely different functionality, and the function names do
not conflict with each other because they’re in different modules.

In this case, because we’re building a library, the file that serves as the
entry point for building our library is *src/lib.rs*. However, in respect to
creating modules, there’s nothing special about *src/lib.rs*. We could also
create modules in *src/main.rs* for a binary crate in the same way as we’re
creating modules in *src/lib.rs* for the library crate. In fact, we can put
modules inside of modules, which can be useful as your modules grow to keep
related functionality organized together and separate functionality apart. The
choice of how you organize your code depends on how you think about the
relationship between the parts of your code. For instance, the `client` code
and its `connect` function might make more sense to users of our library if
they were inside the `network` namespace instead, as in Listing 7-2:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-2: Moving the `client` module inside the
`network` module</span>

In your *src/lib.rs* file, replace the existing `mod network` and `mod client`
definitions with the ones in Listing 7-2, which have the `client` module as an
inner module of `network`. Now we have the functions `network::connect` and
`network::client::connect`: again, the two functions named `connect` don’t
conflict with each other because they’re in different namespaces.

In this way, modules form a hierarchy. The contents of *src/lib.rs* are at the
topmost level, and the submodules are at lower levels. Here’s what the
organization of our example in Listing 7-1 looks like when thought of as a
hierarchy:

```text
communicator
 ├── network
 └── client
```

And here’s the hierarchy corresponding to the example in Listing 7-2:

```text
communicator
 └── network
     └── client
```

The hierarchy shows that in Listing 7-2, `client` is a child of the `network`
module rather than a sibling. More complicated projects can have many modules,
and they’ll need to be organized logically in order to keep track of them. What
“logically” means in your project is up to you and depends on how you and your
library’s users think about your project’s domain. Use the techniques shown
here to create side-by-side modules and nested modules in whatever structure
you would like.

### Moving Modules to Other Files

Modules form a hierarchical structure, much like another structure in computing
that you’re used to: filesystems! We can use Rust’s module system along with
multiple files to split up Rust projects so not everything lives in
*src/lib.rs* or *src/main.rs*. For this example, let’s start with the code in
Listing 7-3:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-3: Three modules, `client`, `network`, and
`network::server`, all defined in *src/lib.rs*</span>

The file *src/lib.rs* has this module hierarchy:

```text
communicator
 ├── client
 └── network
     └── server
```

If these modules had many functions, and those functions were becoming lengthy,
it would be difficult to scroll through this file to find the code we wanted to
work with. Because the functions are nested inside one or more mod blocks, the
lines of code inside the functions will start getting lengthy as well. These
would be good reasons to separate the `client`, `network`, and `server` modules
from *src/lib.rs* and place them into their own files.

First, remove the `client` module code so that your *src/lib.rs* looks like the following:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

Now we need to create the external file with that module name. Create a
*client.rs* file in your *src/* directory and open it. Then enter the
following, which is the `connect` function in the `client` module that we
removed in the previous step:

<span class="filename">Filename: src/client.rs</span>

```rust
fn connect() {
}
```

This file provides the *contents* of the `client` module. Cargo will notice any
file in your *src* directory or subdirectories that is named with a valid Rust
identifier and has the *.rs* extension. The code in each of these files is
included within a module named with the filename.

Now the project should compile successfully, although you’ll get a few
warnings. Remember to use `cargo build` instead of `cargo run` because we have
a library crate rather than a binary crate:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)

warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/client.rs:1:1
  |
1 | fn connect() {
  | ^

warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/lib.rs:4:5
  |
4 |     fn connect() {
  |     ^

warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/lib.rs:8:9
  |
8 |         fn connect() {
  |         ^
```

These warnings tell us that we have functions that are never used. Don’t worry
about these warnings for now; we’ll address them in the “Controlling Visibility
with `pub`” section later in this chapter. The good news is that they’re just
warnings; our project built successfully!

Next, let’s extract the `network` module into its own file using the same
pattern. Delete the `network` module from *src/lib.rs*, which makes that file empty.

Then create a new *src/network.rs* file and enter the following:

<span class="filename">Filename: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Notice that we still have a `mod` declaration within this module file; this is
because we still want `server` to be an inline submodule of `network`.

Run `cargo build` again. Success! We have one more module to extract: `server`.
Because we want it to be a submodule of `network`—that is, the `server` module
is within the `network` module—we'll create a *src/network* directory and put
the *server.rs* file inside that directory rather than in *src*.

First, remove the `server` module from *src/network.rs* so that only the
`connect` function definition remains:

<span class="filename">Filename: src/network.rs</span>

```rust,ignore
fn connect() {
}
```

Then create a *src/network* directory with `mkdir src/network` and create a
*server.rs* file within *src/network*. Enter the contents of the `server`
module that we extracted:

<span class="filename">Filename: src/network/server.rs</span>

```rust
fn connect() {
}
```

Now when we try to run `cargo build`, compilation will work (we’ll still have
warnings though). Our module layout still looks like this, which is exactly the
same as it did when we had all the code in *src/lib.rs* in Listing 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

The corresponding file layout now looks like this:

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   ├── network
│   │   └── server.rs
│   └── network.rs
```

<!-- I'd probably add a section here demonstrating how to call these functions
from each other and how you have to make them `pub` to be able to use them
outside the current module, how to `use` items with `use crate::thing`. I'm
also assuming that unused `pub` code would produce the same warnings that
unused private code does today. /Carol -->

Next, we’ll talk about the `extern` keyword and get rid of those warnings!
