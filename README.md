# Docception

[![CircleCI](https://circleci.com/gh/evnu/docception.svg?style=svg)](https://circleci.com/gh/evnu/docception)

Run doctests on arbitrary markdown files.

## Usage

    $ mix docception files

## Disclaimer

This tool executes any Elixir doctest it encounters (think `eval`). Ensure that it does not
encounter any harmful code!

## Example

This file can also be checked with docception. The following example is run through doctest and
results in an error:

    iex> :hello
    :crash

```
$ mix docception README.md
** (ExUnit.AssertionError)

Doctest failed
code:  :hello === :crash
left:  :hello
right: :crash
```

The following example works:

    iex> :hello
    :hello

## TODOs

* [x] Clean up temporary directory where `.beam` files are stored
* [x] Do not hard-code the temporary directory
* [x] Fix "wrong indent" warnings for heredocs
* [x] Try to fix wrong line number reports (the offset is always the same!)
* [ ] Check if this also works when a doctest refers to dependencies of the project
* [ ] Document how to use it with an alias in order to simplify running it in CI
* [ ] Check if anybody is actually interested in this
* [ ] Publish

## How..?

Docception's approach is pretty simple:

1. Read in a markdown file
1. Create an [Erlang Form](http://erlang.org/doc/apps/erts/absform.html) that makes the file
   look like something that can be executed.
   1. Add a module attribute
   1. Export an `__info__/1` function which wraps `module_info/1`
1. Compile that thing into a binary
1. Store the resulting `.beam` in the filesystem to make `Code.fetch_docs/1` work
1. Call into the undocumented-and-totally-not-for-public-use `ExUnit.DocTest.__doctests__/1`
   function
1. For each of the quoted doctests, spawn a process and call let it run the quoted doctest
1. Gather the results and raise on error

So, except for calling into `__doctests__/1`, this is pretty straight forward. Note that the
implementation relies heavily on the hard work that went into `elixir_erl.erl` from Elixir itself.
Docception needs to create a binary where the `'Docs'` chunk matches such chunks as generated by
`elixir_erl.erl`.
