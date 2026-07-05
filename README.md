# HyperMorphic-Fountain-Codes
HyperMorphic Fountain Codes: rateless erasure coding where droplets decode only when they form a valid algebraic-geometric reconstruction corridor.


# HyperMorphic Fountain Codes

**Rateless erasure coding by algebraic-geometric corridor completion.**

HyperMorphic Fountain Codes, or **HFC**, are a prototype rateless coding primitive where data is recovered from a stream of generated droplets only when the collected droplets form a valid **fountain corridor**.

Classical fountain coding asks:

> Do we have enough independent packets to solve the system?

HFC asks:

> Do the collected droplets have enough finite-field rank **and** enough representation-lane span to form a valid algebraic-geometric reconstruction corridor?

---

## One-Line Summary

**HyperMorphic Fountain Codes are rateless erasure codes where droplets decode only when they form a valid algebraic-geometric reconstruction corridor.**

---

## Core Idea

Traditional fountain codes generate potentially unlimited packets, often called droplets. A receiver collects enough droplets until the original data can be reconstructed.

HFC keeps the rateless idea, but adds a HyperMorphic reconstruction rule:

```text
recoverable ⇔ rank is sufficient
              and lane span is sufficient
              and corridor score exceeds threshold
              and data invariant verifies
```

So decoding is not only about packet count or rank. It also depends on whether the droplets span enough of the coding geometry.

---

## Primitive Lineage

```text
HTC → adaptive threshold coding
HCC → corridor-admissible recovery
HRC → regenerative corridor repair and geometry migration
HFC → rateless fountain coding by corridor completion
```

HFC extends the HyperMorphic coding stack into streaming and lossy transmission settings.

---

## What Makes HFC Different?

A classical fountain decoder accepts once enough linearly independent packets are available.

HFC accepts only when the collected droplet set satisfies:

1. **Finite-field rank**

   * The coefficient matrix must have enough rank to solve for all source symbols.

2. **Representation-lane coverage**

   * Droplets must span enough geometry lanes.

3. **Corridor score**

   * Lane entropy, angular spread, rank ratio, degree density, and entropy span combine into a corridor score.

4. **Data invariant verification**

   * The reconstructed payload must pass an HMAC-based invariant check in the prototype.

This means two droplet sets with the same count can behave differently:

```text
same droplet count
different lane span
different decoding outcome
```

---

## Formal Object

An HFC system is parameterized by:

```text
HFC(D, E_levels, symbol_bytes)
```

Where:

| Parameter      | Meaning                                               |
| -------------- | ----------------------------------------------------- |
| `D`            | number of representation lanes / geometry axes        |
| `E_levels`     | entropy quantization levels                           |
| `symbol_bytes` | bytes per source symbol                               |
| `GF_P`         | finite field modulus, using GF(257) in this prototype |

Data is split into source symbols:

```text
x_0, x_1, ..., x_{K-1}
```

Each emitted droplet is a finite-field linear combination:

```text
y_s = Σ a_{s,j} x_j  over GF(257)
```

The coefficients are generated deterministically from:

```text
seed, entropy level, K
```

Each droplet also has a representation lane:

```text
lane(seed) = seed mod D
```

---

## Encoding

HFC is rateless. You can generate as many droplets as needed:

```python
hfc = HyperMorphicFountainCode()

droplets = hfc.encode(data, droplet_count=200)
```

Or emit an infinite stream:

```python
stream = hfc.droplet_stream(data)

droplet_0 = next(stream)
droplet_1 = next(stream)
droplet_2 = next(stream)
```

Each droplet contains:

* seed
* lane
* entropy level
* degree
* finite-field payload vector
* source symbol count `K`
* data length
* data invariant HMAC
* droplet HMAC

---

## Decoding

The decoder collects droplets and searches for the first completed fountain corridor:

```python
recovered, cert = hfc.decode_with_certificate(droplets)
```

A droplet set is accepted only if:

```text
rank(A) ≥ K
lanes(S) ≥ required_lanes(state)
corridor_score(S) ≥ required_score(state)
data_hmac(recovered) verifies
```

Where:

* `A` is the droplet coefficient matrix
* `K` is the number of source symbols
* `S` is the collected droplet set
* `state` is estimated from entropy level and droplet distribution

---

## Corridor Predicate

The HFC corridor predicate is:

```text
F(S) = 1
```

iff:

```text
rank(A_S) ≥ K
and lane_count(S) ≥ L_required
and σ(S) ≥ τ
and Verify(D(S)) = true
```

Where:

| Symbol   | Meaning                                       |
| -------- | --------------------------------------------- |
| `S`      | collected droplet set                         |
| `A_S`    | coefficient matrix induced by droplets in `S` |
| `K`      | number of original source symbols             |
| `σ(S)`   | corridor score                                |
| `τ`      | required score threshold                      |
| `D(S)`   | decoded payload from droplet set              |
| `Verify` | data invariant check                          |

---

## Corridor Score

The prototype corridor score combines:

```text
rank_ratio
lane_coverage
lane_entropy
angular_spread
degree_density
entropy_span
```

Conceptually:

```text
corridor_score =
    rank strength
  + geometric span
  + lane diversity
  + angular spread
  + mixing density
  + entropy-state diversity
```

This makes HFC not just a rank decoder, but a geometry-aware fountain decoder.

---

## Demonstrated Novel Behaviour

HFC demonstrates the same-count / different-geometry distinction:

```text
Set A: enough droplets, enough rank, full lane span → accepted
Set B: same droplet count, clustered lanes → rejected
```

The important distinction is:

```text
classical fountain code: enough independent droplets
HFC: enough independent droplets in a valid corridor
```

---

## Features

* **Rateless droplet generation**

  * Generate as many droplets as needed.

* **GF(257) linear mixing**

  * Byte values embed directly into the finite field.

* **Corridor-admissible decoding**

  * Recovery requires rank plus geometric invariant span.

* **Streaming recovery**

  * Decode can complete as soon as the stream forms a valid corridor.

* **Same-count subset distinction**

  * Equal-size droplet sets can differ by geometry and acceptance.

* **Loss tolerance**

  * Random droplet loss can be handled by collecting more droplets.

* **Tamper filtering**

  * Prototype HMAC verification skips corrupted droplets.

* **Certificate output**

  * Decoder reports rank, droplet count, overhead, lane distribution, corridor score, and degree distribution.

* **Pure Python prototype**

  * No external dependencies required.

---

## Example Test Results

The included demo validates:

```text
basic fountain roundtrip
same-count different-geometry rejection
30% random droplet loss recovery
tamper filtering
streaming corridor completion
certificate generation
```

Example output:

```text
ALL PASS — HFC primitive validated
```

---

## Quick Start

Clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/hypermorphic-fountain-codes.git
cd hypermorphic-fountain-codes
```

Run the demo:

```bash
python hfc.py
```

Or paste the full code into Google Colab and run the cell.

No external dependencies are required beyond the Python standard library.

---

## Minimal Usage

```python
from hfc import HyperMorphicFountainCode

hfc = HyperMorphicFountainCode(
    geometry_dim=4,
    E_levels=9,
    symbol_bytes=16,
)

data = b"HyperMorphic Fountain Codes"

droplets = hfc.encode(data, droplet_count=120)

recovered = hfc.decode(droplets)

assert recovered == data
```

---

## Decode With Certificate

```python
recovered, certificate = hfc.decode_with_certificate(droplets)

print(certificate)
```

Example certificate fields:

```python
{
    "mode": "incremental",
    "K": 64,
    "droplets_used": 91,
    "overhead_ratio": 1.4219,
    "rank": 64,
    "lanes": 4,
    "corridor_score": 0.87,
    "required_lanes": 4,
    "lane_distribution": {0: 23, 1: 23, 2: 23, 3: 22},
    "degree_distribution": {...},
    "seed_range": [0, 90]
}
```

The certificate shows not just that decoding worked, but why the collected droplets formed an admissible reconstruction corridor.

---

## Streaming Example

```python
hfc = HyperMorphicFountainCode()

data = b"streaming fountain payload" * 100

stream = hfc.droplet_stream(data)

collected = []

for _ in range(300):
    collected.append(next(stream))

    try:
        recovered, cert = hfc.decode_with_certificate(collected)
        break
    except Exception:
        pass

