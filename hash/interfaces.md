# 接口说明
### hash.Hash 
```go
type Hash interface {
    io.Writer
    Sum(b []byte) []byte
    Reset()
    Size() int
    BlockSize() int
}
```

在`src/hash/hash.go`文件中提供了Hash接口的定义，所有Hash类必须实现该接口，下面简单介绍一下每个方法的功能

#### io.Writer

作用：写入需要进行hash计算数据，可以多次调用追加写入，**每次写入的数据长度不限，但是写入数据长度为BlockSize整数倍时性能会更好**

> Hash接口定义中内嵌了`io.Writer`接口，具体定义为`Write(p []byte) (n int, err error)`，但是和`io.Writer`不同的是，`error`永远为`nil`，使用时可以不做错误处理。**自己需要实现`Hash`接口时也需要注意不要返回任何`error`**

#### Sum

作用：在不影响当前hash对象内部状态的基础上，计算hash值，并将结果`append`到入参`[]byte`的末尾，并返回`append`的结果

> `Sum`也是可以重复调用的，可以在返回计算结果后继续调用`Write`追加数据并计算新的hash值

#### Reset

作用：将当前hash对象内部状态置为初始值

#### Size

作用：返回当前hash算法计算结果占用的字节数

#### BlockSize

作用：返回当前hash算法计算时底层数据分组的字节数

### 序列化/反序列化

除了`Hash`接口之外，hash函数还需要实现`src/encoding/encoding.go`中定义的两个接口，接口定义如下：

```go
type BinaryMarshaler interface {
    MarshalBinary() (data []byte, err error)
}
type BinaryUnmarshaler interface {
    UnmarshalBinary(data []byte) error
}
```

从接口名称就可以看出，这是实现`序列化`和`反序列化`的接口

上面说过，`Write`和`Sum`是可以多次调用的，那么在某次调用之后就可以进行`序列化`操作，将hash对象当时的内部状态进行保存，在后续需要的时候进行`反序列化`，这样就能以当时内部状态为基础继续进行`Write`和`Sum`操作，而不用从头开始构造数据

