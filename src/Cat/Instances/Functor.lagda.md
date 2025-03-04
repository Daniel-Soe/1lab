```agda
open import 1Lab.Path.Cartesian

open import Cat.Instances.Product
open import Cat.Univalent using (is-category)
open import Cat.Prelude

import Cat.Functor.Reasoning as Fr
import Cat.Reasoning

open Precategory
open Functor
open _=>_

module Cat.Instances.Functor where

private variable
  o h o₁ h₁ o₂ h₂ : Level
  B C D E : Precategory o h
  F G : Functor C D
```

# Functor (pre)categories

By assigning the identity morphism to every object in $C$, we get a
natural transformation between $F : C \to D$ and itself.

```agda
idnt : {F : Functor C D} → F => F
idnt {C = C} {D = D} .η x = D .id
idnt {C = C} {D = D} .is-natural x y f = D .idl _ ∙ sym (D .idr _)
```

Given two natural transformations $\eta : F \To G$ and $\theta :
G \To H$, we can show that the assignment $x \mapsto \eta_x
\circ \theta_x$ of "componentwise compositions" defines a natural
transformation $F \To H$, which serves as the _composite_ of
$\eta$ and $\theta$.

```agda
_∘nt_ : {F G H : Functor C D} → G => H → F => G → F => H
_∘nt_ {C = C} {D = D} {F} {G} {H} f g = nat
  module ∘nt where
    module D = Cat.Reasoning D

    nat : F => H
    nat .η x = f .η _ D.∘ g .η _
```

<!--
```agda
    nat .is-natural x y h =
      (f .η y D.∘ g .η y) D.∘ F.₁ h  ≡⟨ D.pullr (g .is-natural _ _ _) ⟩
      f .η y D.∘ (G.₁ h D.∘ g .η x)  ≡⟨ D.extendl (f .is-natural _ _ _) ⟩
      H.₁ h  D.∘ f .η _ D.∘ g .η  _  ∎
      where
        module C = Precategory C
        module F = Functor F
        module G = Functor G
        module H = Functor H
```
-->

<!--
```agda
infixr 40 _∘nt_
{-# DISPLAY ∘nt.nat f g = f ∘nt g #-}
```
-->


We can then show that these definitions assemble into a category where
the objects are functors $F, G : C \to D$, and the morphisms are natural
transformations $F \To G$. This is because natural
transformations inherit the identity and associativity laws from the
codomain category $D$, and since hom-sets are sets, `is-natural`{.Agda}
does not matter.

```agda
module _ {o₁ h₁ o₂ h₂} (C : Precategory o₁ h₁) (D : Precategory o₂ h₂) where
  open Precategory

  Cat[_,_] : Precategory (o₁ ⊔ o₂ ⊔ h₁ ⊔ h₂) (o₁ ⊔ h₁ ⊔ h₂)
  Cat[_,_] .Ob = Functor C D
  Cat[_,_] .Hom F G = F => G
  Cat[_,_] .id = idnt
  Cat[_,_] ._∘_ = _∘nt_
```

All of the properties that make up a `Precategory`{.Agda} follow from
the characterisation of equalities in natural transformations: They `are
a set`{.Agda ident=Nat-is-set}, and equality of the components
`determines`{.Agda ident=Nat-path} equality of the transformation.

```agda
  Cat[_,_] .Hom-set F G = Nat-is-set
  Cat[_,_] .idr f = Nat-path λ x → D .idr _
  Cat[_,_] .idl f = Nat-path λ x → D .idl _
  Cat[_,_] .assoc f g h = Nat-path λ x → D .assoc _ _ _
```

Before moving on, we prove the following lemma which characterises
equality in `Functor`{.Agda} (between definitionally equal categories).
Given an identification $p : F_0(x) \equiv G_0(x)$ between the object
mappings, and an identification of morphism parts that lies over $p$, we
can identify the functors $F \equiv G$.

