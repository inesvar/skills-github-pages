---
title: "(Unsigned) Distance For `i32`"
date: 2025-06-07
category: 32-bit-math-surprises
tags: rust, geometry
---

# `fn unsigned_distance(a: i32, b: i32) → u32`

Suppose we use the following objects :

```rust
struct Point {
	x: i32,
	y: i32,
}

struct Size {
	x: u32,
	y: u32,
}
```

The distance between two `Point` types can be described either by a `Point` or a `Size`. However the visual length of a line between two `Point` types actually doesn’t match any of these “intuitive” computations.

| `a: i32` | `b: i32`  | `b - a` (*) | `(b - a).abs()` | `(b - a) as u32` | visual length of the line `(max - min) as u32` |
| --- | --- | --- | --- | --- | --- |
| `0` | `1` | `1` | `1` | `1` | `1` |
| `1` | `0` | `-1` | `1` | `4294967295` | `1` |
| `i32::MIN` | `i32::MAX` | `-1` | `1` | `4294967295` | `4294967295` |
| `i32::MAX` | `i32::MIN` | `1` | `1` | `1` | `4294967295` |

(*) wrapping arithmetic is used here

While computing the visual length of the line is simple in theory (when using *an infinite amount of integers*), it is more complicated using only 32-bit integers.

$$
l = |b - a|
$$

⇒ When you want your arithmetic to be correct not only on half of `i32` but on the entirety of it, math gets really counter-intuitive.

One “easy solution” is using checked arithmetic.

| `a: i32` | `b: i32`  | `b - a`(*)  | `(b - a).abs()` | `(b - a) as u32` | visual length of the line `(max - min) as u32` |
| --- | --- | --- | --- | --- | --- |
| `0` | `1` | `1` | `1` | `1` | `1` |
| `1` | `0` | `-1` | `1` | `4294967295` | `1` |
| `i32::MIN` | `i32::MAX` | overflow ! | overflow ! | overflow ! | overflow ! |
| `i32::MAX` | `i32::MIN` | overflow ! | overflow ! | overflow ! | overflow ! |

(*) checked arithmetic is used here

A code like this would work :

```rust
match b.overflowing_sub(a) {
	diff, true => diff.unsigned_abs().wrapping_neg(), // needs proof !
	diff, false => diff.unsigned_abs(), // trivial case
}
```