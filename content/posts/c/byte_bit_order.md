---
title: "Byte/Bit Order"
date: 2023-11-25T16:21:27+08:00
tags: ["C"]
categories: ["C"]
---


# 字节序与比特序
字节序又称大小段，网络中传输的是大端，在CPU上处理的一般是小端。


# 字节序与比特序转换

## 字节序转换

## 比特序转换
两种方法，一种直接法，**另外有一种优化的技巧**。

（1）
```c
// Bit reverse
unsigned char
bit_reverse(unsigned char x)
{
  unsigned char newx = 0;
  for (int i = 0; i < 8; i++) {
    newx |= (((x >> i) & 1) << (7-i));
  }
  return newx;
}
```
(2) https://mp.weixin.qq.com/s/KNUH_RmIhUHhuSZLSmN4LQ

```c
// Bit reverse(faster)
// 碟式交换法
unsigned char
bit_reverse_faster(unsigned char x)
{
  x = (x<<4) | (x>>4); // [ 5678 1234 ]
  x = ((x<<2)&0xcc) | ((x>>2)&0x33); // [ 78 56 34 12 ]
  x = ((x<<1)&0xaa) | ((x>>1)&0x55); // [ 8 7 6 5 4 3 2 1 ]
  return x;
}
```
测试工具函数:
```c
int
to_binary_str(unsigned long val, char *bin_str, int *len)
{
  char *p, *q;
  int cnt = 0;

  while (val) {
    char tmp = val & 1;
    bin_str[cnt++] = tmp+'0';
    val = val >> 1;
  }

  p = bin_str, q = bin_str+cnt-1;
  while (p < q) {
    char tmp = *p;
    *p = *q;
    *q = tmp;
    p++, q--;
  }

  bin_str[cnt] = 0;
  *len = cnt;
  return 0;
}

int main(void)
{
  char bin[65];
  int len;
  unsigned char x = 0b11110000;
  unsigned char y = 0b10101010;
  to_binary_str(bit_reverse(x), bin, &len);
  printf("bin: %s\n", bin);
  to_binary_str(bit_reverse(y), bin, &len);
  printf("bin: %s\n", bin);

  assert(bit_reverse(x) == bit_reverse_faster(x));
  assert(bit_reverse(y) == bit_reverse_faster(y));

  return 0;
}  
```

