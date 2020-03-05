---
layout: post
published: true
title: One-liner for splitting fasta files into groups of N
subtitle: oneliner
date: '2020-03-05'
---
## One-liner for splitting fasta files into groups of N

### e.g., for n=100:

```bash
time perl -ne BEGIN{$a=0;open OUT,">delme.txt";}if($a%100==0){close OUT;open OUT,">set_$a.fa";}print OUT $_;if(/>/){$a++}; c_elegans.PRJNA13758.WS274.protein.fa
```