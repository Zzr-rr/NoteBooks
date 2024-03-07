## Buffer lcm

> 引入GPT对该term的解释："Buffer LCM" 指的是在软件开发中的核心工具库中使用的一个计算函数或概念，特别是与文件I/O操作相关的场景中。这个术语涉及到为处理两个文件的I/O操作计算合适的缓冲区大小。该函数会计算两个文件块大小的最小公倍数（LCM）来确定操作的高效缓冲区大小。然而，它还包括了一个最大限制（LCM_MAX），以确保缓冲区大小不会超过一个合理的值。这个概念在优化文件操作中非常有用，确保操作高效执行，不浪费资源。

### 源码

```c
#include <config.h>
#include "buffer-lcm.h"

/* Return a buffer size suitable for doing I/O with files whose block
   sizes are A and B.  However, never return a value greater than
   LCM_MAX.  */

size_t
buffer_lcm (size_t a, size_t b, size_t lcm_max)
{
  size_t size;

  /* Use reasonable values if buffer sizes are zero.  */
  if (!a)
    size = b ? b : 8 * 1024;
  else
    {
      if (b)
        {
          /* Return lcm (A, B) if it is in range; otherwise, fall back
             on A.  */

          size_t lcm, m, n, q, r;

          /* N = gcd (A, B).  */
          for (m = a, n = b;  (r = m % n) != 0;  m = n, n = r)
            continue;

          /* LCM = lcm (A, B), if in range.  */
          q = a / n;
          lcm = q * b;
          if (lcm <= lcm_max && lcm / b == q)
            return lcm;
        }

      size = a;
    }

  return size <= lcm_max ? size : lcm_max;
}
```

一般来说，这样的lib都是一些`.c`的文件。该c语言实现了一个基本的buffer lcm，目的就是为了I/O操作设计适合的缓冲区大小。学习一下其设计思想

> Return a buffer size suitable for doing I/O with files whose block
>    sizes are A and B.  However, never return a value greater than
>    LCM_MAX. 

1. 当缓冲块a大小为0的时候，就看块b是否也为0，如果也为0就返回8 * 1024，不然返回b的大小。

2. 不然就返回a和b的最小公倍数以及`LCM_MAX`两者中的较小值。

3. 求解最小公倍数的过程就是先通过欧几里得算法(Euclidean Algorithm)来求出最大公因数(GCD)，再通过GCD求出LCM。

   