# Pamplemousse
Pamplemousse is a proof assistant for a propositional logic based on Gentzen-style proofs.

## Getting Started

### Installation
Pamplemousse is bundled as a [Python](https://www.python.org) package, and requires Python version 3.8 or higher to run.
It can be installed via `pip`, either by cloning this repository locally and then running 
```
$ pip install /path/to/repository
```
or by installing directly from GitHub:
```
$ pip install git+https://github.com/raghav198/pamplemousse
```
This will install the `pamplemousse` executable.

### Writing Proofs
A Pamplemousse proof is an ASCII text file.
The first line of the file is a comma-separated list of formulae that specifies your proof obligations, and each subsequent line is a single proof step.

A proof step consists of an explicit line number, the formula being proved in that step, and finally a justification (the inference rule being used, and an optional comma-separated list of line numbers for arguments).
All lines must end with a semicolon.

A formula is a well-formed expression built out of of propositions (denoted by capital letters) and the following logical connectives:
* Conjunction (`/\`)
* Disjunction (`\/`)
* Implication (`->`)
* Negation (`~`)

To enter a hypothetical world, preface the line number with a vertical line: `|`.
An example proof of the exportation property is shown below:
```
A -> (B -> C)
1. A /\ B -> C prem;
| 2. A hyp;
| | 3. B hyp;
| | 4. A /\ B conj 2, 3;
| | 5. C mp 1, 4;
| 6. B -> C ded 3-5;
7. A -> (B -> C) ded 2, 6;
```
Note that the order of arguments to an inference rule matters! For example, proof checking would fail if line 5 instead read `| | 5. C mp 4, 1;`

A full list of inference rules and their semantics can be found [below](#inference-rules-reference).
For more example proofs, see [examples/](/examples/).

### Checking Proofs
Once you have written a Gentzen-style proof and saved it to a text file, the `pamplemousse` checker can automatically verify that your proof is correct:

```
$ pamplemousse /path/to/proof.txt
```

Pamplemousse will record the proof obligation and then check your proof line by line, attempting to unify the formula on each line with the inference rule written as the justification.

If unification fails for a line, Pamplemousse will print out an error detailing what went wrong and exit.
Once all lines have been successfully verified, Pamplemousse checks the list of formulas proven against the proof obligations, failing if any proof obligations have not been met.

## Inference Rules Reference
The tables below present all the inference rules available to Pamplemousse.
The inference rules are written using type variables (`a`, `b`, `c`).
These type variables can be substituted for _any well-formed formula_, however, the same formula must be substitued for _every instance_ of the same type variable in the same row of the table. 

Remember that the order of the arguments matters, so e.g. the Modus Ponens rule `mp` will fail if you give it the proof of the antecedent before the proof of the implication.

### Directional Inference Rules

|Name | Given | Conclude|
|-----|------|---|
| `mp` | `a -> b`, `a` | `b`|
| `mt` | `a -> b`, `~b` | `~a` |
| `simpl` | `a /\ b` | `a` or `b` |
| `add` | `a` | `a \/ b` or `b \/ a` |
| `hs` | `a -> b`, `b -> c` | `a -> c`|
| `ds` | (`a \/ b` or `b \/ a`), `~a` | `b`|
| `de` | `a \/ b`, `a -> c`, `b -> c` | `c` |
| `hyp`, `prem` | | `a` |
| `conj` | `a`, `b` | `a /\ b` |
| `disj` | `a`, `b` | `a \/ b` |

There is also the deduction rule: `ded` which takes a list of line numbers corresponding to the lines of the "hypothetical world" proof.
Note that in the case of nested hypothetical worlds, the line numbers of the inner world do not also belong to the other world.
For example, the exportation proof presented [above](#writing-proofs) would fail if line 7 instead read
```
7. A -> (B -> C) ded 2-6
```

### Equivalence Rules
The semantics of the equivalence rules differ from those of the directional inference rules in two ways:
1. Equivalence rules can be applied backwards (i.e. they can be used to construct the pattern in either the `Left` or `Right` column, given the pattern in the other column)
2. Each pattern only needs to match a subformula, unlike the patterns above, which need to match the _entire_ formula to be applied.

Notice that common rules like `self`, `dm`, and `comm`/`assoc` have specialized versions for `and` and `or`.

|Name | Left | Right |
|-----|------|---|
| `or_comm` | `a \/ b`| `b \/ a`|
| `and_comm` | `a /\ b` | `b /\ a` |
| `or_assoc` | `a \/ (b \/ c)` | `(a \/ b) \/ c` |
| `and_assoc` | `a /\ (b /\ c)` | `(a /\ b) /\ c` |
| `dn`| `a` | `~~a` |
| `imp` | `a -> b` | `~a \/ b` |
| `dist_ao` | `a /\ (b \/ c)` | `(a /\ b) \/ (a /\ c)` |
| `dist_oa` | `a \/ (b /\ c)` | `(a \/ b) /\ (a \/ c)` |
| `dm_ao` | `~(a /\ b)` | `~a \/ ~b` |
| `dm_oa`| `~(a \/ b)` | `~a /\ ~b` |
| `exp` | `a -> (b -> c)` | `(a /\ b) -> c` |
| `cp` | `a -> b` | `~b -> ~a` |
| `self_or` | `a \/ a` | `a` |
| `self_and` | `a /\ a` | `a` |