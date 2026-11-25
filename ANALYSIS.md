# The Search and the Stack

```text
== trace P07 ==
Call: grandparent(tom,X)
  Call: parent(tom,_)
  Exit: parent(tom,liz)
  Call: parent(liz,X)
  Fail: parent(liz,X)
  Redo: parent(tom,liz)
  Exit: parent(tom,bob)
  Call: parent(bob,X)
  Exit: parent(bob,ann)
Exit: grandparent(tom,ann)
answer 1: X=ann
Redo: grandparent(tom,ann)
  Redo: parent(bob,ann)
  Exit: parent(bob,pat)
Exit: grandparent(tom,pat)
answer 2: X=pat
Redo: grandparent(tom,pat)
  Redo: parent(bob,pat)
  Fail: parent(bob,X)
  Redo: parent(tom,bob)
  Fail: parent(tom,_)
Fail: grandparent(tom,X)
== 2 answers ==
```

## Pushing and popping frames

Every `Call` line pushes a frame onto Activity 15's runtime stack, and every
`Exit` and `Fail` line pops one. In this trace, `Call: grandparent(tom,X)`
pushes the first frame. `Call: parent(tom,_)` pushes on top of it, which is
why the line is indented further in. The indentation shows the stack depth.
`Exit: parent(tom,liz)` pops that frame and hands the binding `liz` back to
`grandparent`. `Fail: parent(liz,X)` pops a frame and returns nothing,
because liz has no children. A `Redo` line neither pushes a new call nor pops
an existing one. It goes back into a frame that has already returned, more on
that in the second question. The last line, `Fail: grandparent(tom,X)`, pops
the last frame and leaves the stack empty.

## What a `Redo` resumes

A `Redo` resumes a goal that has already exited and continues from the clause
after the one it used. An ordinary call stack cannot do this. Once a function
returns, its frame is gone, and calling it again starts from the top. Prolog
instead keeps track of the goals with untried clauses. After answer 1,
`Redo: grandparent(tom,ann)` goes back into `grandparent` even though it had
already exited with `X=ann`. Then `Redo: parent(bob,ann)` moves on to the next
`parent(bob, _)` fact, which gives `pat`. The same thing happens earlier: when
`parent(liz,X)` fails, `Redo: parent(tom,liz)` goes back to the first goal and
tries tom's next child, bob. When a resumed goal has no more clauses left, as
in `Fail: parent(bob,X)`, the search backs up one more level.

## Swapping the clauses of `reach/2` and `ancestor/2`

Swapping the two clauses of `reach/2` changes whether Prolog ever reaches an
answer. In `cases/kb.pl` the recursive clause comes first, and its body starts
with `reach(X,Z)`. Prolog calls `reach/2` again with the same `X` before it
reads a single edge, so each call opens another identical call and the search
never gets to `edge/2` or the base clause. That is why P11 stops at 400 lines
with no answer. With the base clause first, every level tries `edge(X,Y)`
before going deeper, so `reach(a,d)` answers on the way down. The left
recursion is still there, though, so asking for more answers loops forever.
Swapping the clauses of `ancestor/2` only changes the order of the answers,
because both of its clauses read `parent/2` before calling `ancestor/2` again.
Each recursive call is one generation further down, and the family has no
cycles, so the search always runs out of facts and ends, as P08 and P09 show.
