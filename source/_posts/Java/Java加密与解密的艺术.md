---
title: Java加密与解密的艺术
date: 2019-04-18
categories: Java
---

Url Base64算法主要是替换了Base64字符映射表中的第62和63个字符，也就是将“+”和“/”符号替换成了“-”和“_”符号。但对于补位符“=”，一种建议是使用“~”符号，另一种建议是使用“.”符号。其中，由于“~”符号与文件系统冲突，不建议使用；而对于“.”符号，如果出现连续两次，则认为是错误。对于补位符的问题，Bouncy Castle和Commons Codec有差别：Bouncy Castle使用“.”作为补位符，而Commons Codec则完全杜绝使用补位符。

Bouncy Castle和Commons Codec都提供了Base64算法实现，两组织在算法实现上遵循了不同的标准。Bouncy Castle实现了一般Base64编码，而Commons Codec遵循RFC 2045的相关定义做了算法实现。简而言之，RFC 2045要求Base64编码后的字符串每行76个字符，不论每行是否够76个字符，都要在行末添加一个回车换行符（“\r\n”）。Bouncy Castle并没有对于Base64编码字符串的行要求，而Commons Codec遵循了这些要求并提供了Base64算法实现的方法，而且在方法易用性上提供了更广泛的API，方便使用。因此，对于Base64算法的实现，Commons Codec更胜一筹。

**如果需要在76个字符后换行，使用如下方法：**
`Base64.encodeBase64(data.getBytes(ENCODING), true);`
