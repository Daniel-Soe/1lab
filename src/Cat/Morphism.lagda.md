```agda
open import 1Lab.Prelude hiding (_∘_ ; id ; _↪_)

open import Cat.Solver
open import Cat.Base

module Cat.Morphism {o h} (C : Precategory o h) where
```

<!--
```agda
open Precategory C public
private variable
  a b c d : Ob
  f : Hom a b
```
-->

# Morphisms

This module defines the three most important classes of morphisms that
can be found in a category: **monomorphisms**, **epimorphisms**, and
**isomorphisms**.

## Monos

A morphism is said to be **monic** when it is left-cancellable. A
**monomorphism** from $A$ to $B$, written $A \mono B$, is a monic
morphism.

```agda
is-monic : Hom a b → Type _
is-monic {a = a} f = ∀ {c} → (g h : Hom c a) → f ∘ g ≡ f ∘ h → g ≡ h

is-monic-is-prop : ∀ {a b} (f : Hom a b) → is-prop (is-monic f)
is-monic-is-prop f x y i {c} g h p = Hom-set _ _ _ _ (x g h p) (y g h p) i

record _↪_ (a b : Ob) : Type (o ⊔ h) where
  field
    mor   : Hom a b
    monic : is-monic mor

open _↪_ public
```

Conversely, a morphism is said to be **epic** when it is
right-cancellable.  An **epimorphism** from $A$ to $B$, written $A \epi
B$, is an epic morphism.

## Epis

```agda
is-epic : Hom a b → Type _
is-epic {b = b} f = ∀ {c} → (g h : Hom b c) → g ∘ f ≡ h ∘ f → g ≡ h

is-epic-is-prop : ∀ {a b} (f : Hom a b) → is-prop (is-epic f)
is-epic-is-prop f x y i {c} g h p = Hom-set _ _ _ _ (x g h p) (y g h p) i

record _↠_ (a b : Ob) : Type (o ⊔ h) where
  field
    mor  : Hom a b
    epic : is-epic mor

open _↠_ public
```

## Sections

A morphism $s : B \to A$ is a section of another morphism $r : A \to B$
when $r \cdot s = id$. The intuition for this name is that $s$ picks
out a cross-section of $a$ from $b$. For instance, $r$ could map
animals to their species; a section of this map would have to pick out
a canonical representative of each species from the set of all animals.

```agda
_section-of_ : (s : Hom b a) (r : Hom a b) → Type _
s section-of r = r ∘ s ≡ id

section-of-is-prop
  : ∀ {s : Hom b a} {r : Hom a b}
  → is-prop (s section-of r)
section-of-is-prop = Hom-set _ _ _ _

record has-section (r : Hom a b) : Type h where
  constructor make-section
  field
    section : Hom b a
    is-section : section section-of r

open has-section public

id-has-section : ∀ {a} → has-section (id {a})
id-has-section .section = id
id-has-section .is-section = idl _
```

## Retracts

A morphism $r : A \to B$ is a retract of another morphism $s : B \to A$
when $r \cdot s = id$. Note that this is the same equation involved
in the definition of a section; retracts and sections always come in
pairs. If sections solve a sort of "curration problem" where we are
asked to pick out canonical representatives, then retracts solve a
sort of "classification problem".

```agda
_retract-of_ : (r : Hom a b) (s : Hom b a) → Type _
r retract-of s = r ∘ s ≡ id

retract-of-is-prop
  : ∀ {s : Hom b a} {r : Hom a b}
  → is-prop (r retract-of s)
retract-of-is-prop = Hom-set _ _ _ _

record has-retract (s : Hom b a) : Type h where
  constructor make-retract
  field
    retract : Hom a b
    is-retract : retract retract-of s

open has-retract public

id-has-retract : ∀ {a} → has-retract (id {a})
id-has-retract .retract = id
id-has-retract .is-retract = idl _
```

## Isos

Maps $f : A \to B$ and $g : B \to A$ are **inverses** when we have $f
\circ g$ and $g \circ f$ both equal to the identity. A map $f : A \to B$
**is invertible** (or **is an isomorphism**) when there exists a $g$ for
which $f$ and $g$ are inverses. An **isomorphism** $A \cong B$ is a
choice of map $A \to B$, together with a specified inverse.

```agda
record Inverses (f : Hom a b) (g : Hom b a) : Type h where
  field
    invl : f ∘ g ≡ id
    invr : g ∘ f ≡ id

open Inverses

record is-invertible (f : Hom a b) : Type h where
  field
    inv : Hom b a
    inverses : Inverses f inv

  open Inverses inverses public

  op : is-invertible inv
  op .inv = f
  op .inverses .Inverses.invl = invr inverses
  op .inverses .Inverses.invr = invl inverses

record _≅_ (a b : Ob) : Type h where
  field
    to       : Hom a b
    from     : Hom b a
    inverses : Inverses to from

  open Inverses inverses public

open _≅_ public
```

A given map is invertible in at most one way: If you have $f : A \to B$
with two _inverses_ $g : B \to A$ and $h : B \to A$, then not only are
$g = h$ equal, but the witnesses of these equalities are irrelevant.

```agda
Inverses-are-prop : ∀ {f : Hom a b} {g : Hom b a} → is-prop (Inverses f g)
Inverses-are-prop x y i .Inverses.invl = Hom-set _ _ _ _ (x .invl) (y .invl) i
Inverses-are-prop x y i .Inverses.invr = Hom-set _ _ _ _ (x .invr) (y .invr) i

is-invertible-is-prop : ∀ {f : Hom a b} → is-prop (is-invertible f)
is-invertible-is-prop {a = a} {b = b} {f = f} g h = p where
  module g = is-invertible g
  module h = is-invertible h

  g≡h : g.inv ≡ h.inv
  g≡h =
    g.inv             ≡⟨ sym (idr _) ∙ ap₂ _∘_ refl (sym h.invl) ⟩
    g.inv ∘ f ∘ h.inv ≡⟨ assoc _ _ _ ·· ap₂ _∘_ g.invr refl ·· idl _ ⟩
    h.inv             ∎

  p : g ≡ h
  p i .is-invertible.inv = g≡h i
  p i .is-invertible.inverses =
    is-prop→pathp (λ i → Inverses-are-prop {g = g≡h i}) g.inverses h.inverses i
```

We note that the identity morphism is always iso, and that isos compose:

<!--
```agda
make-inverses : {f : Hom a b} {g : Hom b a} → f ∘ g ≡ id → g ∘ f ≡ id → Inverses f g
make-inverses p q .invl = p
make-inverses p q .invr = q

make-invertible : {f : Hom a b} → (g : Hom b a) → f ∘ g ≡ id → g ∘ f ≡ id → is-invertible f
make-invertible g p q .is-invertible.inv = g
make-invertible g p q .is-invertible.inverses .invl = p
make-invertible g p q .is-invertible.inverses .invr = q

make-iso : (f : Hom a b) (g : Hom b a) → f ∘ g ≡ id → g ∘ f ≡ id → a ≅ b
make-iso f g p q ._≅_.to = f
make-iso f g p q ._≅_.from = g
make-iso f g p q ._≅_.inverses .Inverses.invl = p
make-iso f g p q ._≅_.inverses .Inverses.invr = q

instance
  H-Level-is-invertible : ∀ {f : Hom a b} {n} → H-Level (is-invertible f) (suc n)
  H-Level-is-invertible = prop-instance is-invertible-is-prop

inverses→invertible : ∀ {f : Hom a b} {g : Hom b a} → Inverses f g → is-invertible f
inverses→invertible x .is-invertible.inv = _
inverses→invertible x .is-invertible.inverses = x

invertible→iso : (f : Hom a b) → is-invertible f → a ≅ b
invertible→iso f x =
  record
    { to       = f
    ; from     = x .is-invertible.inv
    ; inverses = x .is-invertible.inverses
    }

is-invertible-inverse
  : {f : Hom a b} (g : is-invertible f) → is-invertible (g .is-invertible.inv)
is-invertible-inverse g =
  record { inv = _ ; inverses = record { invl = invr g ; invr = invl g } }
  where open Inverses (g .is-invertible.inverses)

iso→invertible : (i : a ≅ b) → is-invertible (i ._≅_.to)
iso→invertible i = record { inv = i ._≅_.from ; inverses = i ._≅_.inverses }

≅-is-set : is-set (a ≅ b)
≅-is-set x y p q = s where
  open _≅_
  open Inverses

  s : p ≡ q
  s i j .to = Hom-set _ _ (x .to) (y .to) (ap to p) (ap to q) i j
  s i j .from = Hom-set _ _ (x .from) (y .from) (ap from p) (ap from q) i j
  s i j .inverses =
    is-prop→squarep
      (λ i j → Inverses-are-prop {f = Hom-set _ _ (x .to) (y .to) (ap to p) (ap to q) i j}
                                 {g = Hom-set _ _ (x .from) (y .from) (ap from p) (ap from q) i j})
      (λ i → x .inverses) (λ i → p i .inverses) (λ i → q i .inverses) (λ i → y .inverses) i j

private
  ≅-pathp-internal
    : (p : a ≡ c) (q : b ≡ d)
    → {f : a ≅ b} {g : c ≅ d}
    → PathP (λ i → Hom (p i) (q i)) (f ._≅_.to) (g ._≅_.to)
    → PathP (λ i → Hom (q i) (p i)) (f ._≅_.from) (g ._≅_.from)
    → PathP (λ i → p i ≅ q i) f g
  ≅-pathp-internal p q r s i .to = r i
  ≅-pathp-internal p q r s i .from = s i
  ≅-pathp-internal p q {f} {g} r s i .inverses =
    is-prop→pathp (λ j → Inverses-are-prop {f = r j} {g = s j})
      (f .inverses) (g .inverses) i

abstract
  inverse-unique
    : {x y : Ob} (p : x ≡ y) {b d : Ob} (q : b ≡ d) {f : x ≅ b} {g : y ≅ d}
    → PathP (λ i → Hom (p i) (q i)) (f .to) (g .to)
    → PathP (λ i → Hom (q i) (p i)) (f .from) (g .from)
  inverse-unique =
    J′ (λ a c p → ∀ {b d} (q : b ≡ d) {f : a ≅ b} {g : c ≅ d}
      → PathP (λ i → Hom (p i) (q i)) (f .to) (g .to)
      → PathP (λ i → Hom (q i) (p i)) (f .from) (g .from))
      λ x → J′ (λ b d q → {f : x ≅ b} {g : x ≅ d}
                → PathP (λ i → Hom x (q i)) (f .to) (g .to)
                → PathP (λ i → Hom (q i) x) (f .from) (g .from))
            λ y {f} {g} p →
              f .from                     ≡˘⟨ ap (f .from ∘_) (g .invl) ∙ idr _ ⟩
              f .from ∘ g .to ∘ g .from   ≡⟨ assoc _ _ _ ⟩
              (f .from ∘ g .to) ∘ g .from ≡⟨ ap (_∘ g .from) (ap (f .from ∘_) (sym p) ∙ f .invr) ∙ idl _ ⟩
              g .from                     ∎

≅-pathp
  : (p : a ≡ c) (q : b ≡ d) {f : a ≅ b} {g : c ≅ d}
  → PathP (λ i → Hom (p i) (q i)) (f ._≅_.to) (g ._≅_.to)
  → PathP (λ i → p i ≅ q i) f g
≅-pathp p q {f = f} {g = g} r = ≅-pathp-internal p q r (inverse-unique p q {f = f} {g = g} r)

↪-pathp
  : {a : I → Ob} {b : I → Ob} {f : a i0 ↪ b i0} {g : a i1 ↪ b i1}
  → PathP (λ i → Hom (a i) (b i)) (f .mor) (g .mor)
  → PathP (λ i → a i ↪ b i) f g
↪-pathp {a = a} {b} {f} {g} pa = go where
  go : PathP (λ i → a i ↪ b i) f g
  go i .mor = pa i
  go i .monic {c = c} =
    is-prop→pathp
      (λ i → Π-is-hlevel {A = Hom c (a i)} 1
       λ g → Π-is-hlevel {A = Hom c (a i)} 1
       λ h → fun-is-hlevel {A = pa i ∘ g ≡ pa i ∘ h} 1
              (Hom-set c (a i) g h))
      (f .monic)
      (g .monic)
      i

↠-pathp
  : {a : I → Ob} {b : I → Ob} {f : a i0 ↠ b i0} {g : a i1 ↠ b i1}
  → PathP (λ i → Hom (a i) (b i)) (f .mor) (g .mor)
  → PathP (λ i → a i ↠ b i) f g
↠-pathp {a = a} {b} {f} {g} pa = go where
  go : PathP (λ i → a i ↠ b i) f g
  go i .mor = pa i
  go i .epic {c = c} =
    is-prop→pathp
      (λ i → Π-is-hlevel {A = Hom (b i) c} 1
       λ g → Π-is-hlevel {A = Hom (b i) c} 1
       λ h → fun-is-hlevel {A = g ∘ pa i ≡ h ∘ pa i} 1
              (Hom-set (b i) c g h))
      (f .epic)
      (g .epic)
      i
```
-->

```agda
id-invertible : ∀ {a} → is-invertible (id {a})
id-invertible .is-invertible.inv = id
id-invertible .is-invertible.inverses .invl = idl id
id-invertible .is-invertible.inverses .invr = idl id

id-iso : a ≅ a
id-iso .to = id
id-iso .from = id
id-iso .inverses .invl = idl id
id-iso .inverses .invr = idl id

Isomorphism = _≅_

Inverses-∘ : {f : Hom a b} {f⁻¹ : Hom b a} {g : Hom b c} {g⁻¹ : Hom c b}
           → Inverses f f⁻¹ → Inverses g g⁻¹ → Inverses (g ∘ f) (f⁻¹ ∘ g⁻¹)
Inverses-∘ {f = f} {f⁻¹} {g} {g⁻¹} finv ginv = record { invl = l ; invr = r } where
  module finv = Inverses finv
  module ginv = Inverses ginv

  abstract
    l : (g ∘ f) ∘ f⁻¹ ∘ g⁻¹ ≡ id
    l = (g ∘ f) ∘ f⁻¹ ∘ g⁻¹ ≡⟨ cat! C ⟩
        g ∘ (f ∘ f⁻¹) ∘ g⁻¹ ≡⟨ (λ i → g ∘ finv.invl i ∘ g⁻¹) ⟩
        g ∘ id ∘ g⁻¹        ≡⟨ cat! C ⟩
        g ∘ g⁻¹             ≡⟨ ginv.invl ⟩
        id                  ∎

    r : (f⁻¹ ∘ g⁻¹) ∘ g ∘ f ≡ id
    r = (f⁻¹ ∘ g⁻¹) ∘ g ∘ f ≡⟨ cat! C ⟩
        f⁻¹ ∘ (g⁻¹ ∘ g) ∘ f ≡⟨ (λ i → f⁻¹ ∘ ginv.invr i ∘ f) ⟩
        f⁻¹ ∘ id ∘ f        ≡⟨ cat! C ⟩
        f⁻¹ ∘ f             ≡⟨ finv.invr ⟩
        id                  ∎

_∘Iso_ : a ≅ b → b ≅ c → a ≅ c
(f ∘Iso g) .to = g .to ∘ f .to
(f ∘Iso g) .from = f .from ∘ g .from
(f ∘Iso g) .inverses = Inverses-∘ (f .inverses) (g .inverses)

_Iso⁻¹ : a ≅ b → b ≅ a
(f Iso⁻¹) .to = f .from
(f Iso⁻¹) .from = f .to
(f Iso⁻¹) .inverses .invl = f .inverses .invr
(f Iso⁻¹) .inverses .invr = f .inverses .invl
```

