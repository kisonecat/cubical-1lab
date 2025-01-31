```agda
open import Algebra.Semilattice
open import Algebra.Semigroup
open import Algebra.Prelude
open import Algebra.Magma

open import Cat.Displayed.Univalence.Thin

open import Principles.Resizing

import Cat.Reasoning

module Algebra.Frame where
```

# Frames

<!--
```agda
private variable
  ℓ ℓ′ : Level
  A B : Type ℓ
```
-->

A **frame** is a lattice with binary meets and arbitrary joins
satisfying the _infinite distributive_ law

$$
x \cap \bigcup_i f(i) = \bigcup_i (x \cap f(i))\text{.}
$$

In the study of frames, for simplicity, we assume propositional
`resizing`{.Agda}: that way, it suffices for a frame $A$ to have meets
of $\ca{J}$-indexed families, for $\ca{J}$ an arbitrary type in the same
universe as $A$, to have joins for arbitrary subsets of $A$.

```agda
record is-frame
  (_∩_ : A → A → A)
  (⋃ : ∀ {I : Type (level-of A)} → (I → A) → A)
  : Type (lsuc (level-of A)) where
  field
    has-is-slat : is-semilattice _∩_
```

<!--
```agda
  open is-semilattice has-is-slat public
  _≤_ : A → A → Type _
  x ≤ y = x ≡ x ∩ y
```
-->

As usual, we define an ordering on $A$ in terms of the binary meets as
$x \le y$ iff $x \equiv x \cap y$. The properties of the join operator
are then defined in terms of this ordering, rather than being defined
algebraically. Thus, we have a mixed order-theoretic and algebraic
presentation of frames.

```agda
  field
    ⋃-universal  : ∀ {I} {x} (f : I → A) → (∀ i → f i ≤ x) → ⋃ f ≤ x
    ⋃-colimiting : ∀ {I} i (f : I → A) → f i ≤ ⋃ f
    ⋃-distrib    : ∀ {I} x (f : I → A) → x ∩ ⋃ f ≡ ⋃ λ i → x ∩ f i
```

<!--
```agda
  module P = Poset (Semilattice-on→Meet-on has-is-slat) renaming (_∘_ to trans)
  open P using (trans) public

record Frame-on (A : Type ℓ) : Type (lsuc ℓ) where
  field
    _∩_ : A → A → A
    ⋃   : ∀ {I : Type (level-of A)} → (I → A) → A
    has-is-frame : is-frame _∩_ ⋃
  open is-frame has-is-frame public
```
-->

Frames are, of course, complete lattices (and thus, also Heyting
algebras). The difference in naming comes from the morphisms with which
frames are considered: A frame homomorphism need only preserve the
binary meets and arbitrary joins, and it does not need to preserve
infinitary meets (or the Heyting implication).

```agda
record
  is-frame-hom {A B : Type ℓ} (f : A → B) (X : Frame-on A) (Y : Frame-on B)
    : Type (lsuc ℓ) where
  private
    module X = Frame-on X
    module Y = Frame-on Y
  field
    pres-∩ : ∀ x y → f (x X.∩ y) ≡ (f x Y.∩ f y)
    pres-⋃ : ∀ {I} (g : I → A) → f (X.⋃ g) ≡ Y.⋃ λ i → f (g i)
```

<!--
```agda
private unquoteDecl eqv = declare-record-iso eqv (quote is-frame-hom)
private unquoteDecl eqv′ = declare-record-iso eqv′ (quote is-frame)

open Thin-structure
open is-frame-hom
```
-->

Frame homomorphisms still look like homomorphisms of algebraic
structures, though, and so our usual machinery for constructing
categories of "sets-with-structure" applies here.

```agda
Frame-str : ∀ ℓ → Thin-structure {ℓ = ℓ} _ Frame-on
Frame-str ℓ .is-hom f x y .∣_∣   = is-frame-hom f x y
Frame-str ℓ .is-hom f x y .is-tr = is-hlevel≃ 1 (Iso→Equiv eqv e⁻¹) (hlevel 1)
  where instance
    ahl : H-Level _ 2
    ahl = hlevel-instance (Frame-on.has-is-set y)
Frame-str ℓ .id-is-hom .pres-∩ x y = refl
Frame-str ℓ .id-is-hom .pres-⋃ g = refl
Frame-str ℓ .∘-is-hom f g α β .pres-∩ x y = ap f (β .pres-∩ _ _) ∙ α .pres-∩ _ _
Frame-str ℓ .∘-is-hom f g α β .pres-⋃ h = ap f (β .pres-⋃ _) ∙ α .pres-⋃ _
Frame-str ℓ .id-hom-unique α β i .Frame-on._∩_ a b = α .pres-∩ a b i
Frame-str ℓ .id-hom-unique α β i .Frame-on.⋃ f = α .pres-⋃ f i
```

