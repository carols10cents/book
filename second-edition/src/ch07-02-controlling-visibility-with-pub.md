## Controlling Visibility with `pub`

At this point, `cargo build` can compile our project, but we still get warning
messages about the `client::connect`, `network::connect`, and
`network::server::connect` functions not being used:

```text
warning: function is never used: `connect`, #[warn(dead_code)] on by default
src/client.rs:1:1
  |
1 | fn connect() {
  | ^

warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/network.rs:1:1
  |
1 | fn connect() {
  | ^

warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/network/server.rs:1:1
  |
1 | fn connect() {
  | ^
```

So why are we receiving these warnings? After all, we’re building a library
with functions that are intended to be used by our *users*, not necessarily by
us within our own project, so it shouldn’t matter that these `connect`
functions go unused. The point of creating them is that they will be used by
another project, not our own.

To understand why this program invokes these warnings, let’s try using the
`connect` library from another project, calling it externally. To do that,
we’ll create a binary crate in the same directory as our library crate by
making a *src/bin/main.rs* file containing this code:

<!-- I'm also assuming https://github.com/rust-lang/rfcs/pull/2088 gets
accepted -->

<span class="filename">Filename: src/bin/main.rs</span>

```rust,ignore
fn main() {
    ::communicator::client::connect();
}
```

Our package now contains *two* crates. Cargo treats *src/bin/main.rs* as the
root file of a binary crate, which is separate from the existing library crate
whose root file is *src/lib.rs*. This pattern is quite common for executable
projects: most functionality is in a library crate, and the binary crate uses
that library crate. As a result, other programs can also use the library crate,
and it’s a nice separation of concerns.

From the point of view of a crate outside the `communicator` library looking
in, all the modules we’ve been creating are within a module that has the same
name as the crate, `communicator`. We call the top-level module of a crate the
*root module*.

Right now, our binary crate just calls our library’s `connect` function from
the `client` module. However, invoking `cargo build` will now give us an error
after the warnings:

```text
error: module `client` is not exported
 --> src/main.rs:4:5
  |
4 |     ::communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Ah ha! This error tells us that the `client` module hasn't been exported, which
is the crux of the warnings. It’s also the first time we’ve run into the
concept of *exported*. The default state of modules is public: they're visible
anywhere within the current crate but not by code outside it. The default state
of other items is private: only the current module may use the code. We saw in
the last section how we could make an item visible to the rest of the crate
using `pub`. If you don’t use a private or crate-public function within your
program, because your crate is the only code allowed to use that function,
Rust will warn you that the function has gone unused.

After we specify that the `client` module and the `client::connect` function
should be exported, not only will our call to that function from our binary
crate be allowed, but the warning that the function is unused will go away.
Marking a function as `export` lets Rust know that the function will be used by
code outside of our program. Rust considers the theoretical external usage
that’s now possible as the function “being used.” Thus, when something is
marked with the `export` keyword, Rust will not require that it be used in our
program and will stop warning that the item is unused.

### Exporting Modules and Functions

To tell Rust to export something, we add the `export` keyword to the start of
the declaration of the item we want to make public. We’ll focus on fixing the
warning that indicates `client::connect` has gone unused for now, as well as
the `` module `client` is not exported `` error from our binary crate.

Because the `client` module has been declared implicitly by creating
*src/client.rs*, and because modules are public to the crate by default, we
need to specify that we want to export the `client` module in the crate root.
Modify *src/lib.rs* to mark the `client` module with `export`, like so:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
export client;
```

We put the `export` keyword and then the name of the module. Let’s try building
again:

```text
error: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     ::communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Hooray! We have a different error! Yes, different error messages are a cause
for celebration. The new error shows `` function `connect` is private ``, so
let’s edit *src/client.rs* to make `client::connect` exported too:

<span class="filename">Filename: src/client.rs</span>

```rust
export fn connect() {
}
```

Now run `cargo build` again:

```text
warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/network.rs:1:1
  |
