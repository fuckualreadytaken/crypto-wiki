## RSA密码算法

RSA密码算法是最经典的非对称密码算法，这种密码算法是基于大质数分解难题而设计的。

**值得注意的是，任何一种非对称密码算法都是基于这样一种数学难题——正向很容易求解，逆向特别难求解。比如RSA就是由两个大素数求其积比较容易，而已知两个大素数的积来分解两个大素数比较困难。**

## RSA密码算法流程

这里采用小的素数来进行计算并说明RSA密码算法的流程。

1. 选取两个素数，并计算它们的积；  

- 设两个素数分别为p=41、q=43
- 它们的积为n=p*q=1763

2. 设置公钥参数e并计算d;

- 令φ(n)=(q-1)(p-1)=1680  
- 选择公钥参数e=17，且gcd(e,φ(n))=1，其中gcd是求最大公约数的函数
- 求得d，使得d*e = 1 mod n，即d与e的乘积除以n的余数为1，一般来说，求d可以用扩展的欧几里得算法求解，见附录。这里求出d=593  

3. 加解密过程；
在步骤二中求得d之后，RSA的基本元素就凑齐了，第1步中求得n，与第2步中求得的de。其中(n,e)为公钥、(n,d)为私钥。  
首先来看加密过程，设加密的消息m=1207，一般来说，密文的长度不能超过n；  
加密：  
c = m^e mod n = 1207^17 mod 1763 = 1144  
解密：  
m = c^d mod n = 1144^593 mod 1763 = 1207  

## 附录

### 扩展的欧几里得算法

欧几里得算法（欧几里得算法又称辗转相除法）求公约数，这里以56与356的公约数：  
356 ÷ 56 = 6&emsp;&emsp;&emsp;365 % 56 = 20  
56 ÷ 20 = 2&emsp;&emsp;&emsp;56 % 20 = 16  
20 ÷ 16 = 1&emsp;&emsp;&emsp;20 % 16 = 4  
16 ÷ 4 = 4&emsp;&emsp;&emsp;16 % 4 = 0  
最大公约数为4。  
上述算法就为欧几里得算法，扩展的欧几里得算法是基于欧几里得算法的。以上面数字为例，将每一项逆推回去得到：  
4 = 20 - 16x1  
&emsp;= 20 - (56 - 20x2)x1  
&emsp;= 20x3 - 56  
&emsp;= (356 - 56x6)x3 - 56  
&emsp;= 356x3 - 56x19  
整理一下可以表示为：56x19 =-4 + 356x3，56x19 =-4 mod 356  
关于通过公钥参数求d也可以用类似的方法求解：  
ed = 1 mod φ(n)  
17d = 1 mod 1680

先用欧几里得算法求最大公约数：  
1680 ÷ 17 = 98&emsp;&emsp;&emsp;1680 % 17 = 14  
17 ÷ 14 = 1&emsp;&emsp;&emsp;17 % 14 = 3  
14 ÷ 3 = 4&emsp;&emsp;&emsp;14 % 3 = 2  
3 ÷ 2 = 1&emsp;&emsp;&emsp;3 % 2 = 1  
2 ÷ 1 = 2&emsp;&emsp;&emsp;2 % 1 = 0  
扩展的欧几里得算法逆推如下：  
1 = 3 - 2 x 1  
&emsp;=3 x 5 - 14  
&emsp;=17 x 5 - 14 x 6  
&emsp;=17 x 5 - (1680 - 17 x 98) x 6  
&emsp;=17 x 593 - 1680 x 6  
两边同时除以1680得：17 x 593 = 1 mod 1680  
因此d = 593  
## 模幂运算
肯定有很多小伙伴疑惑这个1144^593 mod 1763具体是怎么计算的，因为直接拿代码先计算1144^593可能会报错（之前用Python 3.5会报数值太大的错，现在用3.7好像能计算出来，但是真正用到生活中的比较大的数，用代码直接计算幂肯定不行），不过这后面有个求模运算，因此可以采用求模运算的性质。  
&emsp;&emsp;性质1：(a x b) mod n = ((a mod n) x (b mod n)) mod n
&emsp;&emsp;性质2：(a + b) mod n = ((a mod n) + (b mod n)) mod n
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;(a - b) mod n = ((a mod n) - (b mod n)) mod n
根据上述性质，可以很容易计算1144^593 mod 1763  

1. 首先将593表示为2的幂；
   593 = 2^9(512) + 2^6(64) + 2^4(16) + 2^0(1)
   化为二进制为：1001010001
2. 计算；
   原式可以表示为： 1144^ (2^9 + 2^6 + 2^4 + 2^0) mod 1763  
   这样的话，就可以先算a1 = 1144^2 mod 1763
   那么，a2 = a1^2 mod 1763 = 1144^ (2 ^ 2 ) mod 1763  
        a3 = a2^2 mod 1763 = 1144^ (2 ^ 3) mod 1763
    按照这样算下去，就很容易算出1144^ (2^9) mod 1763,1144^ (2^6) mod 1763,1144^ (2^4) mod 1763，然后根据**性质1**计算出整个式子的值

这个算法实现起来也比较容易：

参考代码：  

```python
def ModExp(n, k, m):
        """This method is use to calculate the big Modular Exponentiation
        :param n: Base
        :param k: Component
        :param m: Modulus
        """
        a = list(bin(k))[2:]
        a.reverse()
        s = 1
        for i in a:
            if i == '1':
                s = (s * n) % m
            n = (n * n) % m
        return s
```