```agda
Functor-path : {F G : Functor C D}
         → (p0 : ∀ x → F₀ F x ≡ F₀ G x)
         → (p1 : ∀ {x y} (f : C .Hom x y)
               → PathP (λ i → D .Hom (p0 x i) (p0 y i)) (F₁ F f) (F₁ G f))
         → F ≡ G
Functor-path p0 p1 i .F₀ x = p0 x i
Functor-path p0 p1 i .F₁ f = p1 f i
```

<!--
```agda
Functor-path {C = C} {D = D} {F = F} {G = G} p0 p1 i .F-id =
  is-prop→pathp (λ j → D .Hom-set _ _ (p1 (C .id) j) (D .id))
    (F-id F) (F-id G) i
Functor-path {C = C} {D = D} {F = F} {G = G} p0 p1 i .F-∘ f g =
  is-prop→pathp (λ i → D .Hom-set _ _ (p1 (C ._∘_ f g) i) (D ._∘_ (p1 f i) (p1 g i)))
    (F-∘ F f g) (F-∘ G f g) i

Functor-pathp
  : {C : I → Precategory o h} {D : I → Precategory o₁ h₁}
    {F : Functor (C i0) (D i0)} {G : Functor (C i1) (D i1)}
  → (p0 : ∀ (p : ∀ i → C i .Ob) → PathP (λ i → D i .Ob) (F₀ F (p i0)) (F₀ G (p i1)))
  → (p1 : ∀ {x y : ∀ i → _}
        → (r : ∀ i → C i .Hom (x i) (y i))
        → PathP (λ i → D i .Hom (p0 x i) (p0 y i))
                (F₁ F (r i0)) (F₁ G (r i1)))
  → PathP (λ i → Functor (C i) (D i)) F G
Functor-pathp {C = C} {D} {F} {G} p0 p1 = fn where
  cob : I → Type _
  cob = λ i → C i .Ob

  exth
    : ∀ i j (x y : C i .Ob) (f : C i .Hom x y)
    → C i .Hom (coe cob i i x) (coe cob i i y)
  exth i j x y f =
    comp (λ j → C i .Hom (coei→i cob i x (~ j ∨ i)) (coei→i cob i y (~ j ∨ i)))
    ((~ i ∧ ~ j) ∨ (i ∧ j))
    λ where
      k (k = i0) → f
      k (i = i0) (j = i0) → f
      k (i = i1) (j = i1) → f

  actm
    : ∀ i (x y : C i .Ob) f
    → D i .Hom (p0 (λ j → coe cob i j x) i) (p0 (λ j → coe cob i j y) i)
  actm i x y f =
    p1 {λ j → coe cob i j x} {λ j → coe cob i j y}
      (λ j → coe (λ j → C j .Hom (coe cob i j x) (coe cob i j y)) i j (exth i j x y f))
      i

  fn : PathP (λ i → Functor (C i) (D i)) F G
  fn i .F₀ x =
    p0 (λ j → coe cob i j x)
      i
  fn i .F₁ {x} {y} f = actm i x y f
  fn i .F-id {x} =
    hcomp (∂ i) λ where
      j (i = i0) → D i .Hom-set (F .F₀ x) (F .F₀ x) (F .F₁ (C i .id)) (D i .id) base (F .F-id) j
      j (i = i1) → D i .Hom-set (G .F₀ x) (G .F₀ x) (G .F₁ (C i .id)) (D i .id) base (G .F-id) j
      j (j = i0) → base
    where
      base = coe0→i (λ i → (x : C i .Ob) → actm i x x (C i .id) ≡ D i .id) i
        (λ _ → F .F-id) x
  fn i .F-∘ {x} {y} {z} f g =
    hcomp (∂ i) λ where
      j (i = i0) → D i .Hom-set (F .F₀ x) (F .F₀ z) _ _ base (F .F-∘ f g) j
      j (i = i1) → D i .Hom-set (G .F₀ x) (G .F₀ z) _ _ base (G .F-∘ f g) j
      j (j = i0) → base
    where
      base = coe0→i (λ i → (x y z : C i .Ob) (f : C i .Hom y z) (g : C i .Hom x y)
                         → actm i x z (C i ._∘_ f g)
                         ≡ D i ._∘_ (actm i y z f) (actm i x y g)) i
        (λ _ _ _ → F .F-∘) x y z f g
```
-->

