```agda
{-# OPTIONS -vtc.def:10 #-}
open import Cat.Univalent.Instances.Opposite
open import Cat.Diagram.Colimit.Base
open import Cat.Diagram.Limit.Base
open import Cat.Functor.Kan.Unique
open import Cat.Instances.Elements
open import Cat.Instances.Functor
open import Cat.Diagram.Terminal
open import Cat.Morphism.Duality
open import Cat.Instances.Sets
open import Cat.Functor.Base
open import Cat.Functor.Hom
open import Cat.Prelude

import Cat.Reasoning

module Cat.Functor.Hom.Representable {o κ} {C : Precategory o κ} where
```

<!--
```agda
private
  module C = Cat.Reasoning C
  module C^ = Cat.Reasoning Cat[ C ^op , Sets κ ]
  module [C,Sets] = Cat.Reasoning Cat[ C , Sets κ ]
  module Sets = Cat.Reasoning (Sets κ)
open Element-hom
open Functor
open Element
open _=>_
```
-->

# Representable functors

A functor $F : \cC\op \to \Sets_\kappa$ (from a [locally
$\kappa$-small category][univ]) is said to be **representable** when it
is [naturally isomorphic][niso] to $\hom(-, X)$ for an object $X :
\cC$ (called the **representing object**) — that is, the functor $F$
classifies the _maps into_ $X$. Note that we can evidently dualise the
definition, to get what is called a **corepresentable functor**, one of
the form $\hom(X, -)$, but we refer informally to both of these
situations as "representables" and "representing objects".

[univ]: 1Lab.intro.html#universes-and-size-issues
[niso]: Cat.Instances.Functor.html#functor-categories

```agda
record Representation (F : Functor (C ^op) (Sets κ)) : Type (o ⊔ κ) where
  no-eta-equality
  field
    rep        : C.Ob
    represents : natural-iso F (Hom-into C rep)

  module rep = natural-iso represents

  equiv : ∀ {a} → C.Hom a rep ≃ ∣ F .F₀ a ∣
  equiv = Iso→Equiv λ where
    .fst                → rep.from .η _
    .snd .is-iso.inv    → rep.to .η _
    .snd .is-iso.rinv x → rep.invr ηₚ _ $ₚ x
    .snd .is-iso.linv x → rep.invl ηₚ _ $ₚ x

  module Rep {a} = Equiv (equiv {a})

open Representation
open Representation using (module Rep) public
```

This definition is _deceptively_ simple: the idea of representable
functor (and of representing object) is _key_ to understanding the idea
of **universal property**, which could be called the most important
concept in category theory. Most constructions in category theory
specified in terms of the existence of certain maps are really instances
of representing objects for functors: [limits], [colimits], [coends],
[adjoint functors], [Kan extensions], etc.

[limits]: Cat.Diagram.Limit.Base.html
[coends]: Cat.Diagram.Coend.html
[colimits]: Cat.Diagram.Colimit.Base.html
[Kan extensions]: Cat.Functor.Kan.Base.html
[adjoint functors]: Cat.Functor.Adjoint.html

The first thing we will observe is an immediate consequence of the
[Yoneda lemma]: representing objects are unique. Intuitively this is
because "$X$ is a representation of $F$" determines how $X$ reacts to
being mapped into, and since the only thing we can probe objects in an
arbitrary category by are morphisms, two objects which react to
morphisms in the same way must be isomorphic.

[Yoneda lemma]: Cat.Functor.Hom.html

```agda
representation-unique : {F : Functor (C ^op) (Sets κ)} (X Y : Representation F)
                      → X .rep C.≅ Y .rep
representation-unique X Y =
  is-ff→essentially-injective {F = よ C} (よ-is-fully-faithful C) よX≅よY where
    よX≅よY : よ₀ C (X .rep) C^.≅ よ₀ C (Y .rep)
    よX≅よY = (X .represents C^.Iso⁻¹) C^.∘Iso Y .represents
```

