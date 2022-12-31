# Glacier · Gleam Incremental Interactive Unit Testing

![Under construction](./resources/glacier-logo.png)

[![Package Version](https://img.shields.io/hexpm/v/glacier)](https://hex.pm/packages/glacier)
[![Hex Docs](https://img.shields.io/badge/hex-docs-ffaff3)](https://hexdocs.pm/glacier/)

## Introduction

**Glacier** brings incremental interactive unit testing to [Gleam](https://gleam.run).
It is meant as a drop-in replacement for [Gleeunit](https://hexdocs.pm/gleeunit) and it relies on it, internally.

*Glacier* differs from *Gleeunit* insofar, that it let's you:

1. Pass in an interactive flag `gleam test -- --glacier`; save a module and only related tests will rerun.
2. Pass in a specific unit test modules to rerun `gleam test -- test/my_module_test.gleam`.
3. If `gleam test` is passed without any `--`-arguments it behaves the same as *Gleeunit*.
4. You can still pass in `--target erlang` or `--target javascript`, like so: `gleam test --target erlang -- --glacier` or like `gleam test --target javascript -- test/my_module_test.gleam`.

To enable this behavior, all you have to do is add *Glacier* as a dev dependency, aka `gleam add glacier`, open `./test/YOUR_PROJECT.gleam` and replace `gleeunit.main()` with `glacier.main()`.

## How does it work?

1. `gleam test` passes through `glacier.run()` and simply executes `gleeunit.main()` as if *Gleeunit* was used directly.
2. `gleam test -- test_module_a test_module_b` passes through `glacier.run()` and executes `gleeunit.test_modules(modules_list)` where `modules_list` is `["foo", "bar"]`. The given modules are checked if they exist as either `.gleam` or `.erl` test module files and then *Gleeunit* runs these test modules.
3. `gleam test -- --glacier` enters `glacier.run()` and starts a file watcher: Upon changes in module files in `./test` it just passes those through as `gleam test -- changed_test_module`(so re-saving test files executes the single test), and if a module file in `./src` got changed it parses that changed module file for any imported modules and puts the module and all chained imported modules in a distinct list of modules that should be tested. Then all test module files are read and imports of those are gathered one by one and cross matched against that list. The result is a list of test modules that need to be run, which then gets done by executing a shell call similar to `gleam test -- detected_test_module_a detected_test_module_b detected_test_module_c etc`, aka jumps to `2.`.

## TODO

- Erlang: Introduce some delay between the file watcher picking up a change and the test running, so that if you find-replace-save-all via an editor it does not try to run the tests 10 times. This requires some medium large changes as the code to detect imports and dependent imports needs to run for 1..n modules and if test modules are affected it needs to be handled separately (added afterwards distinct, unique).
- Handling of JavaScript src and test modules for DENO: <https://deno.land/api@v1.29.1?s=Deno.watchFs>.
- Save import lists per module into an in-memory hash-table. Before a file gets parsed for its imports, build the hash and check if a cached version already exists.
- Use set instead of lists in some places?
- Once gleam does only recompile changed modules: Do not test dependencies if they had been tested recently, aka the first time a module it saved all its dependencies are tested but afterwards only if they changed by keeping a cache table with the module name and an mtime it last run the tests for those modules.
- Backport gleeunit changes.
- Handling of Elixir src and test modules.

## Known Caveats

- With *Glacier* you can only run interactive incremental test against a single target, Erlang or JavaScript, at a time.

## Installation & Usage

For Erlang this library depends on [fs](https://hexdocs.pm/fs/). Because of this on Linux you will need [`inotify-tools` to be installed](https://github.com/synrc/fs#backends). On Mac and Windows it should work out of the box.

For NodeJS version 18 LTS is supported and development always and only happens on the latest stable LTS, such as Node.js say v18.12.1.

If available on Hex this package can be added to your Gleam project:

1. Run `gleam add glacier`
2. Open `./test/YOUR_PROJECT.gleam` and replace `gleeunit.main()` with `glacier.main()`
3. Run `gleam test`
   - save any test module (within `./test`) file to re-run that single test
   - save any src module (within `./src`) to run all associated tests. Associated tests are test modules where the module is imported or where any of the module's imports and their import's imports (import chain) are imported into.

... and its documentation can be found at <https://hexdocs.pm/glacier>.

## Quick start

```sh
git clone https://github.com/inoas/glacier.git
cd glacier
gleam run   # Show instructions how to use the library stand alone
gleam test  # Run the tests on this library
```

## Status

Alpha quality, thus while this is still...

![Under construction](https://web.archive.org/web/20090829023556im_/http://geocities.com/okitsugu/underconstruction.gif)

...it should be working now.
