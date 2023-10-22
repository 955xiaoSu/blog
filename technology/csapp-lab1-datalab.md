# CSAPP —— lab1 datalab

### 准备工作

1. 阅读 README && bits.c 中的要求；
2. &#x20;`./dlc -e bits.c` 用来检测代码是否合规；&#x20;
3. `make btest` 用来编译，而后键入 `./btest` 测试所写代码的正确性；
4. &#x20;`./ishow || ./fshow` 可查看对应数字的十六进制表示；&#x20;
5. 对 op 的作用理解：
   * !：逻辑判断，是否为 0&#x20;
   * \~：二进制补码转换
   * &#x20;&、|：构造特定位上的 0 / 1&#x20;
   * <<、>>：二进制乘除

### WriteUp

1. bitAnd 根据离散数学的知识，对 x & y 取反再取反，得到 \~(\~x | \~y)。
2. getByte 要得到什么？某一字节的数据。那么如何处理其它字节的数据 —— &0 / ^self，方便起见这里采用 &0 的方式。因为所有的数据在底层都是一串 01，所以要想进行乘除，可以采用 shift。做法：将特定字节右移到最低位字节，再 &0xff。
3. logicalShift 正数直接右移没问题，主要处理一下负数右移产生的多余的 1（算术右移，空位补符号位导致），采用 & 的方式将其转变成 0。
4. bitCount 在不允许使用 looping && control statement 的情况下，完全没思路。参考别人的想法——二分法，分组判断有几个 1，最后累加求解。
5. bang 这题比较 tricky，用到的性质 self | self's complement = full 1（即 -1），所以除了 0 以外（0 | 0 = 0），其它所有的值最终都转变为 -1，求解结果再 +1，实现非 0 值 → 0、0 → 1，bang！
6. tmin 根据补码的二进制计算公式，1 << 31 顺理成章。
7. fitsBits 两个点。第一点，如何在 Legal ops: ! \~ & ^ | + << >> 的情况下实现减去某个数（假设为 n），答案是：\~n + 1；第二点：怎么证明用 n bits 可以表示某个数？!(x ^ ((x << shift) >> shift))。能不能先右移再左移？肯定是不行的，因为右移数据会失真，原位上如果有 1，那么通过左移的方式就会变成 0，导致无法匹配。算法错误 → 结果错误。
8. divpwr2 对于正数，向下取整 no problem。对于负数，为了向 0 看齐，得加上一个偏移量，使得自身低 n 位为偶数。奇妙之处就在于，自己 + 自己 = 偶数（对低 n 位而言），因为奇 + 奇 = 偶，偶 + 偶 = 偶。
9. negate 根据反码的计算公式，\~x + 1 水到渠成。
10. isPositive 首先得非负，所以判断其最高位。其次不为 0，逻辑判断 !! 就派上用场了。
11. isLessOrEqual 考虑到 overflow 的情况，因此分情况讨论。当 x 与 y 同号，判断二者差的符号位；当 x 与 y 异号，判断 x 的符号位。表达式为：((!same) & x\_sign) | (same & !(diff\_sign)) ，对于结果为 0 / 1 的情况，可以参考此种手法，实现选择性返回。
12. ilog2 目标：返回最高位 1 所在位数。借鉴 bitCount 思路，采用二分法，分组查询累加。
13. float\_neg tip：从此题开始，可以用各式各样的 op && condition control && looping。若 NaN 返回 argument，否则返回 -argument。NaN 的判断依据是 exp == full 1 && frac ！= 0。可以通过先将 argument 左移一位，去除符号位的影响，再通过 if 判断。
14. float\_i2f 将 int → float，逐步求 sign、exp、frac 即可。注意对有进位的情况进行修正。
15. float\_twice 基本思路同 float\_i2f。注意讨论 exp 全0、全1、非全0全1，三种情况下，应该对 frac / exp 做怎样的处理。

### Code

