# vani-sparse

Sparse matrix format and operations library for the [vāṇी compiler](https://github.com/enthusiasticgeek/vani-compiler).

Depends on [vani-matrix](https://github.com/enthusiasticgeek/vani-matrix)
purely for testing/interop: every operation in this package is validated by
converting to/from a dense vani-matrix matrix and comparing against the
equivalent dense operation, the strongest correctness check available for
sparse code. `src/lib.vani` itself is otherwise self-contained.

**API reference / tutorial:** <https://enthusiasticgeek.github.io/vani-sparse/>

## Add to your project

```toml
# vani.toml
[deps]
sparse = { registry = "kosh", version = "^0.1" }
```

```sh
vanic add sparse
vanic build
```

## Encoding

Two struct types, both with explicit named fields (structs with `Vec`
fields are fully supported by the compiler):

```
struct SparseCOO { rows: Vec<i64>, cols: Vec<i64>, vals: Vec<f64>, n_rows: i64, n_cols: i64 }
struct SparseCSR { row_ptr: Vec<i64>, col_idx: Vec<i64>, vals: Vec<f64>, n_rows: i64, n_cols: i64 }
```

**COO** (coordinate list) is the easy-to-*build* format: push `(row, col,
val)` triples in any order via `sparse_coo_push`; duplicates at the same
position are summed when converted to CSR. **CSR** (compressed sparse row)
is the efficient format every operation works on — row `i`'s entries are
`col_idx[row_ptr[i]..row_ptr[i+1]]` / `vals[row_ptr[i]..row_ptr[i+1]]`,
sorted by column within each row. A CSR matrix's dense form (via
`sparse_csr_to_dense`) is byte-for-byte the same row-major layout
vani-matrix and vani-tensor use.

## What's included (v0.1.0 — complete; see TODO.md)

| Module | Functions |
|---|---|
| Construction | `sparse_coo_new`, `sparse_coo_push`, `sparse_identity_csr` |
| Conversion | `sparse_coo_to_csr`, `sparse_csr_to_dense`, `sparse_csr_from_dense` |
| Queries | `sparse_csr_nnz`, `sparse_csr_get`, `sparse_csr_diagonal` |
| Operations | `sparse_csr_matvec`, `sparse_csr_transpose`, `sparse_csr_scale`, `sparse_csr_add`, `sparse_csr_matmul` |

## sparse_csr_matmul uses Gustavson's algorithm

Row-by-row, with a dense length-`n_cols` accumulator reset after each
output row -- simple to get right at this library's "modest size" scope,
though the per-row reset costs `O(n_cols)` regardless of how sparse the
result actually is. A CSR-native merge-based algorithm without the dense
accumulator would scale better to very wide matrices; not implemented yet.

## Correctness

Every operation is cross-checked against the equivalent dense vani-matrix
operation (`mat_vec_mul`, `mat_transpose`, `mat_mul`) on the same data, not
just compared to a hand-picked expected value. See `tests/test_ops.vani`.

## What this library does NOT provide

These are already vāṇी compiler builtins — call them directly, no import needed:

`abs` `push` `pop` `len` `set` `vec`

## License

MIT