## Functor categories

When the codomain category $D$ is [univalent], then so is the category
of functors $[C,D]$. Essentially, this can be read as saying that
"naturally isomorphic functors are identified". We begin by proving that
the components of a natural isomorphism (a natural transformation with
natural inverse) are themselves isomorphisms in $D$.

[univalent]: Cat.Univalent.html

<!--
```agda
module _ {C : Precategory o h} {D : Precategory o₁ h₁} where
  import Cat.Morphism D as D
  import Cat.Morphism Cat[ C , D ] as [C,D]
```
-->

```agda
  Nat-iso→Iso : F [C,D].≅ G → ∀ x → F₀ F x D.≅ F₀ G x
  Nat-iso→Iso natiso x =
    D.make-iso (to .η x) (from .η x)
      (λ i → invl i .η x) (λ i → invr i .η x)
    where open [C,D]._≅_ natiso
```

We can now prove that $[C,D]$ `is a category`{.Agda ident=is-category},
by showing that, for a fixed functor $F : C \to D$, the space of
functors $G$ equipped with natural isomorphisms $F \cong G$ is
contractible. The centre of contraction is the straightforward part: We
have the canonical choice of $(F, id)$.

<!--
```agda
module _ {C : Precategory o₁ h₁} {D : Precategory o₂ h₂} where
  import Cat.Reasoning Cat[ C , D ] as [C,D]
  import Cat.Reasoning D as D
  open [C,D]
```
-->

```agda
  Functor-is-category : is-category D → is-category Cat[ C , D ]
```

The hard part is showing that, given some other functor $G : C \to D$
with a natural isomorphism $F \cong G$, we can give a continuous
deformation $p : G \equiv F$, such that, over this $p$, the given
isomorphism looks like the identity.

```agda
  Functor-is-category DisCat = functor-cat where
```

The first thing we must note is that we can recover the components of a
natural isomorphism while passing to/from paths in $D$. Since $D$ is a
category, `path→iso`{.Agda} is an equivalence; The lemmas we need then
follow from `equivalences having sections`{.Agda ident=iso→path→iso}.

```agda
    open Cat.Univalent.Univalent DisCat
      using (iso→path ; iso→path→iso ; path→iso→path ; Hom-pathp-iso ; Hom-pathp-reflr-iso)

    module _ {F G} (F≅G : _) where
      ptoi-to
        : ∀ x → path→iso (iso→path (Nat-iso→Iso F≅G _)) .D._≅_.to ≡ F≅G .to .η x
      ptoi-to x = ap (λ e → e .D._≅_.to) (iso→path→iso (Nat-iso→Iso F≅G x))

      ptoi-from : ∀ x → path→iso (iso→path (Nat-iso→Iso F≅G _)) .D._≅_.from
                ≡ F≅G .from .η x
      ptoi-from x = ap (λ e → e .D._≅_.from) (iso→path→iso (Nat-iso→Iso F≅G x))
```

We can then show that the natural isomorphism $F \cong G$ induces a
homotopy between the object parts of $F$ and $G$:

```agda
      F₀≡G₀ : ∀ x → F₀ F x ≡ F₀ G x
      F₀≡G₀ x = iso→path (Nat-iso→Iso F≅G x)
```

A slightly annoying calculation tells us that pre/post composition with
$F \cong G$ does in fact turn $F_1(f)$ into $G_1(f)$; This is because $F
\cong G$ is natural, so we can push it "past" the morphism part of $F$
so that the two halves of the isomorphism annihilate.

