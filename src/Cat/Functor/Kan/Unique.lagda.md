```agda
open import Cat.Instances.Functor.Compose
open import Cat.Instances.Functor
open import Cat.Functor.Kan.Base
open import Cat.Functor.Base
open import Cat.Prelude

import Cat.Functor.Reasoning
import Cat.Reasoning

module Cat.Functor.Kan.Unique where
```

# Uniqueness of Kan extensions

[Kan extensions] (both left and [right]) are [universal constructions],
so they are [unique when they exist]. To get a theorem out of this
intuition, we must be careful about how the structure and the properties
are separated: Informally, we refer to the _functor_ as "the Kan
extension", but in reality, the data associated with "the Kan extension
of $F$ along $p$" also includes the natural transformation. For
accuracy, using the setup from the diagram below, we should say “$(G,
\eta)$ is the Kan extension of $F$ along $p$".

[Kan extensions]: Cat.Functor.Kan.Base.html
[right]: Cat.Functor.Kan.Base.html#right-kan-extensions
[universal constructions]: Cat.Functor.Hom.Representable.html
[unique when they exist]: 1Lab.HLevel.html#is-prop

~~~{.quiver .tall-15}
\[\begin{tikzcd}
  C && D \\
  \\
  {C'}
  \arrow["F", from=1-1, to=1-3]
  \arrow["p"', from=1-1, to=3-1]
  \arrow[""{name=0, anchor=center, inner sep=0}, "G"', from=3-1, to=1-3]
  \arrow["\eta"', shorten <=6pt, Rightarrow, from=0, to=1-1]
\end{tikzcd}\]
~~~

<!--
```agda
private variable
  o ℓ : Level
  C C′ D : Precategory o ℓ

module
  Lan-unique
    {p : Functor C C′} {F : Functor C D}
    {G₁ G₂ : Functor C′ D} {η₁ η₂}
    (l₁ : is-lan p F G₁ η₁)
    (l₂ : is-lan p F G₂ η₂)
  where

  private
    module l₁ = is-lan l₁
    module l₂ = is-lan l₂
    module D = Cat.Reasoning D
    module C′D = Cat.Reasoning Cat[ C′ , D ]

  open C′D._≅_
  open C′D.Inverses
```
-->

To show uniqueness, suppose that $(G_1, \eta_1)$ and $(G_2, \eta_2)$ and
both left extensions of $F$ along $p$. Diagramming this with both
natural transformations shown is a bit of a nightmare: the option which
doesn't result in awful crossed arrows is to duplicate the span $\cC'
\ot \cC \to \cD$. So, to be clear: The upper triangle and the lower
triangle _are the same_.

~~~{.quiver}
\[\begin{tikzcd}
  && \cC \\
  \\
  {\cC'} &&&& \cD \\
  \\
  && \cC
  \arrow["F", from=1-3, to=3-5]
  \arrow["p"', from=1-3, to=3-1]
  \arrow[""{name=0, anchor=center, inner sep=0}, "{G_1}", curve={height=-12pt}, from=3-1, to=3-5]
  \arrow["p", from=5-3, to=3-1]
  \arrow["F"', from=5-3, to=3-5]
  \arrow[""{name=1, anchor=center, inner sep=0}, "{G_2}"', curve={height=12pt}, from=3-1, to=3-5]
  \arrow["{\eta_1}", shorten <=9pt, Rightarrow, from=0, to=1-3]
  \arrow["{\eta_2}"', shorten <=9pt, Rightarrow, from=1, to=5-3]
\end{tikzcd}\]
~~~

Recall that $(G_1, \eta_1)$ being a left extension means we can
(uniquely) factor natural transformations $F \to Mp$ through
transformations $G_1 \to M$. We want a map $G_1 \to G_2$, for which it
will suffice to find a map $F \to G_2p$ --- but $\eta_2$ is right there!
In the other direction, we can factor $\eta_1$ to get a map $G_2 \to
G_1$. Since these factorisations are unique, we have a natural
isomorphism.

```agda
  σ-inversesp
    : ∀ {α : G₁ => G₂} {β : G₂ => G₁}
    → (α ◂ p) ∘nt η₁ ≡ η₂
    → (β ◂ p) ∘nt η₂ ≡ η₁
    → natural-inverses α β
  σ-inversesp α-factor β-factor = C′D.make-inverses
    (l₂.σ-uniq₂ η₂
      (Nat-path λ j → sym (D.pullr (β-factor ηₚ j) ∙ α-factor ηₚ j))
      (Nat-path λ j → sym (D.idl _)))
    (l₁.σ-uniq₂ η₁
      (Nat-path λ j → sym (D.pullr (α-factor ηₚ j) ∙ β-factor ηₚ j))
      (Nat-path λ j → sym (D.idl _)))
```

<!--
```agda
  σ-is-invertiblep
    : ∀ {α : G₁ => G₂}
    → (α ◂ p) ∘nt η₁ ≡ η₂
    → is-natural-invertible α
  σ-is-invertiblep {α = α} α-factor =
    C′D.inverses→invertible (σ-inversesp {α} α-factor l₂.σ-comm)

  σ-inverses : natural-inverses (l₁.σ η₂) (l₂.σ η₁)
  σ-inverses = σ-inversesp l₁.σ-comm l₂.σ-comm

  σ-is-invertible : is-natural-invertible (l₁.σ η₂)
  σ-is-invertible = σ-is-invertiblep l₁.σ-comm

  unique : natural-iso G₁ G₂
  unique = C′D.invertible→iso (l₁.σ η₂) (σ-is-invertiblep l₁.σ-comm)
```
-->

It's immediate from the construction that this isomorphism "sends
$\eta_1$ to $\eta_2$".

```agda
  unit : (l₁.σ η₂ ◂ p) ∘nt η₁ ≡ η₂
  unit = l₁.σ-comm
```

<!--
```agda
module _
    {p : Functor C C′} {F : Functor C D}
    {G : Functor C′ D} {eta}
    (lan : is-lan p F G eta)
    where

  private
    module lan = is-lan lan
    module D = Cat.Reasoning D
    module C′D = Cat.Reasoning Cat[ C′ , D ]
    open _=>_
```
-->

Another _uniqueness-like_ result we can state for left extensions is the
following: Given any functor $G' : \cC' \to \cD$ and candidate "unit"
transformation $\eta' : F \to G'p$, if a left extension $\Lan_p(F)$
sends $\eta'$ to a natural isomorphism, then $(G', \eta')$ is also a
left extension of $F$ along $p$.

```agda
  is-invertible→is-lan
    : ∀ {G' : Functor C′ D} {eta' : F => G' F∘ p}
    → is-natural-invertible (lan.σ eta')
    → is-lan p F G' eta'
  is-invertible→is-lan {G' = G'} {eta'} invert = lan' where
    open is-lan
    open C′D.is-invertible invert

    lan' : is-lan p F G' eta'
    lan' .σ α = lan.σ α ∘nt inv
    lan' .σ-comm {M} {α} = Nat-path λ j →
      (lan.σ α .η _ D.∘ inv .η _) D.∘ eta' .η j                      ≡˘⟨ D.refl⟩∘⟨ (lan.σ-comm ηₚ _) ⟩
      (lan.σ α .η _ D.∘ inv .η _) D.∘ (lan.σ eta' .η _ D.∘ eta .η j) ≡⟨ D.cancel-inner (invr ηₚ _) ⟩
      lan.σ α .η _ D.∘ eta .η j                                      ≡⟨ lan.σ-comm ηₚ _ ⟩
      α .η j                                                         ∎
    lan' .σ-uniq {M} {α} {σ′} p = Nat-path λ j →
      lan.σ α .η j D.∘ inv .η j                  ≡⟨ (lan.σ-uniq {σ′ = σ′ ∘nt lan.σ eta'} (Nat-path λ j → p ηₚ j ∙ D.pushr (sym (lan.σ-comm ηₚ j))) ηₚ j) D.⟩∘⟨refl ⟩
      (σ′ .η j D.∘ lan.σ eta' .η j) D.∘ inv .η _ ≡⟨ D.cancelr (invl ηₚ _) ⟩
      σ′ .η j                                    ∎
```

<!--
```agda
  natural-iso-of→is-lan
    : {F' : Functor C D}
    → (isos : natural-iso F F')
    → is-lan p F' G (eta ∘nt natural-iso.from isos)
  natural-iso-of→is-lan {F' = F'} isos = lan' where
    open is-lan
    module isos = natural-iso isos

    lan' : is-lan p F' G (eta ∘nt isos.from)
    lan' .σ α = lan.σ (α ∘nt isos.to)
    lan' .σ-comm {M} {α} = Nat-path λ j →
      lan.σ (α ∘nt isos.to) .η _ D.∘ eta .η j D.∘ isos.from .η j ≡⟨ D.pulll (lan.σ-comm ηₚ j) ⟩
      (α .η j D.∘ isos.to .η j) D.∘ isos.from .η j               ≡⟨ D.cancelr (isos.invl ηₚ _) ⟩
      α .η j ∎
    lan' .σ-uniq {M} {α} {σ′} p =
      lan.σ-uniq $ Nat-path λ j →
        α .η j D.∘ isos.to .η j                                    ≡⟨ (p ηₚ j) D.⟩∘⟨refl ⟩
        (σ′ .η _ D.∘ eta .η j D.∘ isos.from .η j) D.∘ isos.to .η j ≡⟨ D.deleter (isos.invr ηₚ _) ⟩
        σ′ .η _ D.∘ eta .η j ∎

  natural-iso-ext→is-lan
    : {G' : Functor C′ D}
    → (isos : natural-iso G G')
    → is-lan p F G' ((natural-iso.to isos ◂ p) ∘nt eta)
  natural-iso-ext→is-lan {G' = G'} isos = lan' where
    open is-lan
    module isos = natural-iso isos

    lan' : is-lan p F G' ((isos.to ◂ p) ∘nt eta)
    lan' .σ α = lan.σ α ∘nt isos.from
    lan' .σ-comm {M} {α} = Nat-path λ j →
      (lan.σ α .η _ D.∘ isos.from .η _) D.∘ isos.to .η _ D.∘ eta .η j ≡⟨ D.cancel-inner (isos.invr ηₚ _) ⟩
      lan.σ α .η _ D.∘ eta .η j                                       ≡⟨ lan.σ-comm ηₚ _ ⟩
      α .η j                                                          ∎
    lan' .σ-uniq {M} {α} {σ′} p = Nat-path λ j →
      lan.σ α .η j D.∘ isos.from .η j             ≡⟨ D.pushl (lan.σ-uniq {σ′ = σ′ ∘nt isos.to} (Nat-path λ j → p ηₚ j ∙ D.assoc _ _ _) ηₚ j) ⟩
      σ′ .η j D.∘ isos.to .η j D.∘ isos.from .η j ≡⟨ D.elimr (isos.invl ηₚ _) ⟩
      σ′ .η j                                     ∎

  natural-iso-along→is-lan
    : {p' : Functor C C′}
    → (isos : natural-iso p p')
    → is-lan p' F G ((G ▸ natural-iso.to isos) ∘nt eta)
  natural-iso-along→is-lan {p'} isos = lan' where
    open is-lan
    module isos = natural-iso isos
    open Cat.Functor.Reasoning

    lan' : is-lan p' F G ((G ▸ natural-iso.to isos) ∘nt eta)
    lan' .σ {M} α = lan.σ ((M ▸ isos.from) ∘nt α)
    lan' .σ-comm {M = M} = Nat-path λ j →
      D.pulll ((lan.σ _ .is-natural _ _ _))
      ∙ D.pullr (lan.σ-comm ηₚ _)
      ∙ cancell M (isos.invl ηₚ _)
    lan' .σ-uniq {M = M} {α = α} {σ′ = σ′} q = Nat-path λ c' →
      lan.σ-uniq {α = (M ▸ isos.from) ∘nt α} {σ′ = σ′}
        (Nat-path λ j →
          D.pushr (q ηₚ _)
          ∙ D.pulll (D.pullr (σ′ .is-natural _ _ _)
                     ∙ cancell M (isos.invr ηₚ _))) ηₚ c'

  universal-path→is-lan : ∀ {eta'} → eta ≡ eta' → is-lan p F G eta'
  universal-path→is-lan {eta'} q = lan' where
    open is-lan

    lan' : is-lan p F G eta'
    lan' .σ = lan.σ
    lan' .σ-comm = ap (_ ∘nt_) (sym q) ∙ lan.σ-comm
    lan' .σ-uniq r = lan.σ-uniq (r ∙ ap (_ ∘nt_) (sym q))

module _
    {p p' : Functor C C′} {F F' : Functor C D}
    {G G' : Functor C′ D} {eps eps'}
    where
  private
    module D = Cat.Reasoning D
    open Cat.Functor.Reasoning
    open _=>_
```
-->

Left Kan extensions are also invariant under arbitrary natural
isomorphisms. To get better definitional control, we allow “adjusting”
the resulting construction to talk about any natural transformation
which is propositionally equal to the whiskering:

```agda
  natural-isos→is-lan
    : (p-iso : natural-iso p p')
    → (F-iso : natural-iso F F')
    → (G-iso : natural-iso G G')
    → (natural-iso.to G-iso ◆ natural-iso.to p-iso) ∘nt eps ∘nt natural-iso.from F-iso ≡ eps'
    → is-lan p F G eps
    → is-lan p' F' G' eps'
```

<!--
```agda
  natural-isos→is-lan p-iso F-iso G-iso q lan =
    universal-path→is-lan
      (natural-iso-ext→is-lan
        (natural-iso-of→is-lan
          (natural-iso-along→is-lan
            lan
            p-iso)
          F-iso)
        G-iso)
      (Nat-path λ x →
        D.extendl (D.pulll (G-iso .to .is-natural _ _ _))
        ∙ q ηₚ _)
    where open natural-iso
```
-->

## Into univalent categories

As traditional with universal constructions, if $F : \cC \to \cD$ takes
values in a [univalent category], we can sharpen our result: the type of
left extensions of $F$ along $p$ is a proposition.

[univalent category]: Cat.Univalent.html#univalent-categories

```agda
Lan-is-prop
  : ∀ {p : Functor C C′} {F : Functor C D} → is-category D → is-prop (Lan p F)
Lan-is-prop {C = C} {C′ = C′} {D = D} {p = p} {F = F} d-cat L₁ L₂ = path where
```

<!--
```agda
  module L₁ = Lan L₁
  module L₂ = Lan L₂
  module Lu = Lan-unique L₁.has-lan L₂.has-lan

  open Lan

  c′d-cat : is-category Cat[ C′ , D ]
  c′d-cat = Functor-is-category d-cat
```
-->

That's because if $\cD$ is univalent, then [so is $[\cC',
\cD]$][Functor-is-category], so our natural isomorphism $i : G_1 \cong
G_2$ is equivalent to an identification $i' : G_1 \equiv G_2$. Then, our
tiny lemma stating that this isomorphism "sends $\eta_1$ to $\eta_2$" is
precisely the data of a dependent identification $\eta_1 \equiv \eta_2$
over $i'$.

[Functor-is-category]: Cat.Instances.Functor.html#Functor-is-category

```agda
  functor-path : L₁.Ext ≡ L₂.Ext
  functor-path = c′d-cat .to-path Lu.unique

  eta-path : PathP (λ i → F => functor-path i F∘ p) L₁.eta L₂.eta
  eta-path = Nat-pathp _ _ λ x →
    Univalent.Hom-pathp-reflr-iso d-cat (Lu.unit ηₚ _)
```

Since being a left extension is always a proposition when applied to
$(G, \eta)$, even when the categories are not univalent, we can finish
our proof.

```agda
  path : L₁ ≡ L₂
  path i .Ext = functor-path i
  path i .eta = eta-path i
  path i .has-lan =
    is-prop→pathp (λ i → is-lan-is-prop {p = p} {F} {functor-path i} {eta-path i})
      L₁.has-lan L₂.has-lan i
```

<!--
```agda
module
  Ran-unique
    {p : Functor C C′} {F : Functor C D}
    {G₁ G₂ : Functor C′ D} {ε₁ ε₂}
    (r₁ : is-ran p F G₁ ε₁)
    (r₂ : is-ran p F G₂ ε₂)
  where

  private
    module r₁ = is-ran r₁
    module r₂ = is-ran r₂
    module D = Cat.Reasoning D
    module C′D = Cat.Reasoning Cat[ C′ , D ]

  open C′D._≅_
  open C′D.Inverses

  σ-inversesp
    : ∀ {α : G₂ => G₁} {β : G₁ => G₂}
    → (ε₁ ∘nt (α ◂ p)) ≡ ε₂
    → (ε₂ ∘nt (β ◂ p)) ≡ ε₁
    → natural-inverses α β
  σ-inversesp α-factor β-factor =
    C′D.make-inverses
      (r₁.σ-uniq₂ ε₁
        (Nat-path λ j → sym (D.pulll (α-factor ηₚ j) ∙ β-factor ηₚ j))
        (Nat-path λ j → sym (D.idr _)))
      (r₂.σ-uniq₂ ε₂
        (Nat-path λ j → sym (D.pulll (β-factor ηₚ j) ∙ α-factor ηₚ j))
        (Nat-path λ j → sym (D.idr _)))

  σ-is-invertiblep
    : ∀ {α : G₂ => G₁}
    → (ε₁ ∘nt (α ◂ p)) ≡ ε₂
    → is-natural-invertible α
  σ-is-invertiblep {α} α-factor =
    C′D.inverses→invertible (σ-inversesp {α} α-factor r₂.σ-comm)

  σ-inverses : natural-inverses (r₁.σ ε₂) (r₂.σ ε₁)
  σ-inverses = σ-inversesp r₁.σ-comm r₂.σ-comm

  σ-is-invertible : is-natural-invertible (r₁.σ ε₂)
  σ-is-invertible = σ-is-invertiblep r₁.σ-comm

  unique : natural-iso G₁ G₂
  unique = C′D.invertible→iso (r₁.σ ε₂) (σ-is-invertiblep r₁.σ-comm) ni⁻¹

  counit : ε₁ ∘nt (r₁.σ ε₂ ◂ p) ≡ ε₂
  counit = r₁.σ-comm

module _
    {p : Functor C C′} {F : Functor C D}
    {G : Functor C′ D} {eps}
    (ran : is-ran p F G eps)
    where

  private
    module ran = is-ran ran
    module D = Cat.Reasoning D
    module C′D = Cat.Reasoning Cat[ C′ , D ]
    open _=>_

  -- These are more annoying to do via duality then it is to do by hand,
  -- due to the natural isos.
  is-invertible→is-ran
    : ∀ {G' : Functor C′ D} {eps'}
    → is-natural-invertible (ran.σ eps')
    → is-ran p F G' eps'
  is-invertible→is-ran {G' = G'} {eps'} invert = ran' where
    open is-ran
    open C′D.is-invertible invert

    ran' : is-ran p F G' eps'
    ran' .σ β = inv ∘nt ran.σ β
    ran' .σ-comm {M} {β} = Nat-path λ j →
      sym ((ran.σ-comm ηₚ _) D.⟩∘⟨refl)
      ·· D.cancel-inner (invl ηₚ _)
      ·· (ran.σ-comm ηₚ _)
    ran' .σ-uniq {M} {β} {σ′} p = Nat-path λ j →
      (D.refl⟩∘⟨ ran.σ-uniq {σ′ = ran.σ eps' ∘nt σ′} (Nat-path λ j → p ηₚ j ∙ D.pushl (sym (ran.σ-comm ηₚ j))) ηₚ _)
      ∙ D.cancell (invr ηₚ _)

  natural-iso-of→is-ran
    : {F' : Functor C D}
    → (isos : natural-iso F F')
    → is-ran p F' G (natural-iso.to isos ∘nt eps)
  natural-iso-of→is-ran {F'} isos = ran' where
    open is-ran
    module isos = natural-iso isos

    ran' : is-ran p F' G (isos.to ∘nt eps)
    ran' .σ β = ran.σ (isos.from ∘nt β)
    ran' .σ-comm {M} {β} = Nat-path λ j →
      D.pullr (ran.σ-comm ηₚ j)
      ∙ D.cancell (isos.invl ηₚ _)
    ran' .σ-uniq {M} {β} {σ′} p =
      ran.σ-uniq $ Nat-path λ j →
        (D.refl⟩∘⟨ p ηₚ j)
        ∙ D.deletel (isos.invr ηₚ _)

  natural-iso-ext→is-ran
    : {G' : Functor C′ D}
    → (isos : natural-iso G G')
    → is-ran p F G' (eps ∘nt (natural-iso.from isos ◂ p))
  natural-iso-ext→is-ran {G'} isos = ran' where
    open is-ran
    module isos = natural-iso isos

    ran' : is-ran p F G' (eps ∘nt (isos.from ◂ p))
    ran' .σ β = isos.to ∘nt ran.σ β
    ran' .σ-comm {M} {β} = Nat-path λ j →
      D.cancel-inner (isos.invr ηₚ _)
      ∙ ran.σ-comm ηₚ _
    ran' .σ-uniq {M} {β} {σ′} p = Nat-path λ j →
      D.pushr (ran.σ-uniq {σ′ = isos.from ∘nt σ′} (Nat-path λ j → p ηₚ j ∙ sym (D.assoc _ _ _)) ηₚ j)
      ∙ D.eliml (isos.invl ηₚ _)

  natural-iso-along→is-ran
    : {p' : Functor C C′}
    → (isos : natural-iso p p')
    → is-ran p' F G (eps ∘nt (G ▸ natural-iso.from isos))
  natural-iso-along→is-ran {p'} isos = ran' where
    open is-ran
    module isos = natural-iso isos
    open Cat.Functor.Reasoning

    ran' : is-ran p' F G (eps ∘nt (G ▸ natural-iso.from isos))
    ran' .σ {M} β = ran.σ (β ∘nt (M ▸ natural-iso.to isos))
    ran' .σ-comm {M = M} = Nat-path λ j →
      D.pullr (sym (ran.σ _ .is-natural _ _ _))
      ∙ D.pulll (ran.σ-comm ηₚ _)
      ∙ cancelr M (isos.invl ηₚ _)
    ran' .σ-uniq {M = M} {β = β} {σ′ = σ′} q = Nat-path λ c' →
      ran.σ-uniq {β = β ∘nt (M ▸ isos.to)} {σ′ = σ′}
        (Nat-path λ j →
          D.pushl (q ηₚ _)
          ∙ D.pullr (D.pulll (sym (σ′ .is-natural _ _ _))
                     ∙ cancelr M (isos.invr ηₚ _))) ηₚ c'

  universal-path→is-ran : ∀ {eps'} → eps ≡ eps' → is-ran p F G eps'
  universal-path→is-ran {eps'} q = ran' where
    open is-ran

    ran' : is-ran p F G eps'
    ran' .σ = ran.σ
    ran' .σ-comm = ap (_∘nt _) (sym q) ∙ ran.σ-comm
    ran' .σ-uniq r = ran.σ-uniq (r ∙ ap (_∘nt _) (sym q))

module _
    {p p' : Functor C C′} {F F' : Functor C D}
    {G G' : Functor C′ D} {eps eps'}
    where
  private
    module D = Cat.Reasoning D
    open _=>_

  natural-isos→is-ran
    : (p-iso : natural-iso p p')
    → (F-iso : natural-iso F F')
    → (G-iso : natural-iso G G')
    → (natural-iso.to F-iso ∘nt eps ∘nt (natural-iso.from G-iso ◆ natural-iso.from p-iso)) ≡ eps'
    → is-ran p F G eps
    → is-ran p' F' G' eps'
  natural-isos→is-ran p-iso F-iso G-iso p ran =
    universal-path→is-ran
      (natural-iso-ext→is-ran
        (natural-iso-of→is-ran
          (natural-iso-along→is-ran ran p-iso)
        F-iso)
      G-iso)
    (Nat-path λ c →
      sym (D.assoc _ _ _)
      ·· ap₂ D._∘_ refl (sym $ D.assoc _ _ _)
      ·· p ηₚ _)

Ran-is-prop
  : ∀ {p : Functor C C′} {F : Functor C D} → is-category D → is-prop (Ran p F)
Ran-is-prop {C = C} {C′ = C′} {D = D} {p = p} {F = F} d-cat R₁ R₂ = path where
  module R₁ = Ran R₁
  module R₂ = Ran R₂
  module Ru = Ran-unique R₁.has-ran R₂.has-ran

  open Ran

  c′d-cat : is-category Cat[ C′ , D ]
  c′d-cat = Functor-is-category d-cat

  fp : R₁.Ext ≡ R₂.Ext
  fp = c′d-cat .to-path Ru.unique

  εp : PathP (λ i → fp i F∘ p => F) R₁.eps R₂.eps
  εp = Nat-pathp _ _ λ x → Univalent.Hom-pathp-refll-iso d-cat (Ru.counit ηₚ _)

  path : R₁ ≡ R₂
  path i .Ext = fp i
  path i .eps = εp i
  path i .has-ran =
    is-prop→pathp (λ i → is-ran-is-prop {p = p} {F} {fp i} {εp i})
      R₁.has-ran R₂.has-ran i
```
-->
