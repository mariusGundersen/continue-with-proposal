# Continue With statement

This proposal extends the `continue` statement, when used in a for-of loop, to be able to pass a
parameter to an `iterator.next()` method. The proposed syntax is `continue with <<expression>>`.

## Status

**Stage:** 0
**Champion:** todo

_For more information see the [TC39 proposal process](https://tc39.github.io/process-document/)._

## Authors

* Marius Gundersen (@mariusGundersen)

# Proposal

The proposal is to extend the `continue` statement to accept an expression, the result of which will be
passed in as parameter to the `next()` method of the iterator currently being iterated in the for-of loop.

Similarly, the `break` statement can be extended to be a `break with Expression` statement and the `throw`
statement can be extended to be a `throw into LabelIdentifier Expression` statement, but that is not part
of this proposal (yet).

This is based on a short discussion [in the mailing list](https://esdiscuss.org/topic/proposal-for-of-withas-a-way-to-provide-a-value-to-the-generator).

## Syntax

This proposal changes [13.8](https://tc39.github.io/ecma262/#sec-continue-statement) by adding two more rules:

```
ContinueStatement[Yield, Await]:
  ...
  continue with Expression;
  continue LabelIdentifire with Expression;
```

Newline is not permitted between `continue with`, `with Expression`, `with LabelIdentifier` or `LabelIdentifier Expression`.

## Semantics

This proposal changes [13.8.3](https://tc39.github.io/ecma262/#sec-continue-statement-runtime-semantics-evaluation):

```
ContinueStatement : continue with Expression;
  1. let value be the result of evaluating Expression.
  1. Return Completion{[[Type]]: continue, [[Value]]: value, [[Target]]: empty}.

ContinueStatement : continue LabelIdentifier with Expression;
  1. let value be the result of evaluating Expression.
  2. Let label be the StringValue of LabelIdentifier.
  3. Return Completion{[[Type]]: continue, [[Value]]: value, [[Target]]: label }.
```

TODO: Something has to be changed in [13.7.5.13](https://tc39.github.io/ecma262/#sec-runtime-semantics-forin-div-ofbodyevaluation-lhs-stmt-iterator-lhskind-labelset)
as well.

# Examples

## Simple Example

This is a very simple example, where a value is yielded by the generator and in return the for-of loop
uses `continue with` to send a value into the generator.

```js
function* generator(){
  const result = yield 'output';
  //result === 'input'
  console.log(result);
}

for(const value of generator()){
  //value === 'output';
  console.log(value);
  continue with 'input';
}
```

## Walking a tree

A real world example where `continue with` is not only useful but required is in walking a part of a tree. This
can be done recursively using a generator, like so:

```js
function* walk(tree){
  yield tree.value;
  for(const branch of tree.branches){
    yield* walk(branch);
  }
}

for(const node of walk(tree)){
  console.log(node);
}
```

This simple example will walk the whole tree, but there is no way to skip over a branch (sub-tree). Skipping
part (or most) of a tree can be useful, for example when walking the tree looking for something, or when
comparing two trees for differences.

```js
function* walk(tree){
  const skip = yield tree.value;
  if(skip === true) return;
  for(const branch of tree.branches){
    yield* walk(branch);
  }
}
```

This generator will continue to depth-first walk a tree until `next(true)` is called. This can be done manually,
but it cannot easily be done with a for-of loop. However, it can be done with the proposed `continue with true` syntax.

```js
for(const node of walk(tree)){
  console.log(node);
  if(someTest(node)) continue with true;
}
```

# TODO

The following is a high-level list of tasks to progress through each stage of the [TC39 proposal process](https://tc39.github.io/process-document/):

### Stage 1 Entrance Criteria

* [ ] Identified a "[champion][Champion]" who will advance the addition.
* [ ] [Prose][Prose] outlining the problem or need and the general shape of a solution.
* [ ] Illustrative [examples][Examples] of usage.
* [ ] ~High-level API~ _(proposal does not introduce an API)_.

### Stage 2 Entrance Criteria

* [ ] [Initial specification text][Specification].
* [ ] _Optional_. [Transpiler support][Transpiler].

### Stage 3 Entrance Criteria

* [ ] [Complete specification text][Specification].
* [ ] Designated reviewers have [signed off][Stage3ReviewerSignOff] on the current spec text.
* [ ] The ECMAScript editor has [signed off][Stage3EditorSignOff] on the current spec text.

### Stage 4 Entrance Criteria

* [ ] [Test262](https://github.com/tc39/test262) acceptance tests have been written for mainline usage scenarios and [merged][Test262PullRequest].
* [ ] Two compatible implementations which pass the acceptance tests: [\[1\]][Implementation1], [\[2\]][Implementation2].
* [ ] A [pull request][Ecma262PullRequest] has been sent to tc39/ecma262 with the integrated spec text.
* [ ] The ECMAScript editor has signed off on the [pull request][Ecma262PullRequest].

<!-- The following are shared links used throughout the README: -->

[Champion]: #status
[Prose]: #proposal
[Examples]: #examples
[Specification]: #todo
[Transpiler]: #todo
[Stage3ReviewerSignOff]: #todo
[Stage3EditorSignOff]: #todo
[Test262PullRequest]: #todo
[Implementation1]: #todo
[Implementation2]: #todo
[Ecma262PullRequest]: #todo