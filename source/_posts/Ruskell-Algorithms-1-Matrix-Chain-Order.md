---
title: Ruskell & Algorithms (1) -- Matrix Chain Order
date: 2017-11-29 10:10:12
tags: 
- Rust
- Algorithms
---

这是一个利用Rust&Haskell实现算法导论上的经典算法或其他有趣算法的系列（大概不会坑吧），所有代码可以点击此[在线运行](https://play.rust-lang.org/)得到结果。
第一篇是实现的利用动态规划解决矩阵链乘法问题的问题，下面是相应的Rust代码：

```rust
extern crate time;
extern crate rand;

use time::*;
use std::vec;

fn print_optimal_parens(/*file:&mut File, */s:&Vec<Vec<u32>>, i:u32, j:u32 ) 
{
    if i == j
    {
        print!("A{} ", i);
    }
    else
    {
        print!("( ");
        print_optimal_parens(s, i, s[i as usize][j as usize]);
        print_optimal_parens(s, s[i as usize][j as usize]+1, j);
        print!(") ");
    }
}

fn matrix_chain_order(p:&Vec<u32>) -> (Vec<Vec<u32>>, Vec<Vec<u32>>)
{
    let n = p.len() - 1;
    let mut m:Vec<Vec<u32>> = Vec::with_capacity(n+1);
    let mut s:Vec<Vec<u32>> = Vec::with_capacity(n+1);
    for i in 0..n+1
    {
        let mut temp:Vec<u32> = Vec::with_capacity(n+1);
        for j in 0..n+1
        {
            temp.push(0);
        }
        m.push(temp);
        let mut temp:Vec<u32> = Vec::with_capacity(n+1);
        for j in 0..n+1
        {
            temp.push(0);
        }
        s.push(temp);
    }
    
    for l in 2..n+1
    {
        for i in 1..n-l+2
        {
            let j:usize = i+l-1;
            m[i][j] = std::u32::MAX;
            for k in i..j
            {
                let temp:u32 = p[i-1]* p[k] * p[j];
                let q:u32 = m[i][k] + m[k+1][j] + temp;
                if q < m[i][j]
                {
                    m[i][j] = q;
                    s[i][j] = k as u32;
                }
            }
        }
    }

    (s, m)
}

fn main()
{
    let mut vec = Vec::new();
    for i in 0..31 //generate n+1 (n=30) numbers
    {
        vec.push(rand::random::<u32>() % 100 + 1);
    }
    
    let start = time::now();	
    let (s, m) = matrix_chain_order(&vec);
    
    println!("s:");
    for i in 0..s.len()
    {
        for j in 0..s[i].len()
        {
            print!("{}  ", s[i][j]);
        }
        println!("  new line ");
    }
    println!("m:");
    for i in 0..m.len()
    {
        for j in 0..m[i].len()
        {
            print!("{}  ", m[i][j]);
        }
        println!("  new line  ");
    }
    
	let end = time::now();
	let dura = end - start; 
	println!("{:?}", dura);  
    print_optimal_parens(&s, 1, s.len() as u32 - 1);

}
```

Note:

1. 有一个小坑是关于time的，似乎是由于实现原因，不能用占位符{}打印time，需要用debug下的{:?}来打印时间。
2. Rust的变量生存周期跟c系列还是挺不一样的，写的时候略微有些不习惯