Therefore, if $\cC$ is a univalent category, then the type of
representations for a functor $F$ is a proposition. This does not follow
immediately from the lemma above: we also need to show that the
isomorphism computed by the full-faithfulness of the Yoneda embedding
commutes with the specified representation isomorphism.
This follows by construction, but the proof needs to commute

applications of functors and paths-from-isos, which is never pretty:

```agda
Representation-is-prop : ∀ {F} → is-category C → is-prop (Representation F)
Representation-is-prop {F = F} c-cat x y = path where
  module X = Representation x
  module Y = Representation y

  objs : X.rep ≡ Y.rep
  objs = c-cat .to-path (representation-unique x y)

  path : x ≡ y
  path i .rep = objs i
  path i .represents =
    C^.≅-pathp refl (ap (よ₀ C) objs) {f = X.represents} {g = Y.represents}
      (Nat-pathp _ _ λ a → Hom-pathp-reflr (Sets _)
        {A = F .F₀ a} {q = λ i → el! (C.Hom a (objs i))}
        (funext λ x →
           ap (λ e → e .Sets.to) (ap-F₀-iso c-cat (Hom-from C a) _) $ₚ _
        ·· sym (Y.rep.to .is-natural _ _ _) $ₚ _
        ·· ap Y.Rep.from (sym (X.rep.from .is-natural _ _ _ $ₚ _)
                       ·· ap X.Rep.to (C.idl _)
                       ·· X.Rep.ε _)))
     i
```

## As terminal objects

We begin to connect the idea of representing objects to other universal
constructions by proving this alternative characterisation of
representations: A functor $F$ is representable if, and only if, its
[category of elements](Cat.Instances.Elements.html) $\int F$ has a
[terminal object](Cat.Diagram.Terminal.html).

```agda
terminal-element→representation
  : {F : Functor (C ^op) (Sets κ)} → Terminal (∫ C F) → Representation F
terminal-element→representation {F} term = f-rep where
  module F = Functor F
  open Terminal term
```

From the terminal object in $\int F$^[Which, recall, consists of an
object $X : \cC$ and a section $F(X) : \Sets$], we obtain a natural
transformation $\eta_y : F(y) \to \hom(y,X)$, given componentwise by
interpreting each pair $(y, s)$ as an object of $\int F$, then taking
the terminating morphism $(y, s) \to (X, F(X))$, which satisfies (by
definition) $F(!)(F(X)) = s$. This natural transformation is
componentwise invertible, as the calculation below shows, so it
constitutes a natural isomorphism.

```agda
  nat : F => よ₀ C (top .ob)
  nat .η ob section = has⊤ (elem ob section) .centre .hom
  nat .is-natural x y f = funext λ sect → ap hom $ has⊤ _ .paths $ elem-hom _ $
    F.₁ (has⊤ _ .centre .hom C.∘ f) (top .section)   ≡⟨ happly (F.F-∘ _ _) _ ⟩
    F.₁ f (F.₁ (has⊤ _ .centre .hom) (top .section)) ≡⟨ ap (F.₁ f) (has⊤ _ .centre .commute) ⟩
    F.₁ f sect                                       ∎

  inv : ∀ x → Sets.is-invertible (nat .η x)
  inv x = Sets.make-invertible
    (λ f → F.₁ f (top .section))
    (funext λ x → ap hom $ has⊤ _ .paths (elem-hom x refl))
    (funext λ x → has⊤ _ .centre .commute)

  f-rep : Representation F
  f-rep .rep = top .ob
  f-rep .represents = C^.invertible→iso nat $
    componentwise-invertible→invertible nat inv
```

## Universal constructions

We now show a partial converse to the calculation above: That terminal
objects are representing objects for a particular functor. Consider, to
be more specific, the constant functor $F : \cC\op \to \Sets$ which
sends everything to the terminal set. When is $F$ representable?