1 | fn connect() {
  | ^

warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/network/server.rs:1:1
  |
1 | fn connect() {
  | ^
```

The code compiled, and the warning about `client::connect` not being used is
gone!

Unused code warnings don’t always indicate that an item in your code needs to
be exported: if you *didn’t* want these functions to be part of your external
API, unused code warnings could be alerting you to code you no longer need that
you can safely delete. They could also be alerting you to a bug if you had just
accidentally removed all places within your library where this function is
called.

But in this case, we *do* want the other two functions to be part of our
crate’s public API, so let’s mark them as `pub` as well to get rid of the
remaining warnings. Modify *src/network/mod.rs* to look like the following:

<span class="filename">Filename: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Then compile the code:

```text
warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/network/mod.rs:1:1
  |
1 | pub fn connect() {
  | ^

warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/network/server.rs:1:1
  |
1 | fn connect() {
  | ^
```

Hmmm, we’re still getting an unused function warning, even though
`network::connect` is set to `pub`. The reason is that the function is public
within the module, but the `network` module that the function resides in is not
public. We’re working from the interior of the library out this time, whereas
with `client::connect` we worked from the outside in. We need to change
*src/lib.rs* to make `network` public too, like so:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Now when we compile, that warning is gone:

```text
warning: function is never used: `connect`, #[warn(dead_code)] on by default
 --> src/network/server.rs:1:1
  |
1 | fn connect() {
  | ^
```

Only one warning is left! Try to fix this one on your own!

<!-- I might want to introduce a section here that demonstrates the error you'd
get if you export a function in a module without exporting the module -->

### Privacy Rules

Overall, these are the rules for item visibility:

1. If an item is public, it can be accessed through any of the modules
   in the current crate.
2. If an item is private, it can be accessed only by its immediate parent
   module and any of the parent’s child modules.
3. If an item is exported, and all of its parent modules are exported, it can
   be accessed by code outside of the current crate.

### Privacy Examples

Let’s look at a few more privacy examples to get some practice. Create a new
library project and enter the code in Listing 7-5 into your new project’s
*src/lib.rs*:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    crate::outermost::middle_function();
    crate::outermost::middle_secret_function();
    crate::outermost::inside::inner_function();
    crate::outermost::inside::secret_function();
}
```

<span class="caption">Listing 7-5: Examples of private and public functions,
some of which are incorrect</span>

Before you try to compile this code, make a guess about which lines in the
`try_me` function will have errors. Then, try compiling the code to see whether
you were right, and read on for the discussion of the errors!

#### Looking at the Errors

The `try_me` function is in the root module of our project. The module named
`outermost` is public to the crate because that's the default for modules.
Therefore, the first privacy rule says the `try_me` function is allowed to
access the `outermost` module because `outermost` is defined in the same crate
as `try_me`.

<!-- insert explanation of `crate` here, that `crate` is a special identifier
for the current crate /Carol -->

The call to `outermost::middle_function` will work because `middle_function` is
public, and `try_me` is accessing `middle_function` from the same crate.

The call to `outermost::middle_secret_function` will cause a compilation error.
`middle_secret_function` is private, so the second rule applies. The root
module is neither the current module of `middle_secret_function` (`outermost`
is), nor is it a child module of the current module of `middle_secret_function`.

<!-- This example now behaves the same; I'd probably change it to show an
`extern` error instead /Carol -->

The module named `inside` is public. That means the `try_me` function
is allowed to call `outermost::inside::inner_function` but not
`outermost::inside::secret_function`.

#### Fixing the Errors

Here are some suggestions for changing the code in an attempt to fix the
errors. Before you try each one, make a guess as to whether it will fix the
errors, and then compile the code to see whether or not you’re right, using the
privacy rules to understand why.

* What if the `inside` module was exported?
* What if `outermost` was exported and `inside` was not?
* What if, in the body of `inner_function`, you called
  `crate::outermost::middle_secret_function()`?
  
<!-- I'm not sure these make as much sense anymore, I feel like we don't need
this section as much because the system isn't as surprising so we don't have to
spend as much effort building up the reader's intuition! -->

Feel free to design more experiments and try them out!

Next, let’s talk about bringing items into scope with the `use` keyword.