```cpp
/* 
 * bitAnd - x&y using only ~ and | 
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 8
 *   Rating: 1
 */
int bitAnd(int x, int y) {
    /* To achieve "x & y" expression */
    
    return ~(~x | ~y);
}
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
int getByte(int x, int n) {
   /*
    *  All number is 0 | 1 in nature 
    *  so if you wanna mutiplie, consider shift
    *  First, deal parameter 'n', make it * 8
    *  then shift 'x', last clear
    */

    return (x >> (n << 3)) & 0xFF;
}
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */
int logicalShift(int x, int n) {
    /* Deal negative number high bit 1 */
    
    int recover = (1 << 31) >> n << 1;
    return ((x >> n) & ~recover);
}
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) {
    int count;
    int tpMask1 = 0x55 | (0x55 << 8);
    int mask1 = tpMask1 | (tpMask1 << 16);  //01010101
    
    int tpMask2 = 0x33 | (0x33 << 8);
    int mask2 = tpMask2 | (tpMask2 << 16);  //00110011 

    int tpMask3 = 0x0f | (0x0f << 8);
    int mask3 = tpMask3 | (tpMask3 << 16);  //00001111 

    int mask4 = 0xff | (0xff << 16);        //00000000 11111111 
    int mask5 = 0xff | (0xff << 8);         //00000000 00000000 11111111 11111111
    count = (x & mask1) + ((x>>1) & mask1); 
    count = (count & mask2) + ((count >> 2) & mask2);  //每4位一组，先计算低位的两位，右移2位后再计算高位两位
    count = (count + (count >> 4)) & mask3;  //每8位一组
    count = (count + (count >> 8)) & mask4;  //每16位一组
    count = (count + (count >> 16)) & mask5;  //总共32位为一组，将前16位的结果与后16位的结果相加即得最终答案
    return count;
}
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x) {
    /* Self | self own complement get full 1 and then shift sign bit */
    
    int complement = ~x + 1;
    return ((x | complement) >> 31) + 1; 
}
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
    /* According to the two's complement computation rule */

    return (1 << 31);
}
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n) {
    /* 
     * Judge how many bits we need to represent this num 
     * Which means using shift operation doesn't change value
     * First expand to higher bit and move back
     * Meanwhile, ~n + 1 == -n (two's complement)
     */

     int shift = 32 + (~n + 1);
     return !(x ^ ((x << shift) >> shift)); 
}
/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int divpwr2(int x, int n) {
    /* Check and then shift, negative low n bit add themself */
    
    int mask = ~((1 << 31) >> 31 << n); /* 00001111 */
    int b = (x >> 31) & mask; /* self += self must be even  */ 
    return (x + b) >> n;
}

/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
    return ~x + 1;
}
/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) {
    /* Judge from highest bit & 0? */
    
    int sign = x >> 31;
    int is_not_zero = !!x;
    return (is_not_zero & !sign);
}
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
    /* Use substraction, and then observe signed bit */
    
    int difference = y + (~x) + 1; 
    int x_sign = x >> 31; 
    int y_sign = y >> 31;  
    int same = !(x_sign ^ y_sign); 
    int diff_sign = difference >> 31; 
    return ((!same) & x_sign) | (same & !(diff_sign));  
}

/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
int ilog2(int x) {
    /* Dichotomy to find the highest one */
 
    int count = 0;
    count = !!(x >> 16) << 4;
    count += !!(x >> (count + 8)) << 3;
    count += !!(x >> (count + 4)) << 2;
    count += !!(x >> (count + 2)) << 1;
    count += !!(x >> (count + 1));
    
    return count;  
}
/* 
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) {
    /* Whether a NaN */
  
    if (((0xFF000000 & (uf << 1)) == 0xFF000000) /* exp == 11111111 */
          && (uf << 1) != 0xFF000000)            /* frac != 0 */
       return uf;
    else return uf ^ 0x80000000;
}

/* 
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_i2f(int x) {
    /* int -> float, sign + exp + frac */

    /* special cases */
    if (x == 0x80000000) return 0xcf000000;
    if (x == 0) return 0;
    
    /* sign */
    unsigned sign = (x >> 31) << 31;
    
    /* exp */
    int abs_x = x; 
    /* get abs x */
    if (abs_x < 0) abs_x = -x; 

    int tmp = abs_x;
    int highest_bit = 0;
    while (tmp) {
        highest_bit++;
        tmp >>= 1;
    }
    int weight = highest_bit - 1;

    unsigned exp = (127 + weight) << 23;

    /* frac */
    unsigned frac, frac_cut;
    if (weight > 23) {
        frac = (abs_x >> (weight - 23)) & 0x7fffff;
        frac_cut = (abs_x << (31 - weight)) & 0xff;
        /* carry */
        if ((frac_cut > 0x80) || (frac_cut == 0x80 && (frac & 1))) {
            frac++;
        }
    } else frac = (abs_x << (23 - weight)) & 0x7fffff;

    return sign + exp + frac;
}
/* 
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_twice(unsigned uf) {
    /* sign + exp + frac */
    /* sign */
    unsigned sign = (uf >> 31) << 31;

    /* exp & frac */
    unsigned exp = uf & 0x7f800000;
    unsigned frac = uf & 0x7fffff;

    /* Deal with different situations */
    if (exp == 0x7f800000) return uf;
    else {
        if (exp == 0) frac <<= 1;
        else exp += 0x800000;
    }

    return sign + exp + frac;
}
```
