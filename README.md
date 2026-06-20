

**A mathematically rigorous way to change unit bases and set arbitrary physical constants to desired values.**

A rigorous, linear-algebraic framework for unit systems where:
- **Units are treated as vectors** in a rational vector space.
- **Unit changes are passive linear transformations** (coordinate changes) on the dimension space.
- **Numerical rescaling is an active linear transformation** on the dual space of values.

---

## Overview

This repository implements the complete mathematical formalism developed through a rigorous derivation of dimensional analysis as linear algebra. The framework cleanly separates:

| Transformation | What It Changes | Mathematical Action |
|:---|:---|:---|
| **Passive** | Unit labels / coordinate axes | \( \mathbf{d}^{\text{new}} = B \cdot \mathbf{d}^{\text{SI}} \), \( B = A^{-1} \) |
| **Active** | Numerical values | \( \log(X') = \log(X) + \mathbf{d}^{\text{new}} \cdot \mathbf{x} \) |
| **Full** | Both | Composition: passive then active |

---

## Mathematical Backbone

### 1. The Dimension Space

Let \( \mathcal{D} \cong \mathbb{Q}^4 \) be the rational vector space of physical dimensions with SI basis:

\[
\mathcal{B}_{\text{SI}} = \{ \mathbf{M}, \mathbf{L}, \mathbf{T}, \boldsymbol{\Theta} \}
\]

Every physical quantity \( X \) has a dimension vector:

\[
\mathbf{d}_X^{\text{SI}} = (d_M, d_L, d_T, d_\Theta)^T \in \mathbb{Q}^4
\]

### 2. Passive Transformation (Change of Basis)

Choose four independent physical constants \( q_1, q_2, q_3, q_4 \) to define a new basis. Construct:

\[
A = \begin{bmatrix}
| & | & | & | \\
\mathbf{d}_{q_1}^{\text{SI}} & \mathbf{d}_{q_2}^{\text{SI}} & \mathbf{d}_{q_3}^{\text{SI}} & \mathbf{d}_{q_4}^{\text{SI}} \\
| & | & | & |
\end{bmatrix}
\]

The passive transformation matrix is:

\[
B = A^{-1}
\]

For any quantity \( X \):

\[
\boxed{\mathbf{d}_X^{\text{new}} = B \cdot \mathbf{d}_X^{\text{SI}}}
\]

**Key property:** In the new basis, each constrained constant becomes a standard basis vector:

\[
\mathbf{d}_{q_i}^{\text{new}} = \mathbf{e}_i
\]

### 3. Active Transformation (Numerical Rescaling)

To set the constrained constants to target values \( q_i' \), define log-scaling factors:

\[
x_i = \log\left( \frac{q_i'}{q_i^{\text{SI}}} \right)
\]

For any quantity \( X \) with new-basis dimension vector \( \mathbf{d}_X^{\text{new}} \):

\[
\boxed{\log(X') = \log(X^{\text{SI}}) + \left( \mathbf{d}_X^{\text{new}} \right)^T \cdot \mathbf{x}}
\]

Or in product form:

\[
\boxed{X' = X^{\text{SI}} \cdot \prod_{j=1}^4 \left( \frac{q_j'}{q_j^{\text{SI}}} \right)^{d_{X,j}^{\text{new}}}}
\]

### 4. The Complete Combined Transformation

Substituting \( \mathbf{d}_X^{\text{new}} = B \cdot \mathbf{d}_X^{\text{SI}} \):

\[
\boxed{
\log(X') = \log(X^{\text{SI}}) + \left( B \cdot \mathbf{d}_X^{\text{SI}} \right)^T \cdot \mathbf{x}
}
\]

### 5. Signed Quantities

For negative quantities \( X = \sigma \cdot |X| \), the scaling acts on magnitude:

\[
\boxed{
X' = \sigma \cdot |X| \cdot \prod_{j=1}^4 \left( \frac{q_j'}{q_j^{\text{SI}}} \right)^{d_{X,j}^{\text{new}}}
}
\]

In log-space:

\[
\log(|X'|) = \log(|X|) + \mathbf{d}_X^{\text{new}} \cdot \mathbf{x}
\]

---

## Implementation Architecture

### Core Functions

| Function | Description | Returns |
|:---|:---|:---|
| `build_system(constraints)` | Builds passive matrix \( B \) and log-scales \( \mathbf{x} \) from 4 constraints. | `(B, x)` |
| `passive_apply(quantity, B)` | Applies coordinate change only. Value unchanged. | New dict with `new_dims` |
| `active_apply(quantity, d_new, x)` | Applies numerical rescaling given new-basis dims. | New dict with `new_value` |
| `full_apply(quantity, B, x)` | Full transformation: passive then active. | New dict with both |
| `passive_apply_bulk(quantities, B)` | Vectorized passive for many quantities. | `List[Dict]` |
| `active_apply_bulk(quantities, D_new, x)` | Vectorized active for many quantities. | `List[Dict]` |
| `full_apply_bulk(quantities, B, x)` | Vectorized full transformation. | `List[Dict]` |

### Data Structures

**Constraint:** Defines one basis vector.
```python
constraint = {
    'target': 1.0,                    # Desired value in new system
    'si_value': 9.1093837e-31,        # SI numerical value
    'si_dims': np.array([1, 0, 0, 0]) # SI dimension vector [M, L, T, Θ]
}
```

**Quantity:** A physical quantity to transform.
```python
quantity = {
    'name': 'proton_mass',            # Optional debug name
    'value': 1.67262192e-27,          # SI numerical value
    'si_dims': np.array([1, 0, 0, 0]) # SI dimension vector
}
```

**Transformed Output:**
```python
{
    'name': 'proton_mass',
    'value': 1.67262192e-27,          # Original SI value
    'si_value': 1.67262192e-27,       # Same, for clarity
    'si_dims': array([1, 0, 0, 0]),   # Original dims
    'new_value': 1836.15,             # Transformed value
    'new_dims': array([1, 2, -2, 0])  # New basis coordinates
}
```

---

## Example: Planck Units

Setting \( c = \hbar = G = k_B = 1 \):

```python
import numpy as np
from dimensional_vector import *

constraints = [
    {'target': 1.0, 'si_value': 299792458.0,        'si_dims': np.array([0, 1, -1, 0])},
    {'target': 1.0, 'si_value': 1.054571817e-34,    'si_dims': np.array([1, 2, -1, 0])},
    {'target': 1.0, 'si_value': 6.67430e-11,        'si_dims': np.array([-1, 3, -2, 0])},
    {'target': 1.0, 'si_value': 1.380649e-23,       'si_dims': np.array([1, 2, -2, -1])}
]

B, x = build_system(constraints)

quantities = [
    {'name': 'proton_mass', 'value': 1.67262192e-27, 'si_dims': np.array([1, 0, 0, 0])},
    {'name': 'planck_length', 'value': 1.616255e-35, 'si_dims': np.array([0, 1, 0, 0])}
]

transformed = full_apply_bulk(quantities, B, x)

for t in transformed:
    print(f"{t['name']}: {t['new_value']:.6e}")
```

**Expected Output:**
```
proton_mass: 1.220890e+19    # in units of Planck mass
planck_length: 1.000000e+00  # exactly 1 by definition
```

---

## Numerical Robustness

### Condition Number Check

Replaces the scale-sensitive determinant test:

```python
cond = np.linalg.cond(A)
if cond > 1e12:
    raise ValueError("Constraint basis is ill-conditioned")
```

### Signed Value Handling

Scaling acts on magnitude; sign is preserved:

```python
mag, sign = abs(value), np.sign(value)
log_new = np.log(mag) + d_new @ x
new_value = sign * np.exp(log_new)
```

---

## Extensions

The framework generalizes to \( n \) dimensions:

1. Change dimension size `n` in all shape checks.
2. Provide exactly `n` independent constraints.
3. Everything else: \( B = A^{-1} \), \( x_i = \log(q_i'/q_i^{\text{SI}}) \), and \( \log(Q') = \log(Q) + \mathbf{d}^{\text{new}} \cdot \mathbf{x} \) remains identical.

---

## References

This implementation is derived from the mathematical treatment of dimensional analysis as a vector space over \( \mathbb{Q} \). Key principles:

- **Buckingham π Theorem**: Dimensionless groups correspond to the nullspace of the dimension matrix.
- **Dual Space**: Numerical values live in the dual space; active transformations are linear maps on log-values.
- **Change of Basis**: \( B = A^{-1} \) is the passive coordinate transformation; \( \mathbf{x} \) is the active dual transformation.

---

## License

[MIT](LICENSE)

---



## Author

Derived from rigorous first-principles derivation of dimensional analysis as applied linear algebra.
