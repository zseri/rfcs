---
feature: no-nested-with
start-date: 2021-01-11
author: Alain Zscheile (@zseri)
co-authors: @sternenseemann
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Disallow or discourage usage of multiple `with` expressions covering
the same expression / forbid nesting `with` expressions, even indirectly.
If infeasible in general (e.g. forbidding it in all nix expressions),
this can be limited to nixpkgs.

# Motivation
[motivation]: #motivation

It makes static analysis of nixpkgs easier, because as soon as `with`
expressions are nested, it becomes basically impossible to [statically
deduce where any free variable comes from] without implementing a
full-blown nix evaluator including lazy evaluation, which is difficult as
soon as `with` expressions and mutually recursive imports are involved
(e.g. as currently present in `nixpkgs/lib/systems/{inspect,parse}.nix`).

With this approach, any reference to any free variable can be easily
resolved to the enclosing `with` expression "scope-include", and
because this `with` expression couldn't then be enclosed by another
one, even indirectly, no lookup ambiguity exists.

# Detailed design
[design]: #detailed-design

At least warn about any nested usage of `with` expressions, at least when they get evaluated,
possibly even when they get parsed. After a grace period, abort instead.

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

```nix
pkgs: {
  # allowed
  a = with pkgs; patchelf;

  # disallowed
  b = with pkgs; with lib; patchelf;

  # also disallowed
  c = with pkgs; {
    meta = with lib; {
      license = with licenses; [ mit ];
    };
  };
}
```

# Drawbacks
[drawbacks]: #drawbacks

* It makes it necessary to modify some parts of nixpkgs.

# Alternatives
[alternatives]: #alternatives

* Introduce a kind of `with-only` expression which allows bringing an attrset
  into scope while simultaneously hiding the outer scope, such that all
  inner free variables are either resolved via the given attrset, or
  result in an error.

* Completely ban the usage of `with` in nixpkgs (this would probably result in
  too much churn, which seems excessive for this problem).

# Unresolved questions
[unresolved]: #unresolved-questions

Decide if this is enough.

e.g.
* Mutually recursive imports combined with `with` expressions also make static
  analysis harder, because they require lazyness at the level of scope lookups,
  which is difficult to implement corrently.
* Check if any use case is too much negatively impacted by this.

# Future work
[future]: #future-work

Unknown.