```agda
      F₁≡G₁ : ∀ {x y} (f : C .Hom x y)
            → PathP (λ i → D.Hom (F₀≡G₀ x i) (F₀≡G₀ y i)) (F .F₁ {x} {y} f) (G .F₁ {x} {y} f)
      F₁≡G₁ {x = x} {y} f = Hom-pathp-iso $
        (D.extendl (F≅G .to .is-natural x y f) ∙ D.elimr (F≅G .invl ηₚ x))

      F≡G : F ≡ G
      F≡G = Functor-path F₀≡G₀ λ f → F₁≡G₁ f
```

Putting these homotopies together defines a path `F≡G`{.Agda}. It
remains to show that, over this path, the natural isomorphism we started
with is homotopic to the identity; Equality of `isomorphisms`{.Agda
ident=≅-pathp} and `natural transformations`{.Agda ident=Nat-pathp} are
both tested componentwise, so we can "push down" the relevant equalities
to the level of families of morphisms; By computation, all we have to
show is that $\eta{}_x \circ \id \circ \id = f$.

```agda
      id≡F≅G : PathP (λ i → F ≅ F≡G i) id-iso F≅G
      id≡F≅G = ≅-pathp refl F≡G $ Nat-pathp refl F≡G λ x →
        Hom-pathp-reflr-iso (D.idr _)

    functor-cat : is-category Cat[ C , D ]
    functor-cat .to-path = F≡G
    functor-cat .to-path-over = id≡F≅G
```

A useful lemma is that if you have a natural transformation where each
component is an isomorphism, the evident inverse transformation is
natural too, thus defining an inverse to `Nat-iso→Iso`{.Agda} defined
above.

```agda
module _ {C : Precategory o h} {D : Precategory o₁ h₁} {F G : Functor C D} where
  import Cat.Reasoning D as D
  import Cat.Reasoning Cat[ C , D ] as [C,D]
  private
    module F = Functor F
    module G = Functor G

  open D.is-invertible

  componentwise-invertible→invertible
    : (eta : F => G)
    → (∀ x → D.is-invertible (eta .η x))
    → [C,D].is-invertible eta
  componentwise-invertible→invertible eta invs = are-invs where
    module eta = _=>_ eta

    eps : G => F
    eps .η x = invs x .inv
    eps .is-natural x y f =
      invs y .inv D.∘ ⌜ G.₁ f ⌝                             ≡⟨ ap! (sym (D.idr _) ∙ ap (G.₁ f D.∘_) (sym (invs x .invl))) ⟩
      invs y .inv D.∘ ⌜ G.₁ f D.∘ eta.η x D.∘ invs x .inv ⌝ ≡⟨ ap! (D.extendl (sym (eta.is-natural _ _ _))) ⟩
      invs y .inv D.∘ eta.η y D.∘ F.₁ f D.∘ invs x .inv     ≡⟨ D.cancell (invs y .invr) ⟩
      F.₁ f D.∘ invs x .inv ∎

    are-invs : [C,D].is-invertible eta
    are-invs =
      record
        { inv      = eps
        ; inverses =
          record
            { invl = Nat-path λ x → invs x .invl
            ; invr = Nat-path λ x → invs x .invr
            }
        }
```

# Currying

There is an equivalence between the spaces of bifunctors $\cC \times \cD
\to E$ and the space of functors $\cC \to [\cD,E]$. We refer to the
image of a functor under this equivalence as its _exponential
transpose_, and we refer to the map in the "forwards" direction (as in
the text above) as _currying_:

