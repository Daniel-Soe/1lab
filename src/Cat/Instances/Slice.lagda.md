```agda
open import Cat.Instances.Shape.Terminal
open import Cat.Instances.Shape.Join
open import Cat.Diagram.Limit.Base
open import Cat.Diagram.Limit.Base
open import Cat.Instances.Discrete
open import Cat.Instances.Functor
open import Cat.Diagram.Pullback
open import Cat.Diagram.Terminal
open import Cat.Diagram.Product
open import Cat.Functor.Base
open import Cat.Univalent
open import Cat.Prelude

open import Data.Sum

import Cat.Reasoning

module Cat.Instances.Slice where
```

<!--
```agda
private variable
  o ℓ o′ ℓ′ : Level
open Functor
open _=>_

module _ {o ℓ} {C : Precategory o ℓ} where
  private
    module C = Cat.Reasoning C
    variable a b c : C.Ob
```
-->

# Slice categories

When working in $\Sets$, there is an evident notion of _family indexed
by a set_: a family of sets $(F_i)_{i \in I}$ is equivalently a functor
$[\rm{Disc}(I), \Sets]$, where we have equipped the set $I$ with the
[discrete category] structure. This works essentially because of the
[discrete category-global sections][coh] adjunction, but in general this
can not be applied to other categories, like $\rm{Groups}$. How,
then, should we work with "indexed families" in general categories?

[discrete category]: Cat.Instances.Discrete.html
[coh]: Cat.Instances.StrictCat.Cohesive.html#disc-γ

The answer is to consider, rather than families themselves, the
_projections from their total spaces_ as the primitive objects. A family
indexed by $I$, then, would consist of an object $A$ and a morphism $t :
A \to I$, where $A$ is considered as the "total space" object and $t$
assigns gives the "tag" of each object. By analysing how $t$ [pulls
back] along maps $B \to I$, we recover a notion of "fibres": the
collection with index $i$ can be recovered as the pullback $t^*i$.

[pulls back]: Cat.Diagram.Pullback.html

Note that, since the discussion in the preceding paragraph made no
mention of the category of sets, it applies in any category! More
generally, for any category $\cC$ and object $c : \cC$, we have a
_category of objects indexed by $c$_, the **slice category** $\cC/c$.
An object of "the slice over $c$" is given by an object $d : \cC$ to
serve as the domain, and a map $f : d \to c$.

```agda
  record /-Obj (c : C.Ob) : Type (o ⊔ ℓ) where
    no-eta-equality
    constructor cut
    field
      {domain} : C.Ob
      map      : C.Hom domain c
```

A map between $f : a \to c$ and $g : b \to c$ is given by a map $h : a
\to b$ such that the triangle below commutes. Since we're thinking of
$f$ and $g$ as _families indexed by $c$_, commutativity of the triangle
says that the map $h$ "respects reindexing", or less obliquely
"preserves fibres".

~~~{.quiver}
\[\begin{tikzcd}
  a && b \\
  & c
  \arrow["f"', from=1-1, to=2-2]
  \arrow["g", from=1-3, to=2-2]
  \arrow["h", from=1-1, to=1-3]
\end{tikzcd}\]
~~~

```agda
  record /-Hom (a b : /-Obj c) : Type ℓ where
    no-eta-equality
    private
      module a = /-Obj a
      module b = /-Obj b
    field
      map      : C.Hom a.domain b.domain
      commutes : b.map C.∘ map ≡ a.map
```

<!--
```agda
  /-Obj-path : ∀ {c} {x y : /-Obj c}
             → (p : x ./-Obj.domain ≡ y ./-Obj.domain)
             → PathP (λ i → C.Hom (p i) c) (x ./-Obj.map) (y ./-Obj.map)
             → x ≡ y
  /-Obj-path p q i ./-Obj.domain = p i
  /-Obj-path p q i ./-Obj.map = q i

  /-Hom-pathp : ∀ {c a a′ b b′} (p : a ≡ a′) (q : b ≡ b′)
                {x : /-Hom {c = c} a b} {y : /-Hom a′ b′}
              → PathP (λ i → C.Hom (p i ./-Obj.domain) (q i ./-Obj.domain))
                        (x ./-Hom.map) (y ./-Hom.map)
              → PathP (λ i → /-Hom (p i) (q i)) x y
  /-Hom-pathp p q {x} {y} r = path where
    open /-Hom

    path : PathP (λ i → /-Hom (p i) (q i))  x y
    path i .map = r i
    path i .commutes =
      is-prop→pathp
        (λ i → C.Hom-set (p i ./-Obj.domain) _
                         (q i ./-Obj.map C.∘ r i) (p i ./-Obj.map))
        (x .commutes) (y .commutes) i

  /-Hom-path : ∀ {c a b} {x y : /-Hom {c = c} a b}
             → x ./-Hom.map ≡ y ./-Hom.map
             → x ≡ y
  /-Hom-path = /-Hom-pathp refl refl

  private unquoteDecl eqv = declare-record-iso eqv (quote /-Hom)

  abstract
    /-Hom-is-set : ∀ {c a b} → is-set (/-Hom {c = c} a b)
    /-Hom-is-set {a = a} {b} = hl where abstract
      open C.HLevel-instance

      hl : is-set (/-Hom a b)
      hl = Iso→is-hlevel 2 eqv (hlevel 2)
```
-->

The slice category $\cC/c$ is given by the `/-Obj`{.Agda} and
`/-Hom`{.Agda}s.

```agda
Slice : (C : Precategory o ℓ) → Precategory.Ob C → Precategory _ _
Slice C c = precat where
  import Cat.Reasoning C as C
  open Precategory
  open /-Hom
  open /-Obj

  precat : Precategory _ _
  precat .Ob = /-Obj {C = C} c
  precat .Hom = /-Hom
  precat .Hom-set x y = /-Hom-is-set
  precat .id .map      = C.id
  precat .id .commutes = C.idr _
```

For composition in the slice over $c$, note that if the triangle (the
commutativity condition for $f$) and the rhombus (the commutativity
condition for $g$) both commute, then so does the larger triangle (the
commutativity for $g \circ f$).

~~~{.quiver .tall-1}
\[\begin{tikzcd}
  x && y && z \\
  & c \\
  && c
  \arrow["{x_m}"', from=1-1, to=2-2]
  \arrow["{y_m}", from=1-3, to=2-2]
  \arrow["f", from=1-1, to=1-3]
  \arrow["g", from=1-3, to=1-5]
  \arrow["{z_m}", from=1-5, to=3-3]
  \arrow[Rightarrow, no head, from=2-2, to=3-3]
\end{tikzcd}\]
~~~

```agda
  precat ._∘_ {x} {y} {z} f g = fog where
    module f = /-Hom f
    module g = /-Hom g
    fog : /-Hom _ _
    fog .map = f.map C.∘ g.map
    fog .commutes =
      z .map C.∘ f.map C.∘ g.map ≡⟨ C.pulll f.commutes ⟩
      y .map C.∘ g.map           ≡⟨ g.commutes ⟩
      x .map                     ∎
  precat .idr f = /-Hom-path (C.idr _)
  precat .idl f = /-Hom-path (C.idl _)
  precat .assoc f g h = /-Hom-path (C.assoc _ _ _)
```

## Limits

We discuss some limits in the slice of $\cC$ over $c$. First, every
slice category has a terminal object, given by the identity map
$\id : c \to c$.

```agda
module _ {o ℓ} {C : Precategory o ℓ} {c : Precategory.Ob C} where
  import Cat.Reasoning C as C
  import Cat.Reasoning (Slice C c) as C/c
  open /-Hom
  open /-Obj

  Slice-terminal-object : is-terminal (Slice C c) (cut C.id)
  Slice-terminal-object obj .centre .map = obj .map
  Slice-terminal-object obj .centre .commutes = C.idl _
  Slice-terminal-object obj .paths other =
    /-Hom-path (sym (other .commutes) ∙ C.idl _)
```

Products in a slice category are slightly more complicated, but recall
that another word for pullback is "fibred product". Indeed, in the
pullback page we noted that the pullback of $X \to Z$ and $Y \to Z$ is
exactly the product of those maps in the slice over $Z$.

<!--
```agda
module _ {o ℓ} {C : Precategory o ℓ} {c : Precategory.Ob C} where
  import Cat.Reasoning C as C
  import Cat.Reasoning (Slice C c) as C/c
  private variable
    a b : C.Ob
    f g π₁ π₂ : C.Hom a b
  open /-Hom
  open /-Obj
```
-->

Suppose we have a pullback diagram like the one below, i.e., a limit of
the diagram $a \xrightarrow{f} c \xleftarrow{g} b$, in the category
$\cC$. We'll show that it's also a limit of the (discrete) diagram
consisting of $f$ and $g$, but now in the slice category $\cC/c$.

~~~{.quiver}
\[\begin{tikzcd}
  {a \times_c b} && a \\
  \\
  b && c
  \arrow["{\pi_2}"', from=1-1, to=3-1]
  \arrow["{\pi_1}", from=1-1, to=1-3]
  \arrow["f", from=1-3, to=3-3]
  \arrow["g"', from=3-1, to=3-3]
  \arrow["\lrcorner"{anchor=center, pos=0.125}, draw=none, from=1-1, to=3-3]
\end{tikzcd}\]
~~~

For starters, note that we have seemingly "two" distinct choices for
maps $a \times_c b \to c$, but since the square above commutes, either
one will do. For definiteness, we go with the composite $f \circ \pi_1$.

```agda
  module
    _ {f g : /-Obj c} {Pb : C.Ob} {π₁ : C.Hom Pb (f .domain)}
                                  {π₂ : C.Hom Pb (g .domain)}
      (pb : is-pullback C {X = f .domain} {Z = c} {Y = g .domain} {P = Pb}
        π₁ (map {_} {_} {C} {c} f) π₂ (map {_} {_} {C} {c} g))
    where
    private module pb = is-pullback pb

    is-pullback→product-over : C/c.Ob
    is-pullback→product-over = cut (f .map C.∘ π₁)
```

Now, by commutativity of the square, the maps $\pi_1$ and $\pi_2$ in the
diagram above extend to maps $(f \circ \pi_1) \to f$ and $(f \circ
\pi_1) \to g$ in $\cC/c$. Indeed, note that by scribbling a red line
across the diagonal of the diagram, we get the two needed triangles as
the induced subdivisions.

~~~{.quiver}
\[\begin{tikzcd}
  {X \times_Z Y} && X \\
  \\
  Y && Z
  \arrow["{\pi_2}"', from=1-1, to=3-1]
  \arrow["{\pi_1}", from=1-1, to=1-3]
  \arrow["f", from=1-3, to=3-3]
  \arrow["g"', from=3-1, to=3-3]
  \arrow["{f \circ \pi_1}"{description}, color={rgb,255:red,214;green,92;blue,92}, from=1-1, to=3-3]
\end{tikzcd}\]
~~~

```agda
    is-pullback→π₁ : C/c.Hom is-pullback→product-over f
    is-pullback→π₁ .map      = π₁
    is-pullback→π₁ .commutes i = f .map C.∘ π₁

    is-pullback→π₂ : C/c.Hom is-pullback→product-over g
    is-pullback→π₂ .map        = π₂
    is-pullback→π₂ .commutes i = pb.square (~ i)

    open is-product
```

Unfolding what it means for a diagram to be a universal cone over the
discrete diagram consisting of $f$ and $g$ in the category $\cC/c$,
we see that it is exactly the data of the pullback of $f$ and $g$ in
$\cC$, as below:

```agda
    is-pullback→is-fibre-product
      : is-product (Slice C c) is-pullback→π₁ is-pullback→π₂
    is-pullback→is-fibre-product .⟨_,_⟩ {Q} /f /g = factor
      where
        module f = /-Hom /f
        module g = /-Hom /g

        factor : C/c.Hom Q _
        factor .map = pb.universal (f.commutes ∙ sym g.commutes)
        factor .commutes =
          (f .map C.∘ π₁) C.∘ pb.universal _ ≡⟨ C.pullr pb.p₁∘universal ⟩
          f .map C.∘ f.map                  ≡⟨ f.commutes ⟩
          Q .map                            ∎

    is-pullback→is-fibre-product .π₁∘factor = /-Hom-path pb.p₁∘universal
    is-pullback→is-fibre-product .π₂∘factor = /-Hom-path pb.p₂∘universal
    is-pullback→is-fibre-product .unique other p q =
      /-Hom-path (pb.unique (ap map p) (ap map q))

  Pullback→Fibre-product
    : ∀ {f g : /-Obj c}
    → Pullback C (f .map) (g .map) → Product (Slice C c) f g
  Pullback→Fibre-product pb .Product.apex = _
  Pullback→Fibre-product pb .Product.π₁ = _
  Pullback→Fibre-product pb .Product.π₂ = _
  Pullback→Fibre-product pb .Product.has-is-product =
    is-pullback→is-fibre-product (pb .Pullback.has-is-pb)
```

While products and terminal objects in $\cC/X$ do not correspond to
those in $\cC$, _pullbacks_ (and equalisers) are precisely
equivalent. A square is a pullback in $\cC/X$ iff. the square
consisting of their underlying objects and maps is a square in $\cC$.

The "if" direction (what is `pullback-above→pullback-below`{.Agda}) in
the code is essentially an immediate consequence of the fact that
equality of morphisms in $\cC/X$ may be tested in $\cC$, but we do
have to take some care when extending the "universal" morphism back down
to the slice category (see the calculation marked `{- * -}`{.Agda}).

```agda
module _ where
  open /-Obj
  open /-Hom

  pullback-above→pullback-below
    : ∀ {o ℓ} {C : Precategory o ℓ} {X : Precategory.Ob C}
    → ∀ {P A B c} {p1 f p2 g}
    → is-pullback C (p1 .map) (f .map) (p2 .map) (g .map)
    → is-pullback (Slice C X) {P} {A} {B} {c} p1 f p2 g
  pullback-above→pullback-below {C = C} {P = P} {a} {b} {c} {p1} {f} {p2} {g} pb
    = pb′ where
      open is-pullback
      open Cat.Reasoning C

      pb′ : is-pullback (Slice _ _) _ _ _ _
      pb′ .square = /-Hom-path (pb .square)
      pb′ .universal p .map = pb .universal (ap map p)
      pb′ .universal {P'} {p₁' = p₁'} p .commutes =
        (c .map ∘ pb .universal (ap map p))           ≡˘⟨ (pulll (p1 .commutes)) ⟩
        (P .map ∘ p1 .map ∘ pb .universal (ap map p)) ≡⟨ ap (_ ∘_) (pb .p₁∘universal) ⟩
        (P .map ∘ p₁' .map)                          ≡⟨ p₁' .commutes ⟩
        P' .map                                      ∎ {- * -}
      pb′ .p₁∘universal = /-Hom-path (pb .p₁∘universal)
      pb′ .p₂∘universal = /-Hom-path (pb .p₂∘universal)
      pb′ .unique p q = /-Hom-path (pb .unique (ap map p) (ap map q))

  pullback-below→pullback-above
    : ∀ {o ℓ} {C : Precategory o ℓ} {X : Precategory.Ob C}
    → ∀ {P A B c} {p1 f p2 g}
    → is-pullback (Slice C X) {P} {A} {B} {c} p1 f p2 g
    → is-pullback C (p1 .map) (f .map) (p2 .map) (g .map)
  pullback-below→pullback-above {C = C} {P = P} {p1 = p1} {f} {p2} {g} pb = pb′ where
    open Cat.Reasoning C
    open is-pullback
    pb′ : is-pullback _ _ _ _ _
    pb′ .square = ap map (pb .square)
    pb′ .universal {P′ = P'} {p₁'} {p₂'} p =
      pb .universal {P′ = cut (P .map ∘ p₁')}
        {p₁' = record { map = p₁' ; commutes = refl }}
        {p₂' = record { map = p₂' ; commutes = sym (pulll (g .commutes))
                                             ∙ sym (ap (_ ∘_) p)
                                             ∙ pulll (f .commutes)
                      }}
        (/-Hom-path p)
        .map
    pb′ .p₁∘universal = ap map $ pb .p₁∘universal
    pb′ .p₂∘universal = ap map $ pb .p₂∘universal
    pb′ .unique p q = ap map $ pb .unique
      {lim' = record { map = _ ; commutes = sym (pulll (p1 .commutes)) ∙ ap (_ ∘_) p }}
      (/-Hom-path p) (/-Hom-path q)
```

# Slices of Sets

We now prove the correspondence between slices of $\Sets$ and functor
categories into $\Sets$, i.e. the corresponding between indexing and
slicing mentioned in the first paragraph.

```agda

module _ {I : Set ℓ} where
  open /-Obj
  open /-Hom
```

We shall prove that the functor `Total-space`{.Agda}, defined below, is
an equivalence of categories, i.e. that it is fully faithful and
essentially surjective. But first, we must define the functor! Like its
name implies, it maps the functor $F : I → \Sets$ to the first
projection map $\rm{fst} : \sum F \to I$.

```agda
  Total-space : Functor Cat[ Disc′ I , Sets ℓ ] (Slice (Sets ℓ) I)
  Total-space .F₀ F .domain = el (Σ _ (∣_∣ ⊙ F₀ F)) hlevel!
  Total-space .F₀ F .map = fst

  Total-space .F₁ nt .map (i , x) = i , nt .η _ x
  Total-space .F₁ nt .commutes    = refl

  Total-space .F-id    = /-Hom-path refl
  Total-space .F-∘ _ _ = /-Hom-path refl
```

To prove that the `Total-space`{.Agda} functor is `fully faithful`{.Agda
ident=is-fully-faithful}, we will exhibit a quasi-inverse to its action
on morphisms. Given a fibre-preserving map between $\rm{fst} : \sum
F \to I$ and $\rm{fst} : \sum G \to I$, we recover a natural
transformation between $F$ and $G$. The hardest part is showing
naturality, where we use path induction.

```agda
  Total-space-is-ff : is-fully-faithful Total-space
  Total-space-is-ff {f1} {f2} = is-iso→is-equiv
    (iso from linv (λ x → Nat-path (λ x → funext (λ _ → transport-refl _))))
    where
      from : /-Hom (Total-space .F₀ f1) (Total-space .F₀ f2) → f1 => f2
      from mp = nt where
        eta : ∀ i → ∣ F₀ f1 i ∣ → ∣ F₀ f2 i ∣
        eta i j =
          subst (∣_∣ ⊙ F₀ f2) (happly (mp .commutes) _) (mp .map (i , j) .snd)

        nt : f1 => f2
        nt .η = eta
        nt .is-natural _ _ f =
          J (λ _ p → eta _ ⊙ F₁ f1 p ≡ F₁ f2 p ⊙ eta _)
            (ap (eta _ ⊙_) (F-id f1) ∙ sym (ap (_⊙ eta _) (F-id f2)))
            f
```

<!--
```agda
      linv : is-left-inverse (F₁ Total-space) from
      linv x =
        /-Hom-path (funext (λ y →
          Σ-path (sym (happly (x .commutes) _))
            ( sym (transport-∙ (ap (∣_∣ ⊙ F₀ f2) (happly (x .commutes) y))
                          (sym (ap (∣_∣ ⊙ F₀ f2) (happly (x .commutes) y)))
                          _)
            ·· ap₂ transport (∙-inv-r (ap (∣_∣ ⊙ F₀ f2) (happly (x .commutes) y)))
                             refl
            ·· transport-refl _)))
```
-->

For essential surjectivity, given a map $f : X \to I$, we recover a
family of sets $(f^*i)_{i \in I}$ by taking the `fibre`{.Agda} of $f$
over each point, which cleanly extends to a functor. To show that the
`Total-space`{.Agda} of this functor is isomorphic to the map we started
with, we use one of the auxiliary lemmas used in the construction of an
object classifier: `Total-equiv`{.Agda}. This is cleaner than exhibiting
an isomorphism directly, though it does involve an appeal to univalence.

```agda
  Total-space-is-eso : is-split-eso Total-space
  Total-space-is-eso fam = functor , path→iso path
    where
      functor : Functor _ _
      functor .F₀ i = el! (fibre (fam .map) i)
      functor .F₁ p = subst (fibre (fam .map)) p
      functor .F-id = funext transport-refl
      functor .F-∘ f g = funext (subst-∙ (fibre (fam .map)) _ _)

      path : F₀ Total-space functor ≡ fam
      path = /-Obj-path (n-ua (Total-equiv _  e⁻¹)) (ua→ λ a → sym (a .snd .snd))
```

# Slices preserve univalence

An important property of slice categories is that they preserve
[univalence]. This can be seen intuitively: If $\cC$ is a univalent
category, then let $a, b, c$ be some objects, with the pairs $(a, f)$
and $(b, g)$ objects in the slice $\cC/c$. An isomorphism $h : (a, f)
\cong (b, g)$ induces an identification $a \equiv b$, which extends to
an identification $(a, f) \equiv (b, g)$ since $h \circ g = f$.

[univalence]: Cat.Univalent.html

```agda
module _ {C : Precategory o ℓ} {o : Precategory.Ob C} (isc : is-category C) where
  private
    module C   = Cat.Reasoning C
    module C/o = Cat.Reasoning (Slice C o)

    open /-Obj
    open /-Hom
    open C/o._≅_
    open C._≅_

  slice-is-category : is-category (Slice C o)
  slice-is-category .to-path x = /-Obj-path (isc .to-path $
    C.make-iso (x .to .map) (x .from .map)
      (ap map (C/o._≅_.invl x)) (ap map (C/o._≅_.invr x)))
    (Univalent.Hom-pathp-refll-iso isc (x .from .commutes))
  slice-is-category .to-path-over x = C/o.≅-pathp refl _ $
    /-Hom-pathp _ _ (Univalent.Hom-pathp-reflr-iso isc (C.idr _))
```

# Arbitrary limits in slices

Suppose we have some really weird diagram $F : \cJ \to \cC/c$,
like the one below.  Well, alright, it's not that weird, but it's not a
pullback or a terminal object, so we don't _a priori_ know how to
compute it.

~~~{.quiver}
\[\begin{tikzcd}
  & {(b,g)} && {(d,i)} \\
  {(a,f)} && {(c,h)}
  \arrow["x"', from=1-2, to=2-1]
  \arrow["y", from=1-2, to=2-3]
  \arrow["z"', from=1-4, to=2-3]
\end{tikzcd}\]
~~~

The observation that will let us compute a limit for this diagram is
inspecting the computation of products in slice categories, above. To
compute the product of $(a, f)$ and $(b, g)$, we had to pass to a
_pullback_ of $a \xto{f} c \xot{b}$ in $\cC$ --- which we had assumed
exists. But! Take a look at what that diagram _looks like_:

~~~{.quiver}
\[\begin{tikzcd}
  a && b \\
  \\
  & c
  \arrow["f"', from=1-1, to=3-2]
  \arrow["g", from=1-3, to=3-2]
\end{tikzcd}\]
~~~

We "exploded" a diagram of shape $\bullet \quad \bullet$ to one of shape
$\bullet \to \bullet \ot \bullet$. This process can be described in a
way easier to generalise: We "exploded" our diagram $F : \{*,*\} \to
\cC/c$ to one indexed by a category which contains $\{*,*\}$,
contains an extra point, and has a unique map between each object of
$\{*,*\}$ --- the [_join_] of these categories.

[_join_]: Cat.Instances.Shape.Join.html

<!--
```agda
module
  _ {C : Precategory o ℓ}
    {J : Precategory o′ ℓ′}
    {o : Precategory.Ob C}
    (F : Functor J (Slice C o))
    where

  open Terminal
  open /-Obj
  open /-Hom

  private
    module C   = Cat.Reasoning C
    module J   = Cat.Reasoning J
    module C/o = Cat.Reasoning (Slice C o)
    module F = Functor F
```
-->

Generically, if we have a diagram $F : J \to \cC/c$, we can "explode"
this into a diagram $F' : (J \star \{*\}) \to \cC$, compute the limit
in $\cC$, then pass back to the slice category.

```agda
    F′ : Functor (J ⋆ ⊤Cat) C
    F′ .F₀ (inl x) = F.₀ x .domain
    F′ .F₀ (inr x) = o
    F′ .F₁ {inl x} {inl y} (lift f) = F.₁ f .map
    F′ .F₁ {inl x} {inr y} _ = F.₀ x .map
    F′ .F₁ {inr x} {inr y} (lift h) = C.id
    F′ .F-id {inl x} = ap map F.F-id
    F′ .F-id {inr x} = refl
    F′ .F-∘ {inl x} {inl y} {inl z} (lift f) (lift g) = ap map (F.F-∘ f g)
    F′ .F-∘ {inl x} {inl y} {inr z} (lift f) (lift g) = sym (F.F₁ g .commutes)
    F′ .F-∘ {inl x} {inr y} {inr z} (lift f) (lift g) = C.introl refl
    F′ .F-∘ {inr x} {inr y} {inr z} (lift f) (lift g) = C.introl refl

  limit-above→limit-in-slice : Limit F′ → Limit F
  limit-above→limit-in-slice lims = to-limit (to-is-limit lim) where
    module lims = Limit lims
    open make-is-limit

    apex : C/o.Ob
    apex = cut (lims.ψ (inr tt))

    nadir : (j : J.Ob) → /-Hom apex (F .F₀ j)
    nadir j .map = lims.ψ (inl j)
    nadir j .commutes = lims.commutes (lift tt)

    module Cone
      {x : C/o.Ob}
      (eta : (j : J.Ob) → C/o.Hom x (F .F₀ j))
      (p : ∀ {i j : J.Ob} → (f : J.Hom i j) → F .F₁ f C/o.∘ eta i ≡ eta j)
      where

        ϕ : (j : J.Ob ⊎ ⊤) → C.Hom (x .domain) (F′ .F₀ j)
        ϕ (inl j) = eta j .map
        ϕ (inr _) = x .map

        ϕ-commutes
          : ∀ {i j : J.Ob ⊎ ⊤}
          → (f : ⋆Hom J ⊤Cat i j)
          → F′ .F₁ f C.∘ ϕ i ≡ ϕ j
        ϕ-commutes {inl i} {inl j} (lift f) = ap map (p f)
        ϕ-commutes {inl i} {inr j} (lift f) = eta i .commutes
        ϕ-commutes {inr i} {inr x} (lift f) = C.idl _

        ϕ-factor
          : ∀ (other : /-Hom x apex)
          → (∀ j → nadir j C/o.∘ other ≡ eta j)
          → (j : J.Ob ⊎ ⊤)
          → lims.ψ j C.∘ other .map ≡ ϕ j
        ϕ-factor other q (inl j) = ap map (q j)
        ϕ-factor other q (inr tt) = other .commutes

    lim : make-is-limit F apex
    lim .ψ = nadir
    lim .commutes f =
      /-Hom-path (lims.commutes (lift f))
    lim .universal {x} eta p .map =
      lims.universal (Cone.ϕ eta p) (Cone.ϕ-commutes eta p)
    lim .universal eta p .commutes =
      lims.factors _ _
    lim .factors eta p =
      /-Hom-path (lims.factors _ _)
    lim .unique eta p other q =
      /-Hom-path $ lims.unique _ _ (other .map) (Cone.ϕ-factor eta p other q)
```

In particular, if a category $\cC$ is complete, then so are its slices:

```agda
is-complete→slice-is-complete
  : ∀ {ℓ o ℓ′} {C : Precategory o ℓ} {c : Precategory.Ob C}
  → is-complete o′ ℓ′ C
  → is-complete o′ ℓ′ (Slice C c)
is-complete→slice-is-complete lims F = limit-above→limit-in-slice F (lims _)
```
