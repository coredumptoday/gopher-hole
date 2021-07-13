# 接口

```go
type Hash interface {
    io.Writer
    Sum(b []byte) []byte
    Reset()
    Size() int
    BlockSize() int
}
```

在`src/hash/hash.go`文件中提供了Hash接口的定义，所有Hash函数必须实现该接口，下面简单介绍一下每个方法的功能

`io.Writer`

作用：写入需要进行hash计算的任意长度数据，可以多次调用追加写入，**如果写入数据长度为BlockSize整数倍，则计算性能更好**

> Hash接口定义中内嵌了`io.Writer`接口，具体定义为`Write(p []byte) (n int, err error)`，但是和`io.Writer`不同的是，`error`永远为`nil`，使用时可以不做错误处理。**在扩展开发中也需要注意不要返回任何`error`**

`Sum`

作用：在不影响当前hash对象状态的基础上，计算hash值，并将结果添加到入参`b`的末尾，并且返回新生成的`[]byte`

> `Sum`也是可以重复调用的，可以在返回计算结果后继续调用`Write`追加数据并计算新的hash值

`Reset`

作用：将当前hash对象置为初始状态

`Size`

作用：返回当前hash算法计算结果占用的字节数

`BlockSize`

作用：返回当前hash算法计算时底层数据分组的字节数

除了`Hash`接口之外，hash函数还需要实现`src/encoding/encoding.go`中定义的两个接口，具体如下：

```go
type BinaryMarshaler interface {
    MarshalBinary() (data []byte, err error)
}
type BinaryUnmarshaler interface {
    UnmarshalBinary(data []byte) error
}
```

从接口名称就可以看出，这是用来进行`序列化`和`反序列化`的接口

上面说过，`Write`和`Sum`是可以多次调用，进行数据追加写入和hash值计算。那么`序列化`接口的实现是为了保存当时hash对象的内部运行状态，方便进行后续操作，在需要的时候进行`反序列化`，可以继续追加写入数据并计算，而不用考虑从头填充原始数据的问题

