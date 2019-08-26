# Protobuf： C++ 产生的代码简析（Proto3）

## 前言
参考官方[C++ Generated Code](https://developers.google.cn/protocol-buffers/docs/reference/cpp-generated)文档，主要是参考了官方文档。

主要描述protocol buffer编译器为**Proto3**协议定义生成的C++代码。

## 编译器调用

之前的文档已经描述过，不在关注，如有需要请查看之前的文档。


## Messages

一个简单的Message：

```c
message Foo {}
```

 protocol buffer 编译器生成一个名为foo的类，该类公开继承自` google::protobuf::Message`。该类是一个具体的类；没有任何纯虚拟方法是未实现的。`Message`中是虚拟的但不是纯虚拟的方法可能会被`Foo `覆盖，这取决于优化模式。默认情况下，`Foo `实现所是以最大速度。但是，如果`.proto`文件包含行：

```c
option optimize_for = CODE_SIZE;
```

然后，`Foo` 将覆盖运行所需的最小方法集，并依赖于其余的基于反射的实现。减小了生成的代码的大小，但也降低了性能。或者，如果`.proto`文件包含：

```c
option optimize_for = LITE_RUNTIME;
```

然后`Foo`包括所有方法的快速实现，但将继续实现`google::protobuf::MessageLite`接口，该接口只包含消息方法的一个子集。特别是，它不支持描述符或反射。但是，在这种模式下，生成的代码只需要与`libprotobuf-lite.so`（Windows上的`libprotobuf-lite.lib`）链接，而不需要与`libprotobuf.so`（`libprotobuf.lib`）链接。“Lite”库比完整库小得多，更适合于资源受限的系统，如移动电话。

不应该创建自己的`Foo `子类。如果子类化该类并重写虚拟方法，则override可能被忽略，因为许多生成的方法调用都是de-virtualized的，以提高性能。

` Message`接口定义了允许检查、操作、读取或写入整个消息的方法，包括从解析和序列化为二进制字符串。

- `bool ParseFromString(const string& data)`：解析给定序列化二进制字符串（也称为wire format）中的消息。

- `bool SerializeToString(string* output) const`：将给定消息序列化为二进制字符串。

- `string DebugString()`：返回一个字符串，该字符串给出proto的“文本格式”表示形式（仅用于调试）。


除了这些方法之外，foo类还定义了以下方法：

- `Foo()`：默认构造函数。

- `~Foo()`：默认析构函数。

- `Foo(const Foo& other)`：复制构造函数。

- `Foo& operator=(const Foo& other)`：赋值运算符。

- `void Swap(Foo* other)`：用另一条消息交换内容。

- `const UnknownFieldSet& unknown_fields() const`：返回分析此消息时遇到的一组未知字段。

- `UnknownFieldSet* mutable_unknown_fields()`：返回一个指针，指向解析此消息时遇到的一组可变未知字段。

类还定义了以下静态方法：

- `static const Descriptor* descriptor()`：返回类型的描述符。这包含关于类型的信息，包括它有哪些字段以及它们的类型。这可以与反射一起使用，以编程方式检查字段。

- `static const foo&default_instance（）`：返回一个与新构造的foo实例相同的foo的const singleton实例（所有单字段均未设置，所有重复字段均为空）。请注意，消息的默认实例可以通过调用其` New() `方法用作工厂。

message 可以在另一个message 中声明。例如：`message Foo { message Bar { } }`

在这种情况下，编译器生成两个类：`Foo` 和`Foo_Bar`。此外，编译器在`Foo `内生成`typedef `，如下所示：

```c
typedef Foo_Bar Bar;
```
可以像`Foo::Bar`一样使用嵌套类型的类。但是，请注意C++不允许声明嵌套类型。如果要在另一个文件中的声明`Bar`并使用该声明，则必须将其标识为`Foo_Bar`。


## 字段
除了上述描述的方法外，protocol buffer编译器还为`.proto`文件中message定义的每个字段生成一组访问器方法。

除了访问器方法之外，编译器还为每个包含字段号的字段生成一个整型常量。常量名是字母`k`，后跟转换为驼峰命名法的字段名，后跟`FieldNumber`。例如，给定字段`optional int32 foo_bar = 5;`，编译器将生成常量`static const int kFooBarFieldNumber = 5;`。

对于返回`const `引用的字段访问器，当对消息进行下一次修改访问时，该引用可能会失效。这包括调用任何字段的任何非常量访问器、调用从消息继承的任何非常量方法或通过其他方式修改消息（例如，使用`message`作为`Swap()`的参数）。相应地，如果没有同时对message进行修改访问，则返回引用的地址在访问器的不同调用之间是相同的。

对于返回指针的字段访问器，当对消息进行下一次修改或非修改访问时，该指针可能无效。这包括，常量、调用任何字段的任何访问器、调用从消息继承的任何方法或通过其他方式访问消息（例如，使用复制构造函数复制消息）。相应地，返回指针的值在访问器的两个不同调用中永远不会保证是相同的。

### 单个数字字段（proto3）

对于此字段定义：

```c
Int32 foo=1；
```

编译器将生成以下访问器方法：

- `int32 foo() const: `：返回字段的当前值。如果未设置字段，则返回0。

- `void set_foo(int32 value)`：设置字段的值。调用此函数后，`foo()`将返回值。

- `void clear_foo()`：清除字段值。调用此函数后，`foo()`将返回0。


### 单个字符串字段（proto3）

对于这些字段定义中的任何一个：

```c
string foo = 1;
bytes foo = 1;
```

编译器将生成以下访问器方法：

- `const string& foo() const`：返回字段的当前值。如果未设置字段，则返回空字符串/空字节。

- `void set_foo(const string& value)`：设置字段的值。调用此函数后，`foo()`将返回值的副本。

- `void set_foo(string&& value)`（C++ 11及以上）：设置字段的值，从传递的字符串中赋值。调用此函数后，`foo()`将返回值的副本。

- `void set_foo(const char* value)`：使用C样式的以空结尾的字符串设置字段的值。调用此函数后，`foo()`将返回值的副本。

- `void set_foo(const char* value, int size)`：如上所述，但字符串大小是显式给定的，而不是通过查找空终止符字节来确定的。

- `string* mutable_foo()`：返回存储字段值的可变字符串对象的指针。如果在调用之前未设置字段，则返回的字符串将为空。调用此函数后，foo（）将返回写入给定字符串的任何值。

- `void clear_foo()`：清除字段值。调用此函数后，`foo()`将返回空字符串/空字节。

- `void set_allocated_foo(string* value)`：将string对象设置为字段，并释放前一个字段值（如果存在）。如果字符串指针不为空，则消息将取得已分配字符串对象的所有权。消息可以随时删除已分配的字符串对象，因此对该对象的引用可能会无效。否则，如果值为空，则行为与调用clear_foo（）相同。

- `string* release_foo()`：释放字段的所有权并返回string对象的指针。调用此函数后，调用者将获得所分配字符串对象的所有权，`foo()`将返回空字符串/空字节。

### 单个枚举字段（proto3）

```c
enum Bar {
  BAR_VALUE = 0;
  OTHER_VALUE = 1;
}
```
定义：

```c
Bar foo = 1;
```

编译器将生成以下访问器方法：

- `Bar foo() const`：返回字段的当前值。如果未设置字段，则返回默认值（0）。

- `void set_foo(Bar value)`：设置字段的值。调用此函数后，`foo()`将返回值。

- `void clear_foo()`：清除字段值。调用此函数后，`foo()`将返回默认值。


### 单个嵌入Message字段
给定message 定义
```c
message Bar {}
```
对于这些字段定义中的任何一个：
```c
//proto3
Bar foo = 1;
```
编译器将生成以下访问器方法：

- `bool has_foo() const `:如果设置了字段，则返回true。

- `const Bar& foo() const `：返回字段的当前值。如果未设置字段，则返回一个未设置任何字段的`Bar`（可能为`Bar::default_instance()`）。

- `Bar* mutable_foo()`：返回存储字段值的mutable `Bar`对象的指针。如果在调用之前未设置字段，则返回的栏将不设置任何字段（即，它将与新分配的`Bar`相同）。调用此函数后，`has_foo()`将返回true，`foo()`将返回对同一个`Bar`实例的引用。

- `void clear_foo()`：清除字段值。调用后，`has_foo()`将返回false，`foo()`将返回默认值。

- `void set_allocated_foo(Bar* bar)`：将`Bar `对象设置为字段，并释放上一个字段值（如果存在）。如果`Bar `指针不为空，则消息将取得分配的`Bar `对象的所有权，并且`has_foo()`将返回true。否则，如果条为空，则行为与调用`clear_foo()`相同。

- `Bar* release_foo()`：释放字段的所有权并返回`Bar `对象的指针。调用此函数后，调用者将获得分配的`Bar `对象的所有权，`has_foo()`将返回false，`foo()`将返回默认值。

### Repeated 数字字段

对于此字段定义：

```c
repeated int32 foo = 1;
```


编译器将生成以下访问器方法：

- `int foo_size() const`：返回字段中当前元素的数目。

- `int32 foo(int index) const`：返回给定索引处的元素。使用` [0, foo_size()) `之外的索引调用此方法会产生未定义的行为。

- `void set_foo(int index, int32 value)`：在给定的从零开始的索引处设置元素的值。

- `void add_foo(int32 value)`：将一个新元素附加到具有给定值的字段中。

- `void clear_foo()`：从字段中删除所有元素。调用此函数后，`foo_size()`将返回零。

- `const RepeatedField<int32>& foo() const`：返回存储字段元素的基础`RepeatedField`。这个容器类提供类似STL的迭代器和其他方法。

- `RepeatedField<int32>* mutable_foo()`：返回一个指向存储字段元素的基础mutable ` RepeatedField`的指针。这个容器类提供类似STL的迭代器和其他方法。

### Repeated 字符串字段

对于这些字段定义之一：

```c
repeated string foo = 1;
repeated bytes foo = 1;
```

编译器将生成以下访问器方法：

- `int foo_size() const`：返回字段中当前元素的数目。

- `const string& foo(int index) const`：返回给定索引处的元素。使用`[0, foo_size()) `之外的索引调用此方法会产生未定义的行为。

- `void set_foo(int index, const string& value)`：在给定的从零开始的索引处设置元素的值。

- `void set_foo(int index, const char* value)`：使用C样式的以空结尾的字符串在给定的从零开始的索引处设置元素的值。

- `void set_foo(int index, const char* value, int size)`：与上面类似，但字符串大小是显式给定的，而不是通过查找空终止符字节来确定的。

- `string* mutable_foo(int index)`：返回一个指向mutable `string`对象的指针，该对象在给定的从零开始的索引处存储元素的值。使用` [0, foo_size()) `之外的索引调用此方法会产生未定义的行为。

- `void add_foo(const string& value)`：将一个新元素附加到具有给定值的字段中。

- `void add_foo(const char* value)`：使用C样式的以空结尾的字符串向字段追加一个新元素。

- `void add_foo(const char* value, int size)`：如上所述，但字符串大小是显式给定的，而不是通过查找空终止符字节来确定的。

- `string* add_foo()`：添加新的空字符串元素并返回指向它的指针。

- `void clear_foo()`：从字段中删除所有元素。调用此函数后，`foo_size()`将返回零。

- `const RepeatedPtrField<string>& foo() const`：返回存储字段元素的基础`RepeatedPtrField`。这个容器类提供类似STL的迭代器和其他方法。

- `RepeatedPtrField<string>* mutable_foo()`：返回一个指向存储字段元素的基础mutable ` RepeatedPtrField`的指针。这个容器类提供类似STL的迭代器和其他方法。

### Repeated 枚举字段

给定枚举类型：

```c
enum Bar {
  BAR_VALUE = 0;
  OTHER_VALUE = 1;
}
```

对于此字段定义：

```c
repeated Bar foo = 1;
```

编译器将生成以下访问器方法：

- `int foo_size() const`：返回字段中当前元素的数目。

- `Bar foo(int index) const`：返回给定索引处的元素。使用` [0, foo_size()) `之外的索引调用此方法会产生未定义的行为。

- `void set_foo(int index, Bar value)`：在给定的从零开始的索引处设置元素的值。在调试模式下（即未定义`NDEBUG`），如果值与为`Bar`定义的任何值不匹配，则此方法将中止进程。

- `void add_foo(Bar value)`：将一个新元素附加到具有给定值的字段中。在调试模式下（即未定义`NDEBUG`），如果值与为`Bar`定义的任何值不匹配，则此方法将中止进程。

- `void clear_foo()`：从字段中删除所有元素。调用此函数后，`foo_size() `将返回零。

- `const RepeatedField<int>& foo() const`：返回存储字段元素的基础`RepeatedField`。这个容器类提供类似STL的迭代器和其他方法。

- `RepeatedField<int>* mutable_foo()`：返回一个指向存储字段元素的基础mutable `RepeatedField`的指针。这个容器类提供类似STL的迭代器和其他方法。


### Repeated嵌入的消息字段
给定消息类型：

```c
message Bar {}
```

对于此字段定义：

```c
repeated Bar foo = 1;
```

编译器将生成以下访问器方法：



- `int foo_size() const`：返回字段中当前元素的数目。

- `const Bar& foo(int index) const`：返回给定索引处的元素。使用 `[0, foo_size())` 之外的索引调用此方法会产生未定义的行为。

- `Bar* mutable_foo(int index)`：返回指向mutable `Bar`对象的指针，该对象在给定的基于零的索引处存储元素的值。使用 `[0, foo_size())` 之外的索引调用此方法会产生未定义的行为。

- `Bar* add_foo()`：添加新元素并返回指向它的指针。返回的`Bar`是可变的，不会设置任何字段（即，它将与新分配的条相同）。

- `void clear_foo()`：从字段中删除所有元素。调用此函数后，`foo_size()`将返回零。

- `const RepeatedPtrField<Bar>& foo() const`：返回存储字段元素的基础 `RepeatedPtrField` 。这个容器类提供类似STL的迭代器和其他方法。

- `RepeatedPtrField<Bar>* mutable_foo()`：返回一个指向存储字段元素的基础mutable  `RepeatedPtrField` 的指针。这个容器类提供类似STL的迭代器和其他方法。


### Oneof 数字字段
对于此字段定义：

```c
oneof oneof_name {
    int32 foo = 1;
    ...
}
```

编译器将生成以下访问器方法：

- `int32 foo() const`：如果case之一是`kFoo`，则返回字段的当前值。否则，返回默认值。

- `void set_foo(int32 value)`：

	- 如果在oneof 中设置了任何其他oneof 字段，则调用`clear_oneof_name()`。

	- 设置此字段的值并将oneof case为`kFoo`。


- `void clear_foo():`：

	- 如果oneof case不是`kFoo`，则不会有任何改变。

	- 如果oneof case是`kFoo`，则清除字段值和oneof case.。


### Oneof 字符字段
对于此字段定义：

```c
oneof oneof_name {
    string foo = 1;
    …
}
oneof oneof_name {
    bytes foo = 1;
    ….
}
```
编译器将生成以下访问器方法：

- `const string& foo() const`：如果 oneof case 是`kFoo`，则返回字段的当前值。否则，返回默认值。

- `void set_foo(const string& value)`：

	- 如果在oneof 中设置了任何其他oneof 字段，则调用`clear_oneof_name()`。

	- 设置此字段的值并将 oneof case 设置为`kFoo`。


- `void set_foo(const char* value)`：

	- 如果在 oneof 中设置了任何其他 oneof 字段，则调用`clear_oneof_name()`。

	- 使用C样式的以空结尾的字符串设置字段的值，并将oneof case设置为`kFoo`。


- `void set_foo(const char* value, int size)`：如上所述，但字符串大小是显式给定的，而不是通过查找空终止符字节来确定的。

- `string* mutable_foo()`：

	- 如果在oneof 中设置了任何其他oneof字段，则调用`clear_oneof_name()`。

	- 将oneof case设置为`kFoo `，并返回一个指向存储字段值的可变字符串对象的指针。如果oneof case在调用之前不是`kFoo `，则返回的字符串将为空（不是默认值）。

- `void clear_foo()`：

	- 如果oneof case不是`kFoo`，则不会发生任何变化。

	- 如果oneof case是`kFoo`，则释放字段并清除oneof case。

- `void set_allocated_foo(string* value)`：

	- 调用`clear_oneof_name()`。
	- 如果字符串指针不为空：将字符串对象设置为字段，并将oneof case设置为`kFoo`。

- `string* release_foo()`：

	- 如果其中 oneof case不是`kFoo`，则返回空值。

	- 清除 oneof case，释放字段的所有权并返回字符串对象的指针。调用此函数后，调用者将获得分配字符串对象的所有权。


### Oneof 枚举字段
给定的枚举类型：

```c
enum Bar {
  BAR_VALUE = 0;
  OTHER_VALUE = 1;
}
```

对于此字段定义：

```c
oneof oneof_name {
    Bar foo = 1;
    ...
}
```
编译器将生成以下访问器方法：


- `Bar foo() const`：如果oneof case是`kFoo`，则返回字段的当前值。否则，返回默认值。

- `void set_foo(Bar value)`：

	- 如果oneof 中设置了任何其他oneof 字段，则调用`clear_oneof_name()`。

	- 设置此字段的值并将 oneof case 设置为`kFoo`。

	- 在调试模式下（即未定义`NDEBUG`），如果值与为`Bar`定义的任何值不匹配，则此方法将中止进程。

- `void clear_foo()`：

	- 如果oneof case 不是`kFoo`，则不会发生任何变化。

	- 如果oneof case 是`kFoo`，则清除字段和`kFoo`的值。

### Oneof嵌入Message 字段

给定的枚举类型：

```c
message Bar {}
```

对于此字段定义：

```c
oneof oneof_name {
    Bar foo = 1;
    ...
}
```

编译器将生成以下访问器方法：

- `bool has_foo() const`:如果oneof case是`kFoo`，则返回true。

- `const Bar& foo() const`：如果oneof case是`kFoo`，则返回字段的当前值。否则，返回`Bar::default_instance()`。

- `Bar* mutable_foo()`：

	- 如果在oneof中设置了任何其他oneof 字段，则调用`clear_oneof_name()`。

	- 将oneof case设置为`kFoo `，并返回一个指向存储字段值的可变`Bar`对象的指针。如果其中一个事例在调用之前不是`kFoo `，则返回的`Bar`将不设置任何字段（即，它将与新分配的`Bar`相同）。

	- 调用后，`has_foo()`将返回true，`foo()`将返回对同一个`Bar`实例的引用，`oneof_name_case()`将返回`kFoo `。

- `void clear_foo()`：

	- 如果oneof case不是`kFoo`，则不会发生任何变化。

	- 如果oneof case等于`kFoo`，则释放字段并清除oneof case。`has_foo()`将会返回否，`foo（）`将返回默认值并且` oneof_name_case()`将返回`ONEOF_NAME_NOT_SET`

- `void set_allocated_foo(Bar* bar)`：

	- 调用`clear_oneof_name()`。

	- 如果`Bar `指针不是空：将`Bar `对象设置为字段，并将oneof case设置为`kFoo`。消息拥有分配的`Bar `对象的所有权，` has_foo() `将返回true，`oneof_name_case()`将返回kfoo。

	- 如果指针为空，`has_foo()`将返回false，`oneof_name_case()`将返回`ONEOF_NAME_NOT_SET`。（该行为类似于调用`clear_oneof_name()`）

- `Bar* release_foo()`：

	- 如果其中一个case不是`kFoo`，则返回空值。

	- 如果oneof case是`kFoo`，则清除oneof case，释放字段的所有权并返回`Bar`对象的指针。调用此函数后，调用者将获得分配的条对象的所有权，`has_foo()`将返回false，foo（）将返回默认值，`oneof_name_case()`将返回`ONEOF_NAME_NOT_SET`。


### Map 字段

对于此字段定义：

```c
map<int32, int32> weight = 1;
```
编译器将生成以下访问器方法：


- `const google::protobuf::Map<int32, int32>& weight();`：返回不可变映射。

- `google::protobuf::Map<int32, int32>* mutable_weight();`：返回可变映射。

`google::protobuf::Map`是protocol buffer中用于存储映射字段的特殊容器类型。从下面的接口可以看到，它使用了一个常用的`std::map` 和 `std::unordered_map`方法子集 。

```c
template<typename Key, typename T> {
class Map {
  // Member types
  typedef Key key_type;
  typedef T mapped_type;
  typedef MapPair< Key, T > value_type;

  // Iterators
  iterator begin();
  const_iterator begin() const;
  const_iterator cbegin() const;
  iterator end();
  const_iterator end() const;
  const_iterator cend() const;
  // Capacity
  int size() const;
  bool empty() const;

  // Element access
  T& operator[](const Key& key);
  const T& at(const Key& key) const;
  T& at(const Key& key);

  // Lookup
  int count(const Key& key) const;
  const_iterator find(const Key& key) const;
  iterator find(const Key& key);

  // Modifiers
  pair<iterator, bool> insert(const value_type& value);
  template<class InputIt>
  void insert(InputIt first, InputIt last);
  size_type erase(const Key& Key);
  iterator erase(const_iterator pos);
  iterator erase(const_iterator first, const_iterator last);
  void clear();

  // Copy
  Map(const Map& other);
  Map& operator=(const Map& other);
}
```
添加数据的最简单方法是使用普通映射语法，例如：

```c
std::unique_ptr<ProtoName> my_enclosing_proto(new ProtoName);
(*my_enclosing_proto->mutable_weight())[my_key] = my_value;
```
`pair<iterator, bool> insert(const value_type& value)` 将隐式导致`value_type`实例的深度复制。将新值插入`google::protobuf::Map`的最有效方法如下：

```c
T& operator[](const Key& key): map[new_key] = new_mapped;
```

#### 在标注Map使用`google::protobuf::Map`

`google::protobuf::Map`支持与`std::map`和` std::unordered_map`相同的迭代器API。如果不想直接使用`google::protobuf::Map`，可以通过执行以下操作将`google::protobuf::Map`转换为标准map ：

```c
std::map<int32, int32> standard_map(message.weight().begin(),
                                    message.weight().end());
```

请注意，这将对整个map进行深度复制。

还可以从标准map构造`google::protobuf::Map`，如下所示：

```c
google::protobuf::Map<int32, int32> weight(standard_map.begin(), standard_map.end());
```

## Any

```c
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  google.protobuf.Any details = 2;
}
```

在生成的代码中，`details `字段的getter返回是`google::protobuf::Any`的一个实例。这提供了以下打包和解包`Any`值的特殊方法：

```c
class Any {
 public:
  // Packs the given message into this Any using the default type URL
  // prefix “type.googleapis.com”.
  void PackFrom(const google::protobuf::Message& message);

  // Packs the given message into this Any using the given type URL
  // prefix.
  void PackFrom(const google::protobuf::Message& message,
                const string& type_url_prefix);

  // Unpacks this Any to a Message. Returns false if this Any
  // represents a different protobuf type or parsing fails.
  bool UnpackTo(google::protobuf::Message* message) const;

  // Returns true if this Any represents the given protobuf type.
  template<typename T> bool Is() const;
}
```
## Oneof

```c
oneof oneof_name {
    int32 foo_int = 4;
    string foo_string = 9;
    ...
}
```
编译器将生成以下访问器方法：

```c
enum OneofNameCase {
  kFooInt = 4,
  kFooString = 9,
  ONEOF_NAME_NOT_SET = 0
}
```

此外，它还将生成以下方法：

- `OneofNameCase oneof_name_case() const`：:返回指示设置了哪个字段的枚举。如果未设置任何名称，则返回`ONEOF_NAME_NOT_SET`。

- `void clear_oneof_name()`：如果oneof字段集使用指针（消息或字符串），则释放对象，并将oneof case设置为`ONEOF_NAME_NOT_SET`。

##  枚举

```c
enum Foo {
  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```
protocol buffer 编译器将生成具有相同值集的称为`Foo`的C++枚举类型。此外，编译器将生成以下函数：

- `const EnumDescriptor* Foo_descriptor()`：返回类型的描述符，其中包含有关此枚举类型定义的值的信息。

- `bool Foo_IsValid(int value)`：如果给定的数值与`Foo`定义的值之一匹配，则返回true。在上面的示例中，如果输入为0、5或1234，则返回true。

- `const string& Foo_Name(int value)`：返回给定数值的名称。如果不存在此类值，则返回空字符串。如果多个值具有此数字，则返回定义的第一个值。在上面的示例中，`Foo_Name(5)`将返回“`VALUE_B`”。

- `bool Foo_Parse(const string& name, Foo* value)`：如果`name`是此枚举的有效值名称，则将该值赋给value并返回true。否则返回false。在上面的示例中，`Foo_Parse("VALUE_C", &some_foo`将返回true，并将`some_foo`设置为1234。

- `const Foo Foo_MIN`：枚举的最小有效值（示例中的`VALUE_A` ）。

- `const Foo Foo_MAX`：枚举的最大有效值（示例中的`VALUE_C` ）。

- `const int Foo_ARRAYSIZE`：始终定义为`Foo_MAX + 1`。

> **在switch语句中使用proto3枚举时要小心**。Proto3枚举是开放的枚举类型，可能值超出指定符号的范围。解析proto3消息时将保留无法识别的枚举值，并由枚举字段访问器返回。没有默认大小写的proto3枚举上的switch语句将无法捕获所有大小写，即使列出了所有已知字段。这可能导致意外行为，包括数据损坏和运行时崩溃。添加一个默认的大小写，或者在开关外部显式调用`Foo_IsValid`（int）来处理未知的枚举值。


可以在消息类型内定义枚举。在这种情况下，protocol buffer编译器生成代码，使其看起来枚举类型本身被声明为嵌套在消息的类中。`Foo_descriptor()`和`Foo_IsValid()`函数声明为静态方法。实际上，枚举类型本身及其值是在全局作用域中声明的，名称不完整，并通过typedef和一系列常量定义导入到类的作用域中。这样做只是为了解决声明排序的问题。


## Arena Allocation

Arena Allocation是一个C++独到的特性，它帮助您优化内存使用，并在使用协议缓冲区时提高性能。在.PROTO中启用Arena Allocation，为您的C++生成代码添加了与arenas协同工作的附加代码。


## 服务

如果.proto文件包含以下行：

```c
option cc_generic_services = true;
```

然后protocol buffer编译器将根据文件中的服务定义生成代码。但是，所生成的代码可能不受欢迎，因为它没有绑定到任何特定的RPC系统，因此需要更多级别的间接寻址，以便为一个系统定制代码。如果不希望生成此代码，请将此行添加到文件：

```c
option cc_generic_services = false;
```

如果上述两行都没有给出，则选项默认为false，因为通用服务已被弃用。

> （请注意，在2.4.0之前，选项默认为true）

基于.proto语言服务定义的RPC系统应该提供插件来生成适合系统的代码。这些插件可能需要禁用抽象服务，以便它们可以生成自己的同名类。插件是2.3.0版（2010年1月）中的新插件。

本节的其余部分描述启用抽象服务时protocol buffer 编译器生成的内容。


### 接口

给定服务定义：

```c
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```
protocol buffer编译器将生成一个类`Foo`来表示此服务。`Foo`将为服务定义中定义的每个方法提供一个虚拟方法。在这种情况下，方法`Bar`定义为：

```c
virtual void Bar(RpcController* controller, const FooRequest* request,
                 FooResponse* response, Closure* done);
```

这些参数等同于`Service::CallMethod()`的参数，只是`method`参数是隐含的，`request`和`response`指定了它们的确切类型。

这些生成的方法是虚拟的，但不是纯虚拟的。默认实现只调用`controller->SetFailed()`，并显示一条错误消息，指示该方法未实现，然后调用`done`回调。在实现自己的服务时，必须将生成的服务子类化，并根据需要实现其方法。

`Foo`子类化服务接口。protocol buffer编译器自动生成`Service`方法的实现，如下所示：

- `GetDescriptor`：返回服务的`ServiceDescriptor`。

- `CallMethod`：根据提供的方法描述符确定要调用的方法，并直接调用它，将请求和响应消息对象向下强制转换为正确的类型。

- `GetRequestPrototype`和`GetResponsePrototype`：返回给定方法的正确类型的请求或响应的默认实例。

还生成以下静态方法：

- `static ServiceDescriptor descriptor()`：返回类型的描述符，其中包含有关此服务具有哪些方法及其输入和输出类型的信息。



### Stub

protocol buffer编译器还生成每个服务接口的“stub”实现，希望向实现该服务的服务器发送请求的客户机使用该实现。对于`Foo`服务（上述），将定义存根实现`Foo_Stub`。与嵌套消息类型一样，使用`typedef`，以便`Foo_Stub`也可以称为`Foo::Stub`。

`Foo_Stub`是foo的一个子类，它还实现以下方法：

- `Foo_Stub(RpcChannel* channel)`：构造一个新的stub，它在给定的通道上发送请求。

- `Foo_Stub(RpcChannel* channel, ChannelOwnership ownership)`：构造一个新的stub，它在给定的通道上发送请求，并可能拥有该通道。如果`ownership`为`Service::STUB_OWNS_CHANNEL`，那么当stub对象被删除时，它也将删除该通道。

- `RpcChannel* channel()`：返回传递给构造函数的stub的通道。

stub还将每个服务的方法作为通道的包装器来实现。调用其中一个方法只需调用`channel->CallMethod()`。

 Protocol Buffer 库不包括RPC实现。但是，它包含了将生成的服务类连接到您选择的任意RPC实现所需的所有工具。只需要提供`RpcChannel`和`RpcController`的实现。

## 插件插入点

代码生成器插件要扩展C++代码生成器的输出，可以使用给定的插入点名称插入下列类型的代码。除非另有说明，否则每个插入点都出现在.pb.cc文件和.pb.h文件中。

`includes`：包括指令。

`namespace_scope`：属于文件包/名称空间，但不属于任何特定类的声明。出现在所有其他命名空间范围代码之后。

`global_scope`：属于顶级的声明，在文件的命名空间之外。出现在文件的最后。

`class_scope:TYPENAME`:属于消息类的成员声明。`TYPENAME `是完整的协议名，例如`package.MessageType`。出现在类中所有其他公开声明之后。这个插入点只出现在.pb.h文件中。

> 不要生成依赖于标准代码生成器声明的私有类成员的代码，因为这些实现细节可能在未来的Protocol Buffer版本中发生更改。

