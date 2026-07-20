# vani-sparse — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `abs` `push` `pop` `len` `set` `vec`
>
> Depends on vani-matrix -- used only for dense cross-check testing/interop,
> not by any function in src/lib.vani itself.

---

## v0.1.0 — Implemented ✓

### Construction (3 functions)
- [x] `sparse_coo_new`, `sparse_coo_push` -- build a matrix incrementally in
      COO form, duplicates allowed (summed on conversion to CSR)
- [x] `sparse_identity_csr` -- n x n identity, directly in CSR form

### Conversion (3 functions)
- [x] `sparse_coo_to_csr` -- insertion sort by (row, col), sum duplicates,
      build row_ptr by counting. Validated: duplicate entries at the same
      position sum correctly, row_ptr matches hand-computed values
- [x] `sparse_csr_to_dense`, `sparse_csr_from_dense` -- round-trip through
      vani-matrix's flat row-major layout, validated as an exact round trip

### Queries (3 functions)
- [x] `sparse_csr_nnz`, `sparse_csr_get`, `sparse_csr_diagonal`

### Operations (5 functions)
- [x] `sparse_csr_matvec` -- O(nnz), cross-checked against vani-matrix's
      `mat_vec_mul` on the same data
- [x] `sparse_csr_transpose` -- via COO round-trip, cross-checked against
      `mat_transpose`
- [x] `sparse_csr_scale` -- cross-checked against dense scalar multiply
- [x] `sparse_csr_add` -- via COO round-trip (concatenate entries, let
      duplicate-summing do the addition), cross-checked against `2*dense`
- [x] `sparse_csr_matmul` -- Gustavson's algorithm (dense per-row
      accumulator), cross-checked against `mat_mul` on the same data, plus
      a composed check: `A * identity == A`

### Tests and examples
- [x] `tests/test_coo_csr.vani` -- construction, duplicate-summing,
      row_ptr correctness, dense round trips, queries, identity
- [x] `tests/test_ops.vani` -- every operation cross-checked against its
      dense vani-matrix equivalent, plus the `A * I == A` composed check
- [x] `examples/sparse_linear_system_demo.vani` -- a sparse tridiagonal
      system (the kind vani-pde's 1D solvers produce), solved via
      densify + `mat_solve`, verified directly on the sparse matrix via
      `sparse_csr_matvec`
- [x] `examples/graph_adjacency_demo.vani` -- a directed graph's adjacency
      matrix, `A^2` computed entirely in sparse form to count 2-step paths

### Safety annotations
- [x] `#[bounded_stack(bytes=N)]` on every function, budgets set to `vanic
      check`'s exact reported worst-case (largest: `sparse_csr_matmul` at
      817 bytes, since it composes the dense accumulator loop with the
      full `sparse_coo_to_csr` chain)
- [x] No recursion anywhere in this library

---

## Future

No v0.2.0 is currently planned. Candidates if a concrete need shows up: a
CSR-native `sparse_csr_matmul` that avoids the dense per-row accumulator
(better scaling for very wide matrices), a sparse direct solver (e.g.
sparse LU or conjugate gradient for symmetric positive-definite systems --
`examples/sparse_linear_system_demo.vani` currently densifies and calls
vani-matrix's dense `mat_solve`, which defeats the point of sparsity for
genuinely large systems), a CSC (compressed sparse column) format for
column-oriented algorithms, and binary search in `sparse_csr_get` (currently
a linear scan within each row -- fine at this library's scope since
`col_idx` is already sorted per row if a faster lookup is ever needed).
