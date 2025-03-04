```agda
open import Cat.Functor.Hom.Representable
open import Cat.Instances.Functor.Compose
open import Cat.Instances.Shape.Terminal
open import Cat.Instances.Sets.Complete
open import Cat.Diagram.Colimit.Base
open import Cat.Diagram.Limit.Base
open import Cat.Instances.Functor
open import Cat.Functor.Kan.Base
open import Cat.Functor.Hom
open import Cat.Prelude

import Cat.Reasoning

module Cat.Diagram.Colimit.Representable where
```

## Representability of Colimits

Since [colimits] are defined by universal property, we can also phrase
the definition in terms of an equivalence between $\hom$-functors.

[colimits]: Cat.Diagram.Colimit.Base.html

<!--
```agda
module _
  {o ℓ}
  {J : Precategory ℓ ℓ} {C : Precategory o ℓ} {Dia : Functor J C}
  where
  private
    module C = Cat.Reasoning C
    open Functor
    open _=>_
    open Corepresentation
    open Colimit
    open is-lan
```
-->

Let $\mathrm{Dia} : \cJ \to \cC$ be some diagram in $\cC$. If
$\mathrm{Dia}$ has a colimit $c$, then that means that maps **out** of
$c$ are in bijection with a product of maps $\pi_i$, subject to some
conditions.

```agda
  Lim[C[F-,=]] : Functor C (Sets ℓ)
  Lim[C[F-,=]] .F₀ c = el (Dia => Const c) Nat-is-set
  Lim[C[F-,=]] .F₁ f α = const-nt f ∘nt α
  Lim[C[F-,=]] .F-id = funext λ _ → Nat-path λ _ → C.idl _
  Lim[C[F-,=]] .F-∘ _ _ = funext λ _ → Nat-path λ _ → sym $ C.assoc _ _ _

  Hom-into-inj
    : ∀ {c : C.Ob} (eta : Dia => Const c)
    → Hom-from C c => Lim[C[F-,=]]
  Hom-into-inj eta .η x f = const-nt f ∘nt eta
  Hom-into-inj eta .is-natural x y f = funext λ g → Nat-path λ _ →
    sym $ C.assoc _ _ _

  represents→is-colimit
    : ∀ {c : C.Ob} {eta : Dia => Const c}
    → is-natural-invertible (Hom-into-inj eta)
    → is-colimit Dia c eta
  represents→is-colimit {c} {eta} nat-inv = colim where
    module nat-inv = is-natural-invertible nat-inv

    colim : is-colimit Dia c eta
    colim .σ {M} α =
      hom→⊤-natural-trans $ nat-inv.inv .η _ (idnat-constr ∘nt α)
    colim .σ-comm {M} {α} = Nat-path λ j →
      nat-inv.invl ηₚ _ $ₚ _ ηₚ j ∙ C.idl _
    colim .σ-uniq {M} {α} {σ′} q = Nat-path λ j →
      nat-inv.inv .η _ (idnat-constr ∘nt ⌜ α ⌝)                               ≡⟨ ap! q ⟩
      nat-inv.inv .η _ ⌜ idnat-constr ∘nt (σ′ ◂ !F) ∘nt cocone→unit Dia eta ⌝ ≡⟨ ap! (Nat-path λ _ → C.idl _) ⟩
      nat-inv.inv .η (M .F₀ tt) (const-nt (σ′ .η j) ∘nt eta)                  ≡⟨ nat-inv.invr ηₚ _ $ₚ _ ⟩
      σ′ .η tt ∎
```
