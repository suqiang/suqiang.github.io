---
title: iOS异或(XOR)加密
comments: true
categories: iOS 加密
tags: iOS 加密 XOR
date: 2016-11-02 14:19:50
updated: 2016-11-02 14:19:50
permalink:
---


源码见文末
## 异或简介
* `异或`，二元运算;
* 符号`^`;
* 口诀：相同为0,不相同为1;  
* 举例
  * 10011101^10001010=00010111

## 应用场景
* 密码加密(此场景在日常开发中使用较多,本文将着重介绍此应用场景)
* 数据暂存
* 嵌入式

## `异或加密`原理
* 假设明文A, 密钥B, 密文C,
* 通过`异或加密`过程为A ^ B = C, 得到密文C
* 如果此时知道密文A和密钥B,那么通过`异或解密`过程为C ^ B = A, 重新得到明文A
* ![异或加密](http://o7obltx2h.bkt.clouddn.com/imagexor.png)

## `异或加密`应用
* 企业由于数据具有保密性,需要对敏感数据进行加密。 数据存在的形式可能是客户端的缓存、内置资源文件、网络传输等，这些数据都有可能被用户，竞争对手，黑客截获的潜在风险；通过加密可以进行规避，异或加密是其中较为简单的一种实现方案。
* 例如某公司App要内置一些核心资源配置文件，为防止用户窥探，采用了加密处理，先将方案分享如下：
  * 后端生成配置文件时，用私钥对数据进行与运算处理，生成密文；客户端拿到密文，通过自动或手动打包的app中，app在运行时，采用相同的密钥、按照相同的算法进行异或运算，得到明文，进而使用。
  * 异或算法
    * 假设 私钥`abcdefg`7位，将数据转换为`data`类型，获取所有字节；将字节每七位分别于私钥七位字符异或运算，一直循环到最后得到密文
    * 异或解密算法和加密算法一致，即通过这个算法运行一次是加密，再通过这个算法运行一次就是解密

## iOS代码实现

```objc
// 客户端内置私钥
static NSString const *privateKey = @"ef37c9111210854f5986fc9ebb5548b2ae";

@implementation NSData(XOREncrypt)
- (NSData *)xor_decrypt
{
    return [self xor_encrypt];
}

// 加密
- (NSData *)xor_encrypt
{
    // 获取key的长度
    NSInteger length = privateKey.length;

    // 将OC字符串转换为C字符串
    const char *keys = [privateKey cStringUsingEncoding:NSASCIIStringEncoding];

    unsigned char cKey[length];

    memcpy(cKey, keys, length);

    // 数据初始化，空间未分配 配合使用 appendBytes
    NSMutableData *encryptData = [[NSMutableData alloc] initWithCapacity:length];

    // 获取字节指针
    const Byte *point = self.bytes;

    for (int i = 0; i < self.length; i++) {
        int l = i % length;                     // 算出当前位置字节，要和密钥的异或运算的密钥字节
        char c = cKey[l];
        Byte b = (Byte) ((point[i]) ^ c);       // 异或运算
        [encryptData appendBytes:&b length:1];  // 追加字节
    }
    return encryptData.copy;
}
@end
```

---

## 源码

[XORDemo源码](https://github.com/suqiang/iOSDemo/blob/master/iOSDemo/Classes/Category/Encrypt/XOR/SQHXOREncryptUtil.m)