assert recovered == data

print("corridor completed after", cert["droplets_used"], "droplets")
```

---

## Droplet Loss Example

```python
data = b"loss-tolerant payload" * 200

hfc = HyperMorphicFountainCode()

droplets = hfc.encode(data, droplet_count=260)

kept = hfc.drop_random(droplets, drop_fraction=0.30, seed=42)

recovered, cert = hfc.decode_with_certificate(kept)

assert recovered == data

print(cert["droplets_used"])
print(cert["overhead_ratio"])
```

---

## Tamper Filtering Example

```python
data = b"tamper filtering payload" * 100

hfc = HyperMorphicFountainCode()

droplets = hfc.encode(data, droplet_count=180)

corrupted = hfc.corrupt_droplets(droplets, [0, 3, 8])

recovered, cert = hfc.decode_with_certificate(corrupted)

assert recovered == data

print(cert["valid_droplets_available"])
```

In this prototype, corrupted droplets fail HMAC validation and are skipped.

---

## Comparison

| Property                     | Classical Fountain Code | HyperMorphic Fountain Code |
| ---------------------------- | ----------------------: | -------------------------: |
| Rateless packet generation   |                     yes |                        yes |
| Linear-system recovery       |                     yes |                        yes |
| Rank requirement             |                     yes |                        yes |
| Geometry-lane requirement    |                      no |                        yes |
| Corridor score               |                      no |                        yes |
| Same-count sets can differ   |              usually no |                        yes |
| Streaming completion         |                     yes |                        yes |
| Tamper filtering             |                external |             prototype HMAC |
| Certificate of recovery path |                      no |                        yes |
| HyperMorphic invariant span  |                      no |                        yes |

---

## What This Is Not Claiming

This prototype does **not** claim:

* production readiness
* cryptographic security
* replacement of Raptor/LT codes
* optimal overhead
* formal Byzantine security
* information-theoretic superiority
* network-code optimality
* peer-reviewed coding-theory proof

The defensible claim is:

> HFC demonstrates a rateless coding prototype where decoding is gated by both finite-field rank and algebraic-geometric corridor span.

---

## Suggested Formal Claim

**Fountain Corridor Completion**

Given a droplet set `S`, HFC accepts decoding only if:

```text
rank(A_S) ≥ K
```

and the droplet geometry satisfies a corridor predicate:

```text
C(S) = 1
```

Therefore, HFC recovery is not reducible to droplet count alone.

**Cardinality Non-Equivalence**

There exist droplet sets `S1` and `S2` such that:

```text
|S1| = |S2|
```

but:

```text
C(S1) = 1
C(S2) = 0
```

So equal-cardinality streams may differ in admissibility depending on invariant span.

---

## Roadmap

Planned improvements:

* add LT-code baseline
* add Raptor-style precode
* add Reed–Solomon comparison
* add HCC/HRC comparison benchmark
* add network loss simulations
* add burst-loss and lane-cluster-loss tests
* optimize GF(257) Gaussian elimination
* add sparse peeling decoder mode
* add visualization of corridor completion
* add droplet serialization format
* add file transfer demo
* add formal proof section
* add Colab notebook with plots

---

## Suggested Paper Title

**HyperMorphic Fountain Codes: Rateless Erasure Coding by Algebraic-Geometric Corridor Completion**

---

## Suggested Citation

```bibtex
@misc{hypermorphic_fountain_codes,
  title  = {HyperMorphic Fountain Codes: Rateless Erasure Coding by Algebraic-Geometric Corridor Completion},
  author = {Gerrard, Shaun Paul},
  year   = {2026},
  note   = {Research prototype}
}
```

---

## License

Choose a license before release.

Recommended options:

* **MIT** for maximum openness
* **Apache-2.0** for permissive use with patent-related clarity
* **AGPL-3.0** if you want network-use modifications to remain open

---

## Bottom Line

HFC turns fountain recovery into corridor completion:

```text
droplets stream in
rank grows
geometry span grows
corridor completes
data reconstructs
```

The central idea is:

> data returns when enough algebraic rank and enough invariant geometry survive in the stream.
