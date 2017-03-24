### 目标

源代码中包含明文字符串，很容易通过静态反编译工具分析出来，strings工具也可以将其dump出来，非常不利于敏感文本保护。

用加密文本代替明文，能有效阻止静态反编译工具分析，增加破解难度。

### 加密流程

加密方法选用简单快速的[TEA](https://en.wikipedia.org/wiki/XTEA)，密钥默认{0x0, 0x0, 0x0, 0x0}。

举例来说，字符串“hello”加密后的结果为
|  h   |  e   |  l   |  l   |  o   |  \0  |      |      |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0xca | 0x9d | 0xc3 | 0x69 | 0x36 | 0xed | 0xff | 0xac |

加密后的密文不再以\0结尾，在密文前加上当前密文长度

|  \8  |  0   |  0   |  0   |  h   |  e   |  l   |  l   |  o   |  \0  |      |      |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0x08 | 0x0  | 0x0  | 0x0  | 0xca | 0x9d | 0xc3 | 0x69 | 0x36 | 0xed | 0xff | 0xac |

通常密文长度比较小，很容易被看出来。这里简单对密文长度、密文前4字节、TEA的DELTA进行异或操作，增加破解难度

|  \8  |  0   |  0   |  0   |  h   |  e   |  l   |  l   |  o   |  \0  |      |      |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0x7b | 0xe4 | 0xf4 | 0xf7 | 0xca | 0x9d | 0xc3 | 0x69 | 0x36 | 0xed | 0xff | 0xac |

### 解密流程

解密流程相反，先通过异或解出密文长度，再通过TEA解密字符串。

解密后，将前4字节置为0，表示已解密，下次使用时无需再重复解密。