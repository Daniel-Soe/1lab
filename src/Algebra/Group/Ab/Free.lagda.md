```agda
open import Algebra.Group.Ab.Abelianisation
open import Algebra.Group.Cat.Base
open import Algebra.Group.Free
open import Algebra.Group.Ab
open import Algebra.Group

open import Cat.Displayed.Univalence.Thin
open import Cat.Functor.Adjoint.Compose
open import Cat.Functor.Adjoint
open import Cat.Prelude

module Algebra.Group.Ab.Free where
```

# Free abelian groups

We have already constructed the [free group on a set] and the [free
abelian group on a group]; [Composing these adjunctions], we obtain the
free abelian group on a set.

[free group on a set]: Algebra.Group.Free.html
[free abelian group on a group]: Algebra.Group.Ab.Abelianisation.html
[Composing these adjunctions]: Cat.Functor.Adjoint.Compose.html

```agda
Free-abelian : ∀ {ℓ} → Type ℓ → Abelian-group ℓ
Free-abelian T = Abelianise (Free-Group T)
```

<!--
```agda
mutual
  Free-abelian-functor : ∀ {ℓ} → Functor (Sets ℓ) (Ab ℓ)
  Free-abelian-functor = _
```
-->

```agda
  Free-abelian⊣Forget
    : ∀ {ℓ} → Free-abelian-functor {ℓ} ⊣ Forget-structure (Group-structure ℓ) F∘ Ab↪Grp
  Free-abelian⊣Forget = LF⊣GR
    (Ml.to-left-adjoint make-free-group)
    (Ml.to-left-adjoint make-free-abelian)
```

<!--
```agda
open is-group-hom

module _ {ℓ} (T : Type ℓ) (t-set : is-set T) where
  function→free-ab-hom : (G : Abelian-group ℓ) → (T → ⌞ G ⌟) → Ab.Hom (Free-abelian T) G
  function→free-ab-hom G fn = morp where
    private module G = Abelian-group-on (G .snd)
    go₀ : Free-group T → ⌞ G ⌟
    go₀ = fold-free-group {G = G .fst , G.Abelian→Group-on} fn .hom

    go : ⌞ Free-abelian T ⌟ → ⌞ G ⌟
    go (inc x)              = go₀ x
    go (glue (a , b , c) i) = go₀ a G.* G.commutes {go₀ b} {go₀ c} i
    go (squash x y p q i j) =
      G.has-is-set (go x) (go y) (λ i → go (p i)) (λ i → go (q i)) i j

    morp : Ab.Hom (Free-abelian T) G
    morp .hom = go
    morp .preserves .pres-⋆ = Coeq-elim-prop₂ (λ x y → G.has-is-set _ _) λ x y → refl
```
-->
