---
layout:     post
title:      Swift的HMAC和SHA1加密
subtitle:   swift中利用HMAC的SHA1对文本进行加�?
date:       2017-07-19
author:     BY
header-img: img/init/post-bg-hacker.jpg
catalog: true
tags:
    - Swift
---

>HMAC是密钥相关的哈希运算消息认证码（Hash-based Message Authentication Code）�?HMAC运算利用哈希算法，以一个密钥和一个消息为输入，生成一个消息摘要作为输出。也就是说HMAC通过将哈希算�?SHA1, MD5)与密钥进行计算生成摘要�?

## Objectice-C

在上�?Objectice-C 项目中，使用�?HMAC �?SHA1 进行加密。如下代码：

```objc
+ (NSString *)hmacsha1:(NSString *)text key:(NSString *)secret {
    
    NSData *secretData = [secret dataUsingEncoding:NSUTF8StringEncoding];
    NSData *clearTextData = [text dataUsingEncoding:NSUTF8StringEncoding];
    unsigned char result[20];
    // SHA1加密
    CCHmac(kCCHmacAlgSHA1, [secretData bytes], [secretData length], [clearTextData bytes], [clearTextData length], result);
    char base64Result[32];
    size_t theResultLength = 32;
    // 转为Base64
    Base64EncodeData(result, 20, base64Result, &theResultLength,YES);
    NSData *theData = [NSData dataWithBytes:base64Result length:theResultLength];
    NSString *base64EncodedResult = [[NSString alloc] initWithData:theData encoding:NSUTF8StringEncoding];
    return base64EncodedResult;
}
```



## swift

最近用 swift 重构项目,�?Swift [重写了](https://stackoverflow.com/questions/26970807/implementing-hmac-and-sha1-encryption-in-swift?rq=1) HMAC �?SHA1 加密方式�?

### 使用

```swift
// 使用HMAC和SHA加密
let hmacResult:String = "myStringToHMAC".hmac(HMACAlgorithm.SHA1, key: "myKey")
```

### 代码

使用下面代码时，需要在 OC 桥接文件`xxx-Bridging-Header`�?`#import <CommonCrypto/CommonHMAC.h>`

```swift
extension String {
    func hmac(algorithm: HMACAlgorithm, key: String) -> String {
        let cKey = key.cStringUsingEncoding(NSUTF8StringEncoding)
        let cData = self.cStringUsingEncoding(NSUTF8StringEncoding)
        var result = [CUnsignedChar](count: Int(algorithm.digestLength()), repeatedValue: 0)
        CCHmac(algorithm.toCCHmacAlgorithm(), cKey!, strlen(cKey!), cData!, strlen(cData!), &result)
        var hmacData:NSData = NSData(bytes: result, length: (Int(algorithm.digestLength())))
        var hmacBase64 = hmacData.base64EncodedStringWithOptions(NSDataBase64EncodingOptions.Encoding76CharacterLineLength)
        return String(hmacBase64)
    }
}

enum HMACAlgorithm {
    case MD5, SHA1, SHA224, SHA256, SHA384, SHA512

    func toCCHmacAlgorithm() -> CCHmacAlgorithm {
        var result: Int = 0
        switch self {
        case .MD5:
            result = kCCHmacAlgMD5
        case .SHA1:
            result = kCCHmacAlgSHA1
        case .SHA224:
            result = kCCHmacAlgSHA224
        case .SHA256:
            result = kCCHmacAlgSHA256
        case .SHA384:
            result = kCCHmacAlgSHA384
        case .SHA512:
            result = kCCHmacAlgSHA512
        }
        return CCHmacAlgorithm(result)
    }

    func digestLength() -> Int {
        var result: CInt = 0
        switch self {
        case .MD5:
            result = CC_MD5_DIGEST_LENGTH
        case .SHA1:
            result = CC_SHA1_DIGEST_LENGTH
        case .SHA224:
            result = CC_SHA224_DIGEST_LENGTH
        case .SHA256:
            result = CC_SHA256_DIGEST_LENGTH
        case .SHA384:
            result = CC_SHA384_DIGEST_LENGTH
        case .SHA512:
            result = CC_SHA512_DIGEST_LENGTH
        }
        return Int(result)
    }
}


```