Well, unfolding the definition, it's when we have an object $X : \cC$
with a natural isomorphism $\hom(-,X) \cong F$. Unfolding _that_, it's
an object $X$ for which, given any other object $Y$, we have an
isomorphism of sets $\hom(Y,X) \cong \{*\}$^[which varies naturally in
$Y$, but this naturality is not used in this simple case]. Hence, a
representing object for the "constantly $\{*\}$" functor is precisely a
terminal object. It turns out the

```agda
representable-unit→terminal
  : Representation (Const (el (Lift _ ⊤) (hlevel 2))) → Terminal C
representable-unit→terminal repr .Terminal.top = repr .rep
representable-unit→terminal repr .Terminal.has⊤ ob = retract→is-contr
  (Rep.from repr) (λ _ → lift tt) (Rep.η repr) (hlevel 0)
```

## Corepresentable functors

As noted earlier, we can dualise the definition of a representable
functor to the covariant setting to get **corepresentable** functors.

```agda
record Corepresentation (F : Functor C (Sets κ)) : Type (o ⊔ κ) where
  no-eta-equality
  field
    corep : C.Ob
    corepresents : natural-iso F (Hom-from C corep)

  module corep = natural-iso corepresents

  coequiv : ∀ {a} → C.Hom corep a ≃ ∣ F .F₀ a ∣
  coequiv = Iso→Equiv λ where
    .fst → corep.from .η _
    .snd .is-iso.inv → corep.to .η _
    .snd .is-iso.rinv x → corep.invr ηₚ _ $ₚ x
    .snd .is-iso.linv x → corep.invl ηₚ _ $ₚ x

  module Corep {a} = Equiv (coequiv {a})

open Corepresentation
open Corepresentation using (module Corep) public
```

Much like their contravariant cousins, corepresenting objects are unique up to
isomorphism.

```agda
corepresentation-unique
  : {F : Functor C (Sets κ)} (X Y : Corepresentation F)
  → X .corep C.≅ Y .corep
```

<details>
<summary>We omit the proof, as it is identical to the representable case.
</summary>

```agda
corepresentation-unique X Y =
  is-ff→essentially-injective {F = Functor.op (よcov C)}
    (よcov-is-fully-faithful C)
    (iso→co-iso (Cat[ C , Sets κ ]) ni)
  where
    ni : natural-iso (Hom-from C (Y .corep)) (Hom-from C (X .corep))
    ni = (Y .corepresents ni⁻¹) ni∘ X .corepresents
```
</details>

This implies that the type of corepresentations is a proposition when
$\cC$ is univalent.

```agda
Corepresentation-is-prop : ∀ {F} → is-category C → is-prop (Corepresentation F)
```

<details>
<summary>We opt to not show the proof, as it is even nastier than the
proof for representables due to the fact that the yoneda embedding
for covariant functors is itself contravariant.
</summary>

```agda
Corepresentation-is-prop {F = F} c-cat X Y = path where

  objs : X .corep ≡ Y .corep
  objs = c-cat .to-path (corepresentation-unique X Y)

  path : X ≡ Y
  path i .corep = objs i
  path i .corepresents =
    [C,Sets].≅-pathp refl (ap (Hom-from C) objs)
       {f = X .corepresents} {g = Y .corepresents}
       (Nat-pathp _ _ λ a → Hom-pathp-reflr (Sets _)
         {A = F .F₀ a} {q = λ i → el! (C.Hom (objs i) a)}
         (funext λ x →
           ap (λ e → e .Sets.to) (ap-F₀-iso (opposite-is-category c-cat) (Hom-into C a) _) $ₚ _
           ·· sym (corep.to Y .is-natural _ _ _ $ₚ _)
           ·· ap (Corep.from Y) (sym (corep.from X .is-natural _ _ _ $ₚ _)
                                 ·· ap (Corep.to X) (C.idr _)
                                 ·· Corep.ε X _)))
       i
```
</details>

## Corepresentable functors preserve limits

