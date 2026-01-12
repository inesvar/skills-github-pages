---
title: "(Unsigned) Distance For `i32`"
date: 2025-06-07
category: 32-bit-math-surprises
tags: rust, geometry
---

# `fn unsigned_distance(a: i32, b: i32) → u32`

Suppose we use the following objects (like in [embedded-graphics](https://docs.rs/embedded-graphics/latest/embedded_graphics/geometry/index.html)):

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

The visual length of a line between two `Point` types can be described by `Size` : both the height and the width of the line are between `0` and `u32::MAX = i32::MAX - i32::MIN`. To simplify the situation, let's imagine two `i32` variables `a` and `b` on one axis :

| `a: i32` | `b: i32`  | visual length of the line |
| --- | --- | --- |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `i32::MIN` | `i32::MAX` | `u32::MAX` |
| `i32::MAX` | `i32::MIN` | `u32::MAX` |

## Naive Approach

At first I thought computing this length would be very simple, so I tried using the 3 following simple formulas :

| `a: i32` | `b: i32`  |`(b - a).abs()` (*) | `(b - a) as u32` (*) | `b as u32 - a as u32` (*) |
| --- | --- | --- | --- | --- |
| `0` | `1` | `1` | `1` | `1` |
| `1` | `0` | `1` | <span style="color:red">`u32::MAX`</span> | <span style="color:red">`u32::MAX`</span> |
| `i32::MIN` | `i32::MAX` | <span style="color:red">`1`</span> | <span style="color:red">`u32::MAX`</span> | <span style="color:red">`u32::MAX`</span> |
| `i32::MAX` | `i32::MIN` | <span style="color:red">`1`</span> | <span style="color:red">`1`</span> | <span style="color:red">`1`</span> |

(*) wrapping arithmetic is used here

The classic formula (below) to compute the length of a line infinite integers doesn't work for 32-bit integers.

![equation]({{ "/static/2025-06-07-unsigned-distance-for-i32/equation.svg" | relative_url }})

32-bit math can be counter-intuitive... Luckily Rust provides a lot of arithmetic functions, and using checked arithmetic does shed some light on the problem.

## Using Checked Arithmetic

| `a: i32` | `b: i32`  |`(b - a).abs()` | `(b - a) as u32` |
| --- | --- | --- | --- |
| `0` | `1` | `1` | `1` |
| `1` | `0` | `1` | <span style="color:red">`u32::MAX`</span> |
| `i32::MIN` | `i32::MAX` | <span style="color:red">`overflow`</span> | <span style="color:red">`overflow`</span> |
| `i32::MAX` | `i32::MIN` | <span style="color:red">`overflow`</span> | <span style="color:red">`overflow`</span> |

It turns out that the classic formula `(b - a).abs()` is still correct, but it only works when there's no overflow.

Therefore a correct code has to make the distinction between the overflowing case and the other one :

```rust
match b.overflowing_sub(a) {
	diff, true => diff.unsigned_abs().wrapping_neg(), // needs proof !
	diff, false => diff.unsigned_abs(), // trivial case
}
```

Another option is to notice that the formula `(b - a) as u32` is also correct half of the time, more precisely when `b >= a`.

```rust
if b >= a  {
	(b - a) as u32
} else {
	(a - b) as u32
}
```