<!--
```agda
Frame-str ℓ .id-hom-unique {s = s} {t} α β i .Frame-on.has-is-frame =
  is-prop→pathp (λ i → lemma (λ a b → α .pres-∩ a b i) (λ f → α .pres-⋃ f i))
    (s .Frame-on.has-is-frame)
    (t .Frame-on.has-is-frame) i
  where
  lemma : ∀ a (b : ∀ {I} → (I → A) → A) → is-prop (is-frame a b)
  lemma {A = A} a b x = is-hlevel≃ 1 (Iso→Equiv eqv′ e⁻¹) (hlevel 1) x where instance
    ahl : H-Level A 2
    ahl = hlevel-instance (is-frame.has-is-set x)

Frames : ∀ ℓ → Precategory _ _
Frames a = Structured-objects (Frame-str a)

module Frames ℓ = Cat.Reasoning (Frames ℓ)
```
-->

## Joins of subsets

Imagine you have a frame $A$ whose carrier has size $\kappa$, and thus
has joins for $\kappa$-small families of elements. But imagine that you
have a second universe $\lambda$, and you have a $\lambda$-small
predicate $P : \bb{P}_{\lambda}(A)$. Normally, you'd be out of luck:
there is no reason for $A$ to admit $(\kappa \sqcup \lambda)$-sized
joins.

But since we've assumed the existence of $\Omega$, we can resize
(pointwise) $P$ to be valued in the universe $\kappa$; That way we can
turn the total space $\int P$ into a $\kappa$-small type! By projecting
the first component, we have a $\kappa$-small family of elements, which
has a join. This is a good definition for the **join of the subset
$P$**.

```agda
module _ {ℓ} (F : Frames.Ob ℓ) where
  private module F = Frame-on (F .snd)
  subset-cup : ∀ {ℓ′} (P : ⌞ F ⌟ → Prop ℓ′) → ⌞ F ⌟
  subset-cup P = F.⋃
    {I = Σ[ t ∈ ⌞ F ⌟ ] (resize ℓ ∣ P t ∣ (P t .is-tr) .fst)}
    λ { (x , _) → x }

  subset-cup-colimiting
    : ∀ {ℓ′} (P : ⌞ F ⌟ → Prop ℓ′) {x}
    → ∣ P x ∣ → x F.≤ subset-cup P
  subset-cup-colimiting P x =
    F.⋃-colimiting (_ , Equiv.from (resize _ _ _ .snd) x) λ { (f , w) → f }

  subset-cup-universal
    : ∀ {ℓ′} (P : ⌞ F ⌟ → Prop ℓ′) {x}
    → (∀ i → ∣ P i ∣ → i F.≤ x)
    → subset-cup P F.≤ x
  subset-cup-universal P f =
    F.⋃-universal fst λ { (i , w) → f i (resize _ _ _ .snd .fst w) }
```

Keep imagining that you have a subset $P \sube A$: Can we construct a
meet for it? Yes! By taking the join of all possible upper bounds for
$P$, we get the a lower bound among upper bounds of $P$: a meet for $P$.

```agda
  subset-cap : ∀ {ℓ′} (P : ⌞ F ⌟ → Prop ℓ′) → ⌞ F ⌟
  subset-cap P = subset-cup λ x → el (∀ a → ∣ P a ∣ → x F.≤ a) hlevel!

  subset-cap-limiting
    : ∀ {ℓ′} (P : ⌞ F ⌟ → Prop ℓ′) {x} → ∣ P x ∣ → subset-cap P F.≤ x
  subset-cap-limiting P x∈P =
    subset-cup-universal (λ x → el _ _) λ i a∈P→i≤a → a∈P→i≤a _ x∈P

  subset-cap-universal
    : ∀ {ℓ} (P : ⌞ F ⌟ → Prop ℓ) {x}
    → (∀ i → ∣ P i ∣ → x F.≤ i)
    → x F.≤ subset-cap P
  subset-cap-universal P x∈P = subset-cup-colimiting (λ _ → el _ _) x∈P
```

