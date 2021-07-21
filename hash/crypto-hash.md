# crypto.Hash

## 概述

{% hint style="info" %}
`crypto.Hash`是一个通用的哈希类型，每种哈希算法的具体实现在各自哈希算法实现包中定义
{% endhint %}

## 常量

{% code title="src/crypto/crypto.go" %}
```go
const (
    MD4         Hash = 1 + iota // import golang.org/x/crypto/md4
    MD5                         // import crypto/md5
    SHA1                        // import crypto/sha1
    SHA224                      // import crypto/sha256
    SHA256                      // import crypto/sha256
    SHA384                      // import crypto/sha512
    SHA512                      // import crypto/sha512
    MD5SHA1                     // no implementation; MD5+SHA1 used for TLS RSA
    RIPEMD160                   // import golang.org/x/crypto/ripemd160
    SHA3_224                    // import golang.org/x/crypto/sha3
    SHA3_256                    // import golang.org/x/crypto/sha3
    SHA3_384                    // import golang.org/x/crypto/sha3
    SHA3_512                    // import golang.org/x/crypto/sha3
    SHA512_224                  // import crypto/sha512
    SHA512_256                  // import crypto/sha512
    BLAKE2s_256                 // import golang.org/x/crypto/blake2s
    BLAKE2b_256                 // import golang.org/x/crypto/blake2b
    BLAKE2b_384                 // import golang.org/x/crypto/blake2b
    BLAKE2b_512                 // import golang.org/x/crypto/blake2b
    maxHash
)
```
{% endcode %}

常量中定义了`crypto.Hash`支持的哈希算法，以及具体实现所在位置，其中`maxHash`作为边界校验使用

{% hint style="warning" %}
由于`maxHash`的限制，如果不自己编译Go工具链的话是没有办法自己注册对应的`crypto.Hash`常量的
{% endhint %}

## 示例

{% tabs %}
{% tab title="MD5" %}
```go
if crypto.MD5.Available() {
    h := crypto.MD5.New()
    h.Write([]byte("These pretzels are"))
    h.Write([]byte(" making me thirsty."))
    fmt.Printf("%x, len: %d\n", h.Sum(nil), crypto.MD5.Size() * 2)
}else {
    fmt.Printf("Need import %s package\n", crypto.MD5.String())
}
```
{% endtab %}

{% tab title="SHA1" %}
```go
if crypto.SHA1.Available() {
    h := crypto.SHA1.New()
    h.Write([]byte("These pretzels are"))
    h.Write([]byte(" making me thirsty."))
    fmt.Printf("%x, len: %d\n", h.Sum(nil), crypto.SHA1.Size() * 2)
}else {
    fmt.Printf("Need import %s package\n", crypto.SHA1.String())
}
```
{% endtab %}

{% tab title="SHA256" %}
```go
if crypto.SHA256.Available() {
    h := crypto.SHA256.New()
    h.Write([]byte("These pretzels are"))
    h.Write([]byte(" making me thirsty."))
    fmt.Printf("%x, len: %d\n", h.Sum(nil), crypto.SHA256.Size() * 2)
}else {
    fmt.Printf("Need import %s package\n", crypto.SHA256.String())
}
```
{% endtab %}

{% tab title="SHA512" %}
```go
if crypto.SHA512.Available() {
    h := crypto.SHA512.New()
    h.Write([]byte("These pretzels are"))
    h.Write([]byte(" making me thirsty."))
    fmt.Printf("%x, len: %d\n", h.Sum(nil), crypto.SHA512.Size() * 2)
}else {
    fmt.Printf("Need import %s package\n", crypto.SHA512.String())
}
```
{% endtab %}
{% endtabs %}

`crypto.Hash`以`对象调用方式`提供了哈希值计算通用的方法，便于封装和二次开发

{% hint style="warning" %}
`crypto.Hash`提供的方法进行哈希值计算时需要导入具体的实现包，运行其中的`init`方法完成注册
```go
import (
    _ "crypto/md5"
    _ "crypto/sha1"
    _ "crypto/sha256"
    _ "crypto/sha512"
)

```
{% endhint %}

{% hint style="warning" %}
`crypto.Hash`在使用过程中会校验不通过会触发`panic`，需要做好对应的处理工作
{% endhint %}

## 注册

{% tabs %}
{% tab title="MD5" %}
{% code title="src/crypto/md5/md5.go" %}
```go
func init() {
    crypto.RegisterHash(crypto.MD5, New)
}
```
{% endcode %}
{% endtab %}

{% tab title="SHA1" %}
{% code title="src/crypto/sha1/sha1.go" %}
```go
func init() {
    crypto.RegisterHash(crypto.SHA1, New)
}
```
{% endcode %}
{% endtab %}

{% tab title="SHA256" %}
{% code title="src/crypto/sha256/sha256.go" %}
```go
func init() {
    crypto.RegisterHash(crypto.SHA224, New224)
    crypto.RegisterHash(crypto.SHA256, New)
}
```
{% endcode %}
{% endtab %}

{% tab title="SHA512" %}
{% code title="src/crypto/sha512/sha512.go" %}
```go
func init() {
    crypto.RegisterHash(crypto.SHA384, New384)
    crypto.RegisterHash(crypto.SHA512, New)
    crypto.RegisterHash(crypto.SHA512_224, New512_224)
    crypto.RegisterHash(crypto.SHA512_256, New512_256)
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

每种哈希算法在具体的实现包中调用`crypto.RegisterHash`方法，将自己的实例化方法进行了注册
