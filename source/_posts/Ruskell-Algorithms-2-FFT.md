---
title: Ruskell & Algorithms (2) -- FFT
date: 2017-12-09 23:27:43
tags: 
- Rust
- Algorithms
---

这是一个利用Rust&Haskell实现算法导论上的经典算法或其他有趣算法的系列，代码可以点击此[在线运行](https://play.rust-lang.org/)得到结果，你可能需要补充合理的输入输出与main函数。
第二篇是实现FFT与普通多项式乘法，下面是相应的Rust代码：

首先是FFT算法
```rust
fn dft(src: &mut Vec<Complex<f64>>, flag: bool) {
	// src 的长度为 2 的幂； flag 为 false 时计算 DFT， 为 true 时计算逆 DFT
	let len = src.len();
	let s = len.trailing_zeros();
	for i in 0..len {
		let mut k = 0;
		for j in 0..s {
			k |= ((i >> j) & 1) << (s - 1 - j);
		}
		if i < k {
			src.swap(i, k);
		}
	}
	for i in 0..s {
		let base = (2 << i) as usize;
		let mut j = 0;
		while j < len {
			for k in 0..(base / 2) {
				let w = if flag {
					Complex::from_polar(&1.0, &(-2.0 * f64::consts::PI * k as f64 / base as f64))
				} else {
					Complex::from_polar(&1.0, &(2.0 * f64::consts::PI * k as f64 / base as f64))
				};
				let t = w * src[base / 2 + j + k];
				let u = src[j + k];
				src[j + k] = u + t;
				src[base / 2 + j + k] = u - t;
			}
			j += base;
		}
	}
	if flag {
		for i in 0..len {
			src[i] = src[i] / len as f64;
		}
	}
}
fn poly_mult_fft(a: &[f64], b: &[f64]) -> Vec<f64> {
	let len = 1 << (32 - ((a.len() + b.len()) as i32).leading_zeros());
	let mut aa: Vec<Complex<f64>> = vec![Complex {re: 0.0, im: 0.0}; len];
	for i in 0..a.len() {
		aa[i].re = a[i];
	}
	dft(&mut aa, false);
	let mut bb: Vec<Complex<f64>> = vec![Complex {re: 0.0, im: 0.0}; len];
	for i in 0..b.len() {
		bb[i].re = b[i];
	}
	dft(&mut bb, false);
	for i in 0..len {
		aa[i] *= bb[i];
	}
	dft(&mut aa, true);
	let mut c = Vec::new();
	for i in 0..(a.len() + b.len() - 1) {
		c.push(aa[i].re);
	}
	c
}
```

然后是普通乘法
```rust
fn poly_mult_norm(a: &[f64], b: &[f64]) -> Vec<f64> {
	let al = a.len();
	let bl = b.len();
	let cl = al + bl - 1;
	let mut c = vec![0.0; cl];
	for i in 0..cl {
		let k = if i + 1 < bl { 0 } else { i + 1 - bl };
		let m = if i + 1 > al { al } else { i + 1 };
		for j in k..m {
			c[i] += a[j] * b[i - j];
		}
	}
	c
}
```

Note:

1. 需要`extern crate num`使用复数
2. FFT需要大约2000+的输入量才能发挥其数量级优势，否则被常数拖死，还没有普通乘法快