We also note that invertible morphisms are both epic and monic.

```agda
invertible→monic : is-invertible f → is-monic f
invertible→monic {f = f} invert g h p =
  g             ≡˘⟨ idl g ⟩
  id ∘ g        ≡˘⟨ ap (_∘ g) (is-invertible.invr invert) ⟩
  (inv ∘ f) ∘ g ≡˘⟨ assoc inv f g ⟩
  inv ∘ (f ∘ g) ≡⟨ ap (inv ∘_) p ⟩
  inv ∘ (f ∘ h) ≡⟨ assoc inv f h ⟩
  (inv ∘ f) ∘ h ≡⟨ ap (_∘ h) (is-invertible.invr invert) ⟩
  id ∘ h        ≡⟨ idl h ⟩
  h ∎
  where
    open is-invertible invert

invertible→epic : is-invertible f → is-epic f
invertible→epic {f = f} invert g h p =
  g             ≡˘⟨ idr g ⟩
  g ∘ id        ≡˘⟨ ap (g ∘_) (is-invertible.invl invert) ⟩
  g ∘ (f ∘ inv) ≡⟨ assoc g f inv ⟩
  (g ∘ f) ∘ inv ≡⟨ ap (_∘ inv) p ⟩
  (h ∘ f) ∘ inv ≡˘⟨ assoc h f inv ⟩
  h ∘ (f ∘ inv) ≡⟨ ap (h  ∘_) (is-invertible.invl invert) ⟩
  h ∘ id        ≡⟨ idr h ⟩
  h ∎
  where
    open is-invertible invert

iso→monic : (f : a ≅ b) → is-monic (f .to)
iso→monic f = invertible→monic (iso→invertible f)

iso→epic : (f : a ≅ b) → is-epic (f .to)
iso→epic f = invertible→epic (iso→invertible f)
```

Furthermore, isomorphisms also yield section/retraction pairs.

```agda
inverses→to-has-section
  : ∀ {f : Hom a b} {g : Hom b a}
  → Inverses f g → has-section f
inverses→to-has-section {g = g} inv .section = g
inverses→to-has-section inv .is-section = Inverses.invl inv

inverses→from-has-section
  : ∀ {f : Hom a b} {g : Hom b a}
  → Inverses f g → has-section g
inverses→from-has-section {f = f} inv .section = f
inverses→from-has-section inv .is-section = Inverses.invr inv

inverses→to-has-retract
  : ∀ {f : Hom a b} {g : Hom b a}
  → Inverses f g → has-retract f
inverses→to-has-retract {g = g} inv .retract = g
inverses→to-has-retract inv .is-retract = Inverses.invr inv

inverses→from-has-retract
  : ∀ {f : Hom a b} {g : Hom b a}
  → Inverses f g → has-retract g
inverses→from-has-retract {f = f} inv .retract = f
inverses→from-has-retract inv .is-retract = Inverses.invl inv

iso→to-has-section : (f : a ≅ b) → has-section (f .to)
iso→to-has-section f .section = f .from
iso→to-has-section f .is-section = f .invl

iso→from-has-section : (f : a ≅ b) → has-section (f .from)
iso→from-has-section f .section = f .to
iso→from-has-section f .is-section = f .invr

iso→to-has-retract : (f : a ≅ b) → has-retract (f .to)
iso→to-has-retract f .retract = f .from
iso→to-has-retract f .is-retract = f .invr

iso→from-has-retract : (f : a ≅ b) → has-retract (f .from)
iso→from-has-retract f .retract = f .to
iso→from-has-retract f .is-retract = f .invl
```