```agda
Curry : Functor (C ×ᶜ D) E → Functor C Cat[ D , E ]
Curry {C = C} {D = D} {E = E} F = curried where
  open import Cat.Functor.Bifunctor {C = C} {D = D} {E = E} F

  curried : Functor C Cat[ D , E ]
  curried .F₀ = Right
  curried .F₁ x→y = NT (λ f → first x→y) λ x y f →
       sym (F-∘ F _ _)
    ·· ap (F₁ F) (Σ-pathp (C .idr _ ∙ sym (C .idl _)) (D .idl _ ∙ sym (D .idr _)))
    ·· F-∘ F _ _
  curried .F-id = Nat-path λ x → F-id F
  curried .F-∘ f g = Nat-path λ x →
    ap (λ x → F₁ F (_ , x)) (sym (D .idl _)) ∙ F-∘ F _ _

Uncurry : Functor C Cat[ D , E ] → Functor (C ×ᶜ D) E
Uncurry {C = C} {D = D} {E = E} F = uncurried where
  import Cat.Reasoning C as C
  import Cat.Reasoning D as D
  import Cat.Reasoning E as E
  module F = Functor F

  uncurried : Functor (C ×ᶜ D) E
  uncurried .F₀ (c , d) = F₀ (F.₀ c) d
  uncurried .F₁ (f , g) = F.₁ f .η _ E.∘ F₁ (F.₀ _) g

  uncurried .F-id {x = x , y} = path where abstract
    path : E ._∘_ (F.₁ (C .id) .η y) (F₁ (F.₀ x) (D .id)) ≡ E .id
    path =
      F.₁ C.id .η y E.∘ F₁ (F.₀ x) D.id ≡⟨ E.elimr (F-id (F.₀ x)) ⟩
      F.₁ C.id .η y                     ≡⟨ (λ i → F.F-id i .η y) ⟩
      E.id                              ∎

  uncurried .F-∘ (f , g) (f′ , g′) = path where abstract
    path : uncurried .F₁ (f C.∘ f′ , g D.∘ g′)
         ≡ uncurried .F₁ (f , g) E.∘ uncurried .F₁ (f′ , g′)
    path =
      F.₁ (f C.∘ f′) .η _ E.∘ F₁ (F.₀ _) (g D.∘ g′)                       ≡˘⟨ E.pulll (λ i → F.F-∘ f f′ (~ i) .η _) ⟩
      F.₁ f .η _ E.∘ F.₁ f′ .η _ E.∘ ⌜ F₁ (F.₀ _) (g D.∘ g′) ⌝            ≡⟨ ap! (F-∘ (F.₀ _) _ _) ⟩
      F.₁ f .η _ E.∘ F.₁ f′ .η _ E.∘ F₁ (F.₀ _) g E.∘ F₁ (F.₀ _) g′       ≡⟨ cat! E ⟩
      F.₁ f .η _ E.∘ ⌜ F.₁ f′ .η _ E.∘ F₁ (F.₀ _) g ⌝ E.∘ F₁ (F.₀ _) g′   ≡⟨ ap! (F.₁ f′ .is-natural _ _ _) ⟩
      F.₁ f .η _ E.∘ (F₁ (F.₀ _) g E.∘ F.₁ f′ .η _) E.∘ F₁ (F.₀ _) g′     ≡⟨ cat! E ⟩
      ((F.₁ f .η _ E.∘ F₁ (F.₀ _) g) E.∘ (F.₁ f′ .η _ E.∘ F₁ (F.₀ _) g′)) ∎
```

