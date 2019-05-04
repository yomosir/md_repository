# protobuf

protobuf相关知识点记录

## 优点

相比于json，xml它更小，更快捷。protobuf适合做数据存储或者RPC数据格式交换。

## 基本语法(proto3)

```java
syntax = "proto3";

package package_name;
import "google/protobuf/any.proto";//引入其他的类

option java_package = "full_package_name";//文件所在包名
option java_outer_classname = "ClassName";//文件类名

message Message{//定义类名
    string version = 1; //定义属性
    string time = 2;
    int32 message_type = 3;
    google.protobuf.Any message = 4;//Any参数，可以填充其他使用protobuf定义的类
}
```

需要注意的时这个Any的使用方式

- 在java中使用Any对象中的`unpack()`方法进行解包成对应的类的对象，使用`Any.pack`方法去进行将其他的类转换成`Any`对象。
- 在js中：

```javaScript
var value = new proto.google.protobuf.Any
value.pack(clientMessage.serializeBinary(),'protocol.ClientMessage')
```

- 枚举

```java
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
```

## 生成类

可以在github上下载各个操作系统对应的生成程序，然后使用命令行进行生成。

```shell
protoc  --proto_path=IMPORT_PATH    --cpp_out=DST_DIR    path/to/file.proto
                                    --java_out=DST_DIR 
                                    --python_out=DST_DIR 
                                    --go_out=DST_DIR 
                                    --ruby_out=DST_DIR 
                                    --objc_out=DST_DIR 
                                    --csharp_out=DST_DIR 
```
