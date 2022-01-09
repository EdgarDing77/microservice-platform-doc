# Bcrypt加密

## Introduction

Bcrypt是一种跨平台的文件加密工具。Bcrypt 使用的是布鲁斯·施内尔在1993年发布的 Blowfish 加密算法。由它加密的文件可在所有支持的操作系统和处理器上进行转移。它的口令必须是8至56个字符，并将在内部被转化为448位的密钥。

Bcrypt就是一款加密工具，可以比较方便地实现数据的加密工作。你也可以简单理解为它内部自己实现了随机加盐处理。使用Bcrypt，每次加密后的密文是不一样的。

**对一个密码，Bcrypt每次生成的hash都不一样，那么它是如何进行校验的？**

虽然对同一个密码，每次生成的hash不一样，但是hash中包含了salt（hash产生过程：先随机生成salt，salt跟password进行hash）；
在下次校验时，从hash中取出salt，salt跟password进行hash；得到的结果跟保存在DB中的hash进行比对。
BCrypt算法将salt随机并混入最终加密后的密码，验证时也无需单独提供之前的salt，从而无需单独处理salt问题。





## Reference

- https://blog.csdn.net/m0_37609579/article/details/100785947