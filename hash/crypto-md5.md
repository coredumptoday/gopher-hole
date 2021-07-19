# crypto/md5

## 概述

{% hint style="info" %}
md5包实现了 [RFC 1321](https://rfc-editor.org/rfc/rfc1321.html) 中定义的 MD5 哈希算法
{% endhint %}

{% hint style="warning" %}
MD5 算法的密码学可靠性已经被攻破，不适用于安全应用
{% endhint %}

## 常量

{% code title="src/crypto/md5.go" %}
```go
const BlockSize = 64
const Size = 16
```
{% endcode %}

## 示例

{% tabs %}
{% tab title="包调用方式" %}
```go
data := []byte("These pretzels are making me thirsty.")
fmt.Printf("%x\n", md5.Sum(data))

//output
//b0804ec967f48520697662a204f5fe72
```
{% endtab %}

{% tab title="对象调用方式" %}
```go
h := md5.New()
h.Write([]byte("These pretzels are"))
h.Write([]byte(" making me thirsty."))
fmt.Printf("%x\n", h.Sum(nil))

//output
//b0804ec967f48520697662a204f5fe72
```
{% endtab %}
{% endtabs %}

## MD5处理流程

![MD5&#x5904;&#x7406;&#x6D41;&#x7A0B;&#x793A;&#x610F;&#x56FE;](../.gitbook/assets/md5-process.png)

1. 初始化第一个分组所需要的MD5值（4个uint32的魔数）
2. 整个待签名的数据需要先做分组处理，每64个byte为一组，如果末尾不足64个，需要进行填充
3. 将上一分组的MD5值（第一组没有上一分组就用初始化的MD5）和本分组数据作为输入，经过4轮位移计算生成新的MD5值，如果是最后的分组，该值就是最终的输出

## Go提供的MD5生成方式

Go文档中提供了两种生成MD5的方式

* 一种是调用`New`方法，生成一个`digest`结构体，然后调用`Write`方法进行数据写入，最后调用`digest`结构体所属的`Sum`方法进行输出
* 一种是直接调用包级别的`Sum`方法，传入待签名数据，返回MD5值

```go
package main

import (

  "crypto/md5"

  "fmt"

)

func main() {

  b := []byte("These pretzels are making me thirsty.")

  //方法一

  h := md5.New()

  h.Write(b)

  fmt.Printf("%x\n", h.Sum(nil))  //h.Sum返回[]byte

  //方法二

  fmt.Printf("%x\n", md5.Sum(b))  //md5.Sum返回[16]byte

}
```

既然有两种使用方式，那就应该有对应的应用场景，要不为什么要写两种？

## Go中MD5的实现方式

### 1. digest结构体

Go语言中MD5的计算都依赖于`digest`结构体

![](../.gitbook/assets/go-md5-digest.png)

如图`digest`结构体包含四个部分，标注的色块为MD5处理流程示意图中的相关位置

`digest.s`：存储的是MD5的值

`digest.x`：存储的是分组后待计算的当前分组内容

`digest.nx`：存储的是当前待处理分组中字符串的长度，也就是`len(digest.x)`的值

`digest.len`：存储的是待签名字符串的总长度

### 2. 方法调用流程

Go语言提供的两种方式底层调用流程基本是一样的，主要是通过三个函数进行签名的

`Reset`：对`digest`结构体中的变量附初始值

`Write`：写入数据并生成中间分组的md5签名，可以多次调用，追加写入，凑够分组就进行一次MD5计算，不够进行计算的数据存入`digest.x`中进行缓存，等待下次调用Write写入

`checkSum`：计算最后分组的填充数据，调用`Write`填充数据，格式化`digest.s`中数据输出MD5值

### 3. 两种方式应用场景

从源码实现角度分析两种方式的应用场景

`md5.Sum方式`：需要开发人员自己拼合\[\]byte后调用，在预先分配内存的场景下速度更快，但是伴随的就会有内存回收的问题

MD5虽然已经被证明有安全问题，但是在现行的接口签名中还是有部分使用。相应的签名格式定义问题就可能引发长度扩展攻击

