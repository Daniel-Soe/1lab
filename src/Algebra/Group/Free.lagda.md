```agda
open import Algebra.Group.Cat.Base
open import Algebra.Group

open import Cat.Diagram.Initial
open import Cat.Functor.Adjoint
open import Cat.Instances.Comma
open import Cat.Prelude

module Algebra.Group.Free where
```

<!--
```agda
private variable
  ℓ : Level
  A : Type ℓ
open is-group-hom
open Group-on
open Initial
open ↓Obj
open ↓Hom
```
-->

# Free Groups

We give a direct, higher-inductive constructor of the **free group
$F(A)$ on a type $A$ of generators**. While we allow the parameter to be
any type, these are best behaved in the case where $A$ is a set; In this
case, $F$ satisfies the expected universal property.

```agda
data Free-group (A : Type ℓ) : Type ℓ where
  inc : A → Free-group A
```

The higher-inductive presentation of free algebraic structures is very
direct: There is no need to describe a set of words and then quotient by
an appropriate (complicated) equivalence relation: we can define point
constructors corresponding to the operations of a group, then add in the
path constructors that make this into a group.

```agda
  _◆_ : Free-group A → Free-group A → Free-group A
  inv : Free-group A → Free-group A
  nil : Free-group A
```

We postulate precisely the data that is needed by `make-group`{.Agda}.
This is potentially more data than is needed, but constructing maps out
of `Free-group`{.Agda} is most conveniently done using the universal
property, and there, this redundancy doesn't matter.

```agda
  f-assoc : ∀ x y z → (x ◆ y) ◆ z ≡ x ◆ (y ◆ z)
  f-invl : ∀ x → inv x ◆ x ≡ nil
  f-idl  : ∀ x → nil ◆ x ≡ x
  squash : is-set (Free-group A)
```

We can package these constructors together to give a group with
underlying set `Free-group`{.Agda}. See what was meant by "precisely the
data needed by `make-group`{.Agda}"?

```agda
Free-Group : Type ℓ → Group ℓ
Free-Group A = to-group fg where
  fg : make-group (Free-group A)
  fg .make-group.group-is-set = squash
  fg .make-group.unit = nil
  fg .make-group.mul = _◆_
  fg .make-group.inv = inv
  fg .make-group.assoc = f-assoc
  fg .make-group.invl = f-invl
  fg .make-group.idl = f-idl
```

This lemma will be very useful later. It says that whenever you want to
prove a proposition by induction on `Free-group`{.Agda}, it suffices to
address the value constructors. This is because propositions
automatically respect (higher) path constructors.

```agda
Free-elim-prop
  : ∀ {ℓ} (B : Free-group A → Type ℓ)
  → (∀ x → is-prop (B x))
  → (∀ x → B (inc x))
  → (∀ x y → B x → B y → B (x ◆ y))
  → (∀ x → B x → B (inv x))
  → B nil
  → ∀ x → B x
```

<details>
<summary>
The proof of it is a direct (by which I mean repetitive) case analysis,
so I've put it in a `<details>`{.html} tag.
</summary>

```agda
Free-elim-prop B bp bi bd binv bnil = go where
  go : ∀ x → B x
  go (inc x) = bi x
  go (x ◆ y) = bd x y (go x) (go y)
  go (inv x) = binv x (go x)
  go nil = bnil
  go (f-assoc x y z i) =
    is-prop→pathp (λ i → bp (f-assoc x y z i))
      (bd (x ◆ y) z (bd x y (go x) (go y)) (go z))
      (bd x (y ◆ z) (go x) (bd y z (go y) (go z))) i
  go (f-invl x i) =
    is-prop→pathp (λ i → bp (f-invl x i)) (bd (inv x) x (binv x (go x)) (go x)) bnil i
  go (f-idl x i) = is-prop→pathp (λ i → bp (f-idl x i)) (bd nil x bnil (go x)) (go x) i
  go (squash x y p q i j) =
    is-prop→squarep (λ i j → bp (squash x y p q i j))
      (λ i → go x) (λ i → go (p i)) (λ i → go (q i)) (λ i → go y) i j
```

</details>

## Universal Property

We now prove the universal property of `Free-group`{.Agda}, or, more
specifically, of the map `inc`{.Agda}: It gives a [universal way of
mapping] from the category of sets to an object in the category of
groups, in that any map from a set to (the underlying set of) a group
factors uniquely through `inc`{.Agda}. To establish this result, we
first implement a helper function, `fold-free-group`{.Agda}, which,
given the data of where to send the generators of a free group,
determines a group homomorphism.

[universal way of mapping]: Cat.Functor.Adjoint.html#universal-morphisms

```agda
fold-free-group
  : {A : Type ℓ} {G : Group ℓ}
  → (A → ⌞ G ⌟) → Groups.Hom (Free-Group A) G
fold-free-group {A = A} {G = G , ggrp} map = total-hom go go-hom where
  module G = Group-on ggrp
```

While it might seem that there are many cases to consider when defining
the function `go`{.Agda}, for most of them, our hand is forced: For
example, we must take multiplication in the free group (the `_◆_`{.Agda}
constructor) to multiplication in the codomain.

```agda
  go : Free-group A → ∣ G ∣
  go (inc x) = map x
  go (x ◆ y) = go x G.⋆ go y
  go (inv x) = go x G.⁻¹
  go nil = G.unit
```

Since `_◆_`{.Agda} is interpreted as multiplication in $G$, it's $G$'s
associativity, identity and inverse laws that provide the cases for
`Free-group`{.Agda}'s higher constructors.

```agda
  go (f-assoc x y z i) =
    G.associative {x = go x} {y = go y} {z = go z} (~ i)
  go (f-invl x i) = G.inversel {x = go x} i
  go (f-idl x i) = G.idl {x = go x} i
  go (squash x y p q i j) =
    G.has-is-set (go x) (go y) (λ i → go (p i)) (λ i → go (q i)) i j

  open is-group-hom

  go-hom : is-group-hom _ _ go
  go-hom .pres-⋆ x y = refl
```

Now, given a set $S$, we must come up with a group $G$, with a map
$\eta : S \to U(G)$ (in $\Sets$, where $U$ is the [underlying set functor]),
such that, for any other group $H$, any map $S \to U(H)$ can be factored
uniquely as $S \xrightarrow{\eta} U(G) \to U(H)$. As hinted above, we
pick $G = \rm{Free}(S)$, the free group with $S$ as its set of
generators, and the universal map $\eta$ is in fact `inc`{.Agda}.

[underlying set functor]: Algebra.Group.Cat.Base.html#the-underlying-set

```agda
make-free-group : make-left-adjoint (Forget {ℓ})
make-free-group .Ml.free S = Free-Group ∣ S ∣
make-free-group .Ml.unit _ = inc
make-free-group .Ml.universal f = fold-free-group f
make-free-group .Ml.commutes f = refl
make-free-group .Ml.unique {y = y} {g = g} p =
  Homomorphism-path $ Free-elim-prop _ (λ _ → hlevel!)
    (p $ₚ_)
    (λ a b p q → ap₂ y._⋆_ p q ∙ sym (g .preserves .is-group-hom.pres-⋆ _ _))
    (λ a p → ap y.inverse p ∙ sym (is-group-hom.pres-inv (g .preserves)))
    (sym (is-group-hom.pres-id (g .preserves)))
  where module y = Group-on (y .snd)
module Free-groups {ℓ} = make-left-adjoint (make-free-group {ℓ = ℓ})
```