A useful fact about corepresentable functors is that they preserve
all limits. To show this, we first need to show that the covariant
hom functor $\cC(x,-)$ preserves limits.

To get an intuition for why this is true, consider how the
functor $\cC(x,-)$ behaves on products. The set of morphisms
$\cC(x,a \times b)$ is equivalent to the set $\cC(x, a) \times \cC(x, b)$
of pairs of morphisms (See [`product-repr`] for a proof of this
equivalence).

[`product-repr`]: Cat.Diagram.Product.html#product-repr

```agda
Hom-from-preserves-limits
  : ∀ {o′ κ′}
  → (c : C.Ob)
  → is-continuous o′ κ′ (Hom-from C c)
Hom-from-preserves-limits c {Diagram = Dia} {K} {eps} lim =
  to-is-limitp ml (funext λ _ → refl) where
  open make-is-limit
  module lim = is-limit lim

  ml : make-is-limit _ _
  ml .ψ j f = lim.ψ j C.∘ f
  ml .commutes f = funext λ g →
    C.pulll (sym (eps .is-natural _ _ _))
    ∙ (C.elimr (K .F-id) C.⟩∘⟨refl)
  ml .universal eta p x =
    lim.universal (λ j → eta j x) (λ f → p f $ₚ x)
  ml .factors _ _ = funext λ _ →
    lim.factors _ _
  ml .unique eps p other q = funext λ x →
    lim.unique _ _ _ λ j → q j $ₚ x
```

Preservation of limits by corepresentable functors then follows from
a general fact about functors: if $F$ preserves limits, and $F$ is
naturally isomorphic to $F'$, then $F'$ must also preserve limits.

```agda
corepresentable-preserves-limits
  : ∀ {o′ κ′} {F}
  → Corepresentation F
  → is-continuous o′ κ′ F
corepresentable-preserves-limits F-corep lim =
   natural-iso→preserves-limits
     (F-corep .corepresents ni⁻¹)
     (Hom-from-preserves-limits (F-corep .corep))
     lim
```

We can show a similar fact for representable functors, but with a twist:
they **reverse** colimits! This is due to the fact that a representable
functor $F : \cC\op \to \Sets$ is contravariant. Specifically, $F$ will
take limits in $\cC\op$ to limits in $\Sets$, but limits in $\cC\op$
are colimits, so $F$ will take colimits in $\cC$ to limits in $\Sets$.

A less formal perspective on this is that the collection of maps
out of a colimit is still defined as a limit in $\Sets$. For instance,
to give a $a + b \to x$ out of a coproduct, we are required to give
a pair of maps $a \to x$ and $b \to x$.

```agda
よ-reverses-colimits
  : ∀ {o′ κ′}
  → (c : C.Ob)
  → is-cocontinuous o′ κ′ (Functor.op (よ₀ C c))
よ-reverses-colimits c {Diagram = Dia} {K} {eta} colim =
  to-is-colimitp mc (funext λ _ → refl) where
  open make-is-colimit
  module colim = is-colimit colim

  mc : make-is-colimit _ _
  mc .ψ j f = f C.∘ colim.ψ j
  mc .commutes f = funext λ g →
    C.pullr (eta .is-natural _ _ _)
    ∙ (C.refl⟩∘⟨ C.eliml (K .F-id))
  mc .universal eps p x =
    colim.universal (λ j → eps j x) (λ f → p f $ₚ x)
  mc .factors eps p = funext λ _ →
    colim.factors _ _
  mc .unique eps p other q = funext λ x →
    colim.unique _ _ _ λ j → q j $ₚ x

representable-reverses-colimits
  : ∀ {o′ κ′} {F}
  → Representation F
  → is-cocontinuous o′ κ′ (Functor.op F)
representable-reverses-colimits F-rep colim =
  natural-iso→preserves-colimits
    ((F-rep .represents ni^op) ni⁻¹)
    (よ-reverses-colimits (F-rep .rep))
    colim
```