<!--
```agda
PSh : ∀ κ {o ℓ} → Precategory o ℓ → Precategory _ _
PSh κ C = Cat[ C ^op , Sets κ ]

F∘-assoc
  : ∀ {o ℓ o′ ℓ′ o′′ ℓ′′ o₃ ℓ₃}
      {C : Precategory o ℓ} {D : Precategory o′ ℓ′} {E : Precategory o′′ ℓ′′} {F : Precategory o₃ ℓ₃}
      {F : Functor E F} {G : Functor D E} {H : Functor C D}
  → F F∘ (G F∘ H) ≡ (F F∘ G) F∘ H
F∘-assoc = Functor-path (λ x → refl) λ x → refl

F∘-idl
  : ∀ {o′′ ℓ′′ o₃ ℓ₃}
      {E : Precategory o′′ ℓ′′} {E′ : Precategory o₃ ℓ₃}
      {F : Functor E E′}
  → Id F∘ F ≡ F
F∘-idl = Functor-path (λ x → refl) λ x → refl

F∘-idr
  : ∀ {o′′ ℓ′′ o₃ ℓ₃}
      {E : Precategory o′′ ℓ′′} {E′ : Precategory o₃ ℓ₃}
      {F : Functor E E′}
  → F F∘ Id ≡ F
F∘-idr = Functor-path (λ x → refl) λ x → refl

module
  _ {o ℓ o′ ℓ′ o′′ ℓ′′}
    {C : Precategory o ℓ} {D : Precategory o′ ℓ′} {E : Precategory o′′ ℓ′′}
  where
    private
      module DE = Cat.Reasoning Cat[ D , E ]
      module CE = Cat.Reasoning Cat[ C , E ]

    F∘-iso-l : {F F′ : Functor D E} {G : Functor C D}
             → F DE.≅ F′ → (F F∘ G) CE.≅ (F′ F∘ G)
    F∘-iso-l {F} {F′} {G} isom =
      CE.make-iso to from
        (Nat-path λ x → isom.invl ηₚ _)
        (Nat-path λ x → isom.invr ηₚ _)
      where
        module isom = DE._≅_ isom
        to : (F F∘ G) => (F′ F∘ G)
        to .η _ = isom.to .η _
        to .is-natural _ _ _ = isom.to .is-natural _ _ _

        from : (F′ F∘ G) => (F F∘ G)
        from .η _ = isom.from .η _
        from .is-natural _ _ _ = isom.from .is-natural _ _ _

module
  _ {o ℓ o′ ℓ′}
    {C : Precategory o ℓ} {D : Precategory o′ ℓ′}
  where

  private
    module DD = Cat.Reasoning Cat[ D , D ]
    module CD = Cat.Reasoning Cat[ C , D ]
    module D = Cat.Reasoning D
    module C = Cat.Reasoning C

  natural-inverses : {F G : Functor C D} → F => G → G => F → Type _
  natural-inverses = CD.Inverses

  is-natural-invertible : {F G : Functor C D} → F => G → Type _
  is-natural-invertible = CD.is-invertible

  natural-iso : (F G : Functor C D) → Type _
  natural-iso F G = F CD.≅ G

  module natural-inverses {F G : Functor C D} {α : F => G} {β : G => F} (inv : natural-inverses α β) =
    CD.Inverses inv
  module is-natural-invertible {F G : Functor C D} {α : F => G} (inv : is-natural-invertible α) =
    CD.is-invertible inv
  module natural-iso {F G : Functor C D} (eta : natural-iso F G) = CD._≅_ eta

  idni : natural-iso F F
  idni = CD.id-iso

  _ni∘_ : ∀ {F G H : Functor C D}
          → natural-iso F G → natural-iso G H
          → natural-iso F H
  _ni∘_ = CD._∘Iso_

  _ni⁻¹ : ∀ {F G : Functor C D} → natural-iso F G → natural-iso G F
  _ni⁻¹ = CD._Iso⁻¹

  F∘-iso-id-l
    : {F : Functor D D} {G : Functor C D}
    → F DD.≅ Id → (F F∘ G) CD.≅ G
  F∘-iso-id-l {F} {G} isom = subst ((F F∘ G) CD.≅_) F∘-idl (F∘-iso-l isom)


  record make-natural-iso (F G : Functor C D) : Type (o ⊔ ℓ ⊔ ℓ′) where
    no-eta-equality
    field
      eta : ∀ x → D.Hom (F .F₀ x) (G .F₀ x)
      inv : ∀ x → D.Hom (G .F₀ x) (F .F₀ x)
      eta∘inv : ∀ x → eta x D.∘ inv x ≡ D.id
      inv∘eta : ∀ x → inv x D.∘ eta x ≡ D.id
      natural : ∀ x y f → G .F₁ f D.∘ eta x ≡ eta y D.∘ F .F₁ f

  to-natural-inverses
    : {F G : Functor C D} {α : F => G} {β : G => F}
    → (∀ x → α .η x D.∘ β .η x ≡ D.id)
    → (∀ x → β .η x D.∘ α .η x ≡ D.id)
    → natural-inverses α β
  to-natural-inverses p q =
    CD.make-inverses (Nat-path p) (Nat-path q)

  to-is-natural-invertible
    : {F G : Functor C D} {α : F => G}
    → (β : G => F)
    → (∀ x → α .η x D.∘ β .η x ≡ D.id)
    → (∀ x → β .η x D.∘ α .η x ≡ D.id)
    → is-natural-invertible α
  to-is-natural-invertible β p q = CD.make-invertible β (Nat-path p) (Nat-path q)

  to-natural-iso : {F G : Functor C D} → make-natural-iso F G → F CD.≅ G
  to-natural-iso {F = F} {G = G} x = isom where
    open CD._≅_
    open CD.Inverses
    open make-natural-iso x
    module F = Functor F
    module G = Functor G

    isom : F CD.≅ G
    isom .to .η = eta
    isom .to .is-natural x y f = sym (natural _ _ _)
    isom .from .η = inv
    isom .from .is-natural x y f =
      inv y D.∘ ⌜ G.₁ f ⌝                     ≡⟨ ap! (sym (D.idr _) ∙ ap (G.₁ f D.∘_) (sym (eta∘inv x))) ⟩
      inv y D.∘ ⌜ G.₁ f D.∘ eta x D.∘ inv x ⌝ ≡⟨ ap! (D.extendl (natural _ _ _)) ⟩
      inv y D.∘ eta y D.∘ F.₁ f D.∘ inv x     ≡⟨ D.cancell (inv∘eta y) ⟩
      F.₁ f D.∘ inv x ∎
    isom .inverses .invl = Nat-path eta∘inv
    isom .inverses .invr = Nat-path inv∘eta

  natural-inverses→inverses
    : ∀ {α : F => G} {β : G => F}
    → natural-inverses α β
    → ∀ x → D.Inverses (α .η x) (β .η x)
  natural-inverses→inverses inv x =
    D.make-inverses
      (CD.Inverses.invl inv ηₚ x)
      (CD.Inverses.invr inv ηₚ x)

  is-natural-invertible→invertible
    : ∀ {α : F => G}
    → is-natural-invertible α
    → ∀ x → D.is-invertible (α .η x)
  is-natural-invertible→invertible inv x =
    D.make-invertible
      (CD.is-invertible.inv inv .η x)
      (CD.is-invertible.invl inv ηₚ x)
      (CD.is-invertible.invr inv ηₚ x)

  is-natural-invertible→natural-iso
    : ∀ {α : F => G}
    → is-natural-invertible α
    → natural-iso F G
  is-natural-invertible→natural-iso nat-inv =
    CD.invertible→iso _ nat-inv

  natural-iso→is-natural-invertible
    : (i : natural-iso F G)
    → is-natural-invertible (natural-iso.to i)
  natural-iso→is-natural-invertible i =
    CD.iso→invertible i

open _=>_

_ni^op : natural-iso F G → natural-iso (Functor.op F) (Functor.op G)
_ni^op α =
  Cat.Reasoning.make-iso _
    (_=>_.op (natural-iso.from α))
    (_=>_.op (natural-iso.to α))
    (Nat-path λ j → natural-iso.invl α ηₚ _)
    (Nat-path λ j → natural-iso.invr α ηₚ _)

module _
  {o ℓ o′ ℓ′ o₂ ℓ₂}
  {C : Precategory o ℓ}
  {D : Precategory o′ ℓ′}
  {E : Precategory o₂ ℓ₂}
  where
  private
    de = Cat[ D , E ]
    cd = Cat[ C , D ]
  open Cat.Reasoning using (to ; from)
  open Cat.Univalent

  whisker-path-left
    : ∀ {G G′ : Functor D E} {F : Functor C D}
        (ecat : is-category de)
    → (p : Cat.Reasoning._≅_ de G G′) → ∀ {x}
    → path→iso {C = E} (λ i → (Univalent.iso→path ecat p i F∘ F) .F₀ x) .to
    ≡ p .to .η (F₀ F x)
  whisker-path-left {G} {G′} {F} p =
    de.J-iso
      (λ B isom → ∀ {x} → path→iso {C = E} (λ i → F₀ (de.iso→path isom i F∘ F) x) .to ≡ isom .to .η (F₀ F x))
      λ {x} → ap (λ e → path→iso {C = E} e .to)
        (λ i j → de.iso→path-id {a = G} i j .F₀ (F₀ F x))
        ∙ transport-refl _
    where module de = Univalent p

  whisker-path-right
    : ∀ {G : Functor D E} {F F′ : Functor C D}
        (cdcat : is-category cd)
    → (p : Cat.Reasoning._≅_ cd F F′) → ∀ {x}
    → path→iso {C = E} (λ i → F₀ G (Univalent.iso→path cdcat p i .F₀ x)) .from
    ≡ G .F₁ (p .from .η x)
  whisker-path-right {G} {G′} {F} cdcat =
    cd.J-iso
      (λ B isom → ∀ {x} → path→iso {C = E} (λ i → F₀ G (cd.iso→path isom i .F₀ x)) .from ≡ G .F₁ (isom .from .η x))
      λ {x} → ap (λ e → path→iso {C = E} e .from)
        (λ i j → G .F₀ (cd.iso→path-id {a = G′} i j .F₀ x))
        ∙ transport-refl _ ∙ sym (G .F-id)
    where module cd = Univalent cdcat

module _ {o ℓ κ} {C : Precategory o ℓ} where
  open Functor
  open _=>_

  natural-iso-to-is-equiv
    : {F G : Functor C (Sets κ)}
    → (eta : natural-iso F G)
    → ∀ x → is-equiv (natural-iso.to eta .η x)
  natural-iso-to-is-equiv eta x =
    is-iso→is-equiv $
      iso (natural-iso.from eta .η x)
          (λ x i → natural-iso.invl eta i .η _ x)
          (λ x i → natural-iso.invr eta i .η _ x)

  natural-iso-from-is-equiv
    : {F G : Functor C (Sets κ)}
    → (eta : natural-iso F G)
    → ∀ x → is-equiv (natural-iso.from eta .η x)
  natural-iso-from-is-equiv eta x =
    is-iso→is-equiv $
      iso (natural-iso.to eta .η x)
          (λ x i → natural-iso.invr eta i .η _ x)
          (λ x i → natural-iso.invl eta i .η _ x)

  natural-iso→equiv
    : {F G : Functor C (Sets κ)}
    → (eta : natural-iso F G)
    → ∀ x → ∣ F .F₀ x ∣ ≃ ∣ G .F₀ x ∣
  natural-iso→equiv eta x =
    natural-iso.to eta .η x ,
    natural-iso-to-is-equiv eta x

module _ where
  open Cat.Reasoning

  -- [TODO: Reed M, 14/03/2023] Extend the coherence machinery to handle natural
  -- isos.
  ni-assoc : {F : Functor D E} {G : Functor C D} {H : Functor B C}
         → natural-iso (F F∘ G F∘ H) ((F F∘ G) F∘ H)
  ni-assoc {E = E} = to-natural-iso λ where
    .make-natural-iso.eta _ → E .id
    .make-natural-iso.inv _ → E .id
    .make-natural-iso.eta∘inv _ → E .idl _
    .make-natural-iso.inv∘eta _ → E .idl _
    .make-natural-iso.natural _ _ _ → E .idr _ ∙ sym (E .idl _)
```
-->
