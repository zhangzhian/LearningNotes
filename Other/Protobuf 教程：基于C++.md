# Protobuf 教程：基于C++

## 前言
参考官方[Protocol Buffer Basics: C++](https://developers.google.cn/protocol-buffers/docs/cpptutorial)文档，主要是参考了官方文档。

本文使用C++实现一个简单的应用程序，介绍 protocol buffer C++ API，并展示创建和使用.proto文件的基础知识。还提供了完整示例代码。

该教程是基于proto2的

## 简介
Protocol buffer是一种灵活、高效、自动化的解决方案。

使用 protocol buffer，可以编写要存储的数据结构的.proto描述。由此， protocol buffer编译器创建了一个类，该类以有效的二进制格式实现 protocol buffer数据的自动编码和解析。生成的类为构成 protocol buffer的字段提供getter和setter，并负责作为一个单元读取和写入协议缓冲区的详细信息。

重要的是， protocol buffer格式支扩展格式的思想，这样代码仍然可以读取用旧格式编码的数据。

## 定义协议格式

`.proto`文件中的定义：为要序列化的每个数据结构添加消息，然后为消息中的每个字段指定名称和类型。这里是定义消息的`.proto`文件，`addressbook.proto`。

```c
syntax = "proto2";

package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

语法类似于C++或Java。

`.proto`文件以包声明开头，这有助于防止不同项目之间的命名冲突。在C++中，生成的类将放置在与包名匹配的命名空间中。

接下来，是message定义。message只是包含一组类型化字段的集合。许多标准的简单数据类型作为字段类型，包括`bool`、`int32`、`float`、`double`和`string`。还可以使用其他message类型作为字段类型向消息添加进一步的结构（在上面的示例中，`Person` message包含`PhoneNumber` messages，而`AddressBook` message包含`Person` messages）。可以定义嵌套在其他message中的message类型（`PhoneNumber` 类型是在Person中定义的）。如果希望某个字段具有预定义值列表中的一个值，也可以定义枚举类型（要指定电话号码可以是`MOBILE`、`HOME`或`WORK`）。

每个元素上的`“=1”`、`“=2”`标记标识字段在二进制编码中使用的唯一“标记”。标签号1-15需要比更高的数字少一个字节进行编码，因此作为优化，您可以决定将这些标签用于常用或重复的元素，而将标签16和更高的标签用于不常用的可选元素。重复字段中的每个元素都需要重新编码标记号，因此重复字段尤其适合于此优化。

每个字段必须用以下修饰符之一进行修饰：

- `required`：必须提供值，否则消息将被视为“未初始化”。如果在调试模式下编译libprotobuf，则序列化未初始化的消息将导致断言失败。在优化的生成中，将跳过检查，并无论如何写入消息。但是，解析未初始化的消息总是失败（通过从parse方法返回false）。除此之外，required字段与optional完全相同。

- `optional`：字段值可以设置，也可以不设置。如果未设置可选字段值，则使用默认值。对于简单类型，您可以指定自己的默认值，正如示例中`optional PhoneType type = 2 [default = HOME];`。否则，将使用系统默认值：对于数值类型为零，对于字符串为空字符串，对于布尔值为假。对于嵌入的消息，默认值始终是消息的“默认实例”或“原型”，它没有设置任何字段。调用访问器以获取尚未显式设置的可选（或必需）字段的值，始终返回该字段的默认值。

- `repeated`：字段可以重复任意次数（包括零）。重复值的顺序将保留在protocol buffer中。将重复字段视为动态大小的数组。

> “`required` ”是永久性的，应该非常小心地根据需要标记字段。如果在某个时刻希望停止写入或发送一个`required `字段，将该字段更改为`optional `字段将是一个问题——旧的读者会认为没有该字段的消息是不完整的，并且可能会无意中拒绝或删除它们。应该考虑为缓冲区编写特定于应用程序的自定义验证例程。谷歌的一些工程师得出了这样的结论：按需使用弊大于利；他们宁愿只使用可选的和重复的。然而，这种观点并不是共识。


## 编译写好的Protocol Buffer文件
生成需要读写AddressBook （以及Person和PhoneNumber）消息的类。为此，需要在`.proto`上运行protocol buffer编译器protoc:
1. 如果尚未安装编译器，请下载该包并按照描述文件中的说明进行操作。

2. 现在运行编译器，指定源目录（应用程序源代码所在的目录——如果不提供值，则使用当前目录）、目标目录（希望生成的代码所在的目录，通常与`$SRC_DIR`相同）和`.proto`的路径。
```c
    protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
```
因为需要C++类，所以使用`--cpp_out`选项，他支持的语言也有类似的选项。

这将在指定的目标目录中生成以下文件：

- `addressbook.pb.h`，声明生成的类的头。

- `addressbook.pb.cc`，其中包含类的实现。

## Protocol Buffer API
在addressbook.pb.h中，可以看到在addressbook.proto中指定的每条消息都有一个类。在Person类，可以看到编译器已经为每个字段生成了访问器。例如，对于名称、ID、电子邮件和电话字段，有以下方法：

```c
 // name
  inline bool has_name() const;
  inline void clear_name();
  inline const ::std::string& name() const;
  inline void set_name(const ::std::string& value);
  inline void set_name(const char* value);
  inline ::std::string* mutable_name();

  // id
  inline bool has_id() const;
  inline void clear_id();
  inline int32_t id() const;
  inline void set_id(int32_t value);

  // email
  inline bool has_email() const;
  inline void clear_email();
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline ::std::string* mutable_email();

  // phones
  inline int phones_size() const;
  inline void clear_phones();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
  inline const ::tutorial::Person_PhoneNumber& phones(int index) const;
  inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  inline ::tutorial::Person_PhoneNumber* add_phones();
```

getter的名称与字段的大小写完全相同，setter方法以`set_`开头。对于每个单数（required 或 optional）字段，也有`has_`方法，如果设置了该字段，则返回“真”。最后，每个字段都有`clear_ `的方法，可以取消将字段设置回其空状态。

`id `字段只有上面描述的基本访问集，但是`name`和`email`字段有一些额外的方法，因为它们是字符串——`mutable_`的getter，可以让您直接获得指向字符串的指针，以及一个额外的setter。即使尚未设置email，也可以调用`mutable_email()`；它将自动初始化为空字符串。如果您在这个例子中有一个单一的message 字段，那么它也将有一个`mutable_`的方法，而不是`set_`方法。

`Repeated` 字段也有一些特殊的方法：

- 检查`Repeated`字段的`_size`（换句话说，与此人关联的电话号码有多少）。

- 使用其索引获取指定的电话号码。

- 在指定索引处更新现有电话号码。

- 将另一个电话号码添加到您可以编辑的message中（重复的标量类型有一个“`add_`”，只允许您传入新值） 。

### 枚举和嵌套类

生成的代码包含与`.proto`枚举对应的`PhoneType`枚举。您可以将此类型称为`Person::PhoneType`，其值为`Person::MOBILE`、`Person::HOME`和`Person::WORK`。

编译器还为您生成了一个名为`Person::PhoneNumber`的嵌套类。如果查看代码，可以看到“real”类实际上被称为`Person_PhoneNumber`，但是`Person `内部定义的`typedef`允许您将其视为嵌套类。唯一不同的是，如果您想在另一个文件中向前声明该类，则不能向前声明C++中的嵌套类型，而是可以向前声明`Person_PhoneNumber`。

### 标准消息方法

每个message 类还包含许多其他方法，这些方法允许检查或操作整个message ，包括：

- `bool IsInitialized() const;`：检查是否已设置所有必需字段。

- `string DebugString() const;`：返回消息的可读表示形式，对于调试特别有用。

- `void CopyFrom(const Person& from);`：用给定消息的值覆盖消息。

- `void Clear();`：将所有元素清除回空状态。

### 解析和序列化

每个 protocol buffer 类都有使用 protocol buffer 二进制格式写入和读取所选类型的消息的方法。其中包括：



- `bool SerializeToString(string* output) const;`：序列化消息并将字节存储在给定的字符串中。注意，字节是二进制的，而不是文本；我们只使用字符串类作为方便的容器。

- `bool ParseFromString(const string& data);`：解析给定字符串中的消息。

- `bool SerializeToOstream(ostream* output) const;`：将消息写入给定的C++`ostream`。

- `bool ParseFromIstream(istream* input);`；从给定的C++ `istream`解析消息。

## 写 Message

创建和填充protocol buffer类的实例，然后将它们写入输出流。

这里有一个程序，它从一个文件中读取一个`AddressBook `，根据用户输入向其中添加一个新的人，然后将新的地址簿重新写回到文件中。

```c
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
  cout << "Enter person ID number: ";
  int id;
  cin >> id;
  person->set_id(id);
  cin.ignore(256, '\n');

  cout << "Enter name: ";
  getline(cin, *person->mutable_name());

  cout << "Enter email address (blank for none): ";
  string email;
  getline(cin, email);
  if (!email.empty()) {
    person->set_email(email);
  }

  while (true) {
    cout << "Enter a phone number (or leave blank to finish): ";
    string number;
    getline(cin, number);
    if (number.empty()) {
      break;
    }

    tutorial::Person::PhoneNumber* phone_number = person->add_phones();
    phone_number->set_number(number);

    cout << "Is this a mobile, home, or work phone? ";
    string type;
    getline(cin, type);
    if (type == "mobile") {
      phone_number->set_type(tutorial::Person::MOBILE);
    } else if (type == "home") {
      phone_number->set_type(tutorial::Person::HOME);
    } else if (type == "work") {
      phone_number->set_type(tutorial::Person::WORK);
    } else {
      cout << "Unknown phone type.  Using default." << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!input) {
      cout << argv[1] << ": File not found.  Creating a new file." << endl;
    } else if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  // Add an address.
  PromptForAddress(address_book.add_people());

  {
    // Write the new address book back to disk.
    fstream output(argv[1], ios::out | ios::trunc | ios::binary);
    if (!address_book.SerializeToOstream(&output)) {
      cerr << "Failed to write address book." << endl;
      return -1;
    }
  }

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

**注意**`GOOGLE_PROTOBUF_VERIFY_VERSION` 宏。在使用C++`Protocol Buffer `库之前，执行这个宏是很好的实践（不是严格必要的）。它验证该版本与编译时使用的头的版本是否兼容。如果检测到版本不匹配，程序将中止。注意，每个`.pb.cc`文件在启动时都会自动调用这个宏。



**注意**在程序结束时调用`ShutdownProtobufLibrary() `。删除协议缓冲区库分配的所有全局对象。对于大多数程序来说，这是不必要的，因为进程无论如何都将退出，操作系统将负责回收其所有内存。但是，如果使用要求释放每个对象的内存泄漏检查器，或者编写的库可能由单个进程多次加载和卸载，则可能需要强制协议缓冲区清除所有内容。


## 读取Message

此示例读取由上述示例创建的文件并打印其中的所有信息。

```c
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// Iterates though all people in the AddressBook and prints info about them.
void ListPeople(const tutorial::AddressBook& address_book) {
  for (int i = 0; i < address_book.people_size(); i++) {
    const tutorial::Person& person = address_book.people(i);

    cout << "Person ID: " << person.id() << endl;
    cout << "  Name: " << person.name() << endl;
    if (person.has_email()) {
      cout << "  E-mail address: " << person.email() << endl;
    }

    for (int j = 0; j < person.phones_size(); j++) {
      const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

      switch (phone_number.type()) {
        case tutorial::Person::MOBILE:
          cout << "  Mobile phone #: ";
          break;
        case tutorial::Person::HOME:
          cout << "  Home phone #: ";
          break;
        case tutorial::Person::WORK:
          cout << "  Work phone #: ";
          break;
      }
      cout << phone_number.number() << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file and prints all
//   the information inside.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  ListPeople(address_book);

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```
## 扩展Protocol Buffer

如果希望新的Protocol Buffer向后兼容，而旧的缓冲区向前兼容，需要遵循一些规则。在新版本的协议缓冲区中：


- 不能更改任何现有字段的标记号。

- 不能添加或删除任何必需字段。

- 您可以删除可选或重复的字段。

- 您可以添加新的可选或重复字段，但必须使用新的标记号（即从未在此协议缓冲区中使用过的标记号，即使是已删除的字段）。

> 这些规则有一些例外，但很少使用。


如果遵循这些规则，旧代码将很高兴地读取新消息，并且只忽略任何新字段。对于旧代码，删除的`optional` 字段具有其默认值，删除的重复字段将为空。新代码还将透明地读取旧消息。

但是，请记住，新的可选字段不会出现在旧消息中，因此需要检查它们是用`has_`设置的，或者在.`proto`文件中在标记号后用`[default = value]`提供一个合理的默认值。

如果没有为可选元素指定默认值，则使用特定于类型的默认值：对于字符串，默认值为空字符串。对于布尔值，默认值为false。对于数字类型，默认值为零。

还要注意，如果添加了一个新的重复字段，那么您的新代码将无法分辨它是保留为空（由新代码）还是从未设置过（由旧代码），因为它没有`has_`标志。




## 优化提示

C++Protocol Buffer库已经被极大地优化。但是，正确的使用可以进一步提高性能。下面是一些提示：

- 尽可能重用message 对象。message 试图保留它们分配给重用的任何内存，即使它们被清除。因此，如果您连续处理许多具有相同类型和类似结构的消息，那么最好每次都重用相同的消息对象，以减轻内存分配器的负载。但是，随着时间的推移，对象可能会变得膨胀，特别是如果消息在“形状”上有所不同，或者偶尔构建的消息比平常大得多。您应该通过调用`SpaceUsed `方法来监视消息对象的大小，并在它们变得太大时将其删除。

- 对于从多个线程分配大量小对象，系统的内存分配器可能没有得到很好的优化。尝试使用` Google's tcmalloc `。

## 高级用法
Protocol buffers提供的一个关键特性是反射。您可以迭代消息的字段并操作它们的值，而无需针对任何特定的消息类型编写代码。使用反射的一个非常有用的方法是将protocol message与其他编码（如XML或JSON）进行相互转换。反射的一个更高级的用途可能是发现同一类型的两个消息之间的差异，或者开发一种“protocol message的正则表达式”，在这种表达式中可以编写与特定message内容匹配的表达式。

反射由` Message::Reflection interface`提供。