<!--
```agda
open Frame-on

open is-semilattice
open is-frame

record make-frame {ℓ} (A : Type ℓ) : Type (lsuc ℓ) where
  field
    has-is-set : is-set A
    _cap_ : A → A → A
    cup   : ∀ {I : Type ℓ} → (I → A) → A
    idempotent  : ∀ {a} → a cap a ≡ a
    commutative : ∀ {a b} → a cap b ≡ b cap a
    associative : ∀ {a b c} → a cap (b cap c) ≡ (a cap b) cap c

  _le_ : A → A → Type _
  x le y = x ≡ x cap y
  field
    universal  : ∀ {I} {x} (f : I → A) → (∀ i → f i le x) → cup f le x
    colimiting : ∀ {I} i (f : I → A) → f i le cup f
    distrib    : ∀ {I} x (f : I → A) → x cap cup f ≡ cup λ i → x cap f i

open make-frame
open is-magma
to-frame-on : ∀ {ℓ} {A : Type ℓ} → make-frame A → Frame-on A
to-frame-on mfr ._∩_ = mfr ._cap_
to-frame-on mfr .⋃ = mfr .cup
to-frame-on mfr .has-is-frame .has-is-slat .has-is-semigroup .has-is-magma .has-is-set = mfr .has-is-set
to-frame-on mfr .has-is-frame .has-is-slat .has-is-semigroup .associative = mfr .associative
to-frame-on mfr .has-is-frame .has-is-slat .commutative = mfr .commutative
to-frame-on mfr .has-is-frame .has-is-slat .idempotent = mfr .idempotent
to-frame-on mfr .has-is-frame .⋃-universal = mfr .universal
to-frame-on mfr .has-is-frame .⋃-colimiting = mfr .colimiting
to-frame-on mfr .has-is-frame .⋃-distrib = mfr .distrib
```
-->

# Power frames

A canonical source of frames are power sets: The power set of any type
is a frame, because it is a complete lattice satisfying the infinite
distributive law; This can be seen by some computation, as is done
below.

```agda
Power-frame : ∀ {ℓ} (A : Type ℓ) → Frames.Ob ℓ
Power-frame {ℓ = ℓ} A .fst = el (A → Ω) (hlevel 2)
Power-frame A .snd = to-frame-on go where
  go : make-frame (A → Ω)
  go .has-is-set = hlevel 2
  go ._cap_ f g x .∣_∣   = ∣ f x ∣ × ∣ g x ∣
  go ._cap_ f g x .is-tr = ×-is-hlevel 1 (f x .is-tr) (g x .is-tr)
  go .cup {I} P x = elΩ (∃ I λ i → ∣ P i x ∣) squash
  go .idempotent = funext λ i → Squish-prop-ua (bi fst λ x → x , x)
  go .commutative = funext λ i → Squish-prop-ua $ bi
    (λ { (x , y) → y , x }) (λ { (x , y) → y , x })
  go .associative = funext λ i → Squish-prop-ua $ bi
    (λ { (x , y , z) → (x , y) , z })
    (λ { ((x , y) , z) → x , y , z })
  go .universal {x = x} f W = funext λ i → elΩₗ-ua
    (∥-∥-rec (×-is-hlevel 1 (hlevel 1) (x i .is-tr)) λ (a , w) →
      true→elΩ (inc (_ , w)) , transport (ap ∣_∣ (W a $ₚ i)) w .snd)
    λ x → elΩ→true (x .fst)
  go .colimiting i f = funext λ j → Squish-prop-ua $ bi
    (λ x → x , true→elΩ (inc (_ , x))) fst
  go .distrib x f = funext λ i → sym $ elΩₗ-ua
    (∥-∥-rec (×-is-hlevel 1 (x i .is-tr) (hlevel 1))
      λ { (x , y , z) → y , true→elΩ (inc (_ , z)) })
    λ (x , i) → (λ (y , z) → _ , x , z) <$> elΩ→true i
```
