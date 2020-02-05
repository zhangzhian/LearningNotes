# Protobuf 教程：基于Java

参考官方[Protocol Buffer Basics: Java](https://developers.google.cn/protocol-buffers/docs/javatutorial)文档。

笔者在此之前有C++版本的详细描述（可添加微信公众号查看，见文末），Java版本与前者类似，暂仅以此文档描述Java版本，不进行详细展开。

本教程提供了Java程序员使用protocol buffer的基本介绍。通过创建一个简单的示例应用程序，展示了如何

- 在.proto文件中定义消息格式。
- 使用协议缓冲编译器。
- 使用Java协议缓冲区API来编写和读取消息。

这不是在Java中使用protocol buffer的全面指南。有关更多详细的参考信息，请参阅 [Protocol Buffer Language Guide](https://developers.google.cn/protocol-buffers/docs/proto)，  [Java API Reference](https://developers.google.cn/protocol-buffers/docs/reference/java/index.html)， [Java Generated Code Guide ](https://developers.google.cn/protocol-buffers/docs/reference/java-generated)和 [Encoding Reference](https://developers.google.cn/protocol-buffers/docs/encoding)。



## 简介

序列化和检索结构化数据，有以下几种方法：

- 使用Java序列化。这是默认方法，因为它已内置在Java语言中，但它存在许多众所周知的问题，如果用C ++或Python编写的应用程序需要与之共享数据，也无法很好地工作。
- 可以发明一种将数据项编码为字符串的临时方法，例如将4个整数编码为“ 12：3：-23：67”。尽管确实需要编写一次性编码和解析代码，但是这是一种简单且灵活的方法，而且解析带来的运行时成本很小。这对编码非常简单的数据最有效。
- 将数据序列化为XML。XML是一种可读的，并且存在用于多种语言的库，因此该方法可能很常用。如果要与其他应用程序/项目共享数据，这可能是一个不错的选择。但是，XML占用大量空间，对它进行编码/解码会给应用程序带来巨大的性能损失。而且，检索XML DOM树比通常检索类中的简单字段要复杂得多。

Protocol buffer是解决此问题的灵活，高效，自动化的解决方案。使用Protocol buffer，编写`.proto`要存储的数据结构的描述。由此，Protocol buffer编译器创建了一个类，该类以有效的二进制格式实现Protocol buffer数据的自动编码和解析。生成的类为构成Protocol buffer的字段提供获取器和设置器，并以协议为单位来读取和写入Protocol buffer。重要的是，协议缓冲区格式支持随时间扩展格式的想法，以使代码仍可以读取以旧格式编码的数据。



## 定义协议格式

示例代码包含在源代码包中的“ examples”目录下。 [在这里下载](https://developers.google.cn/protocol-buffers/docs/downloads.html)。

要创建地址簿应用程序，需要从`.proto`文件开始。`.proto`文件中的定义很简单：您为要序列化的每个数据结构添加一条*message*，然后为消息中的每个字段指定名称和类型。这是`.proto`定义的消息的文件`addressbook.proto`。

```c
syntax = "proto2";

package tutorial;

option java_package = "com.example.tutorial";
option java_outer_classname = "AddressBookProtos";

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

语法类似于C ++或Java。

该`.proto`文件以程序包声明开头，这有助于防止不同项目之间的命名冲突。在Java中，除非已明确指定`java_package`，否则包名称将用作Java包。即使您确实提供了`java_package`，也仍然应该定义一个普通值`package`，以避免协议缓冲区名称空间以及非Java语言中的名称冲突。

在包声明之后，有两个特定于Java的选项： `java_package`和`java_outer_classname`。 `java_package`指定生成的类的Java包名称。如果未明确指定，则仅与`package`声明中给出的包名称匹配，但是这些名称通常不是适当的Java包名称（因为它们通常不是以域名开头）。该`java_outer_classname`选项定义了类名称，该名称应包含此文件中的所有类。如果您没有`java_outer_classname`明确给出，将通过将文件名转换为驼峰大小写来生成。例如，默认情况下，`my_proto.proto`将使用`MyProto`作为外部类名称。

接下来是message定义。message是包含一组类型字段的汇总。许多标准的简单数据类型都可以作为字段类型，包括`bool`，`int32`，`float`，`double`，和`string`。还可以通过使用其他message作为字段类型来为message添加更多结构：在上述示例中，`Person`消息包含`PhoneNumber`消息，而`AddressBook`消息包含`Person`消息。甚至可以定义嵌套在其他消息中的消息类型：该`PhoneNumber`类型在内部定义`Person`。如果某个字段具有一个预定义的值列表之一，也可以定义`enum`类型：在这里您要指定电话号码可以是`MOBILE`，`HOME`或`WORK`。

每个元素上的“ = 1”，“ = 2”标记标识该字段在二进制编码中使用的唯一“标记”。标签编号1至15与较高的编号相比，编码所需的字节减少了一个字节，因此，为了进行优化，您可以决定将这些标签用于常用或重复的元素，而将标签16和更高的标签用于较少使用的可选元素。重复字段中的每个元素都需要重新编码标签号，因此重复字段是此优化的最佳候选者。

每个字段都必须使用以下修饰符之一进行注释（proto2特性）：

- `required`：必须提供该字段的值，否则该消息将被视为“未初始化”。尝试构建未初始化的消息将引发`RuntimeException`。解析未初始化的消息将引发`IOException`。除此之外，必填字段的行为与可选字段完全相同。
- `optional`：可能会或可能不会设置该字段。如果未设置可选字段值，则使用默认值。对于简单类型，可以指定自己的默认值，就像`type`在示例中为电话号码所做的那样。否则，将使用系统默认值：数字类型为零，字符串为空字符串，布尔值为false。对于嵌入式消息，默认值始终是消息的“默认实例”或“原型”，没有设置任何字段。调用访问器以获取未显式设置的可选（或必填）字段的值始终会返回该字段的默认值。
- `repeated`：该字段可以重复任意次（包括零次）。重复值的顺序将保留在协议缓冲区中。将重复字段视为动态大小的数组。

> **永远是必需的** 您将字段标记为时应格外小心`required`。如果希望停止写入或发送必填字段，则将字段更改为可选字段会很成问题：老读者会认为没有该字段的邮件不完整，可能会无意拒绝或丢弃它们。应该考虑为Protocol buffer编写特定于应用程序的自定义验证例程。Google的一些工程师得出的结论是，使用`required`弊大于利。他们更喜欢只使用`optional`和`repeated`。但是，这种观点并不普遍。

`.proto`可以在 [ “Protocol Buffer Language Guide”](https://developers.google.cn/protocol-buffers/docs/proto)（笔者的公众号和blog中有相应的中文翻译版）中找到有关编写文件的完整指南，包括所有可能的字段类型。但是，不要去寻找类似于类继承的工具：Protocol buffer不能做到这一点。

## 编译 Protocol Buffer

既然有了`.proto`，接下来需要做的是生成读取和写入`AddressBook`（因此`Person`和`PhoneNumber`）消息所需的类。要做到这一点，需要运行编译器`protoc`对`.proto`进行编译：

1. 如果尚未安装编译器，请[下载软件包](https://developers.google.cn/protocol-buffers/docs/downloads.html)并按照README中的说明进行操作。

2. 现在运行编译器

   ```
   protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   - ` $SRC_DIR`:  `.proto`所在的路径

   - `--java_out`: 编译结果输出的路径

   - `$SRC_DIR/addressbook.proto`: 完整路径，带文件名（`$SRC_DIR`可以不添加，直接写文件名）

这将`com/example/tutorial/AddressBookProtos.java`在指定的目标目录中生成。



## Protocol Buffer API

查看`AddressBookProtos.java`，可以看到它定义了一个名为的类`AddressBookProtos`，该类嵌套在其中，是在`addressbook.proto`中指定的每个message的类。每个类都有自己的`Builder`类，可用于创建该类的实例。可以在下面的**“Builders vs. Messages”**部分中找到有关构建者的更多信息。

Message和构建器都为消息的每个字段都自动生成了访问器方法。Message只有getters，而建造者既有getters又有setters。以下是`Person`该类的一些访问器（为简洁起见，省略了实现）：

```java
// required string name = 1;
public boolean hasName();
public String getName();

// required int32 id = 2;
public boolean hasId();
public int getId();

// optional string email = 3;
public boolean hasEmail();
public String getEmail();

// repeated .tutorial.Person.PhoneNumber phones = 4;
public List<PhoneNumber> getPhonesList();
public int getPhonesCount();
public PhoneNumber getPhones(int index);
```

同时，`Person.Builder`具有相同的getter和setter：

```java
// required string name = 1;
public boolean hasName();
public java.lang.String getName();
public Builder setName(String value);
public Builder clearName();

// required int32 id = 2;
public boolean hasId();
public int getId();
public Builder setId(int value);
public Builder clearId();

// optional string email = 3;
public boolean hasEmail();
public String getEmail();
public Builder setEmail(String value);
public Builder clearEmail();

// repeated .tutorial.Person.PhoneNumber phones = 4;
public List<PhoneNumber> getPhonesList();
public int getPhonesCount();
public PhoneNumber getPhones(int index);
public Builder setPhones(int index, PhoneNumber value);
public Builder addPhones(PhoneNumber value);
public Builder addAllPhones(Iterable<PhoneNumber> value);
public Builder clearPhones();
```

每个字段都有简单的JavaBeans风格的getter和setters。`has`方法，已设置该字段，则返回true。最后，每个字段都有一种`clear`将字段重置为空状态的方法。

重复字段具有一些额外的方法：一种`Count`方法（仅是列表大小的简写），通过索引获取或设置列表的特定元素的getter和setter `add`方法，将新元素附加到列表的`addAll`方法以及一个方法这会将充满元素的整个容器添加到列表中。

请注意，即使`.proto`文件使用带下划线的小写字母，访问器方法也使用驼峰式命名。此转换由协议缓冲区编译器自动完成，以使生成的类与标准Java样式约定匹配。应始终在`.proto`文件中的字段名称中使用带下划线的小写字母；这样可以确保所有生成的语言都具有良好的命名习惯。有关`.proto`样式的更多信息，请参见[样式指南](https://developers.google.cn/protocol-buffers/docs/style)。

有关协议编译器为任何特定字段定义确切生成哪些成员的更多信息，请参见[Java生成的代码参考](https://developers.google.cn/protocol-buffers/docs/reference/java-generated)。

## 枚举和嵌套类

生成的代码包括一个`PhoneType`嵌套在以下位置的Java 5枚举`Person`：

```java
public static enum PhoneType {
  MOBILE(0, 0),
  HOME(1, 1),
  WORK(2, 2),
  ;
  ...
}
```

## Builders vs. Messages

Protocol buffer编译器生成的message类都是*不可变* (immutable)的。一旦构造了message对象，就无法像Java一样对其进行修改`String`。要构造消息，必须首先构造一个构建器，将要设置的任何字段设置为所选值，然后调用该构建器的`build()`方法。

修改message的构建器的每个方法都会返回另一个构建器。返回的对象实际上是在其上调用方法的同一生成器。为了方便起见，将其返回，以便您可以在一行代码中将多个设置器链在一起。

这是一个如何创建实例的示例`Person`：

```java
Person john =
  Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .addPhones(
      Person.PhoneNumber.newBuilder()
        .setNumber("555-4321")
        .setType(Person.PhoneType.HOME))
    .build();
```

## 解析和序列化

最后，每个Protocol buffer类都有使用[二进制格式](https://developers.google.cn/protocol-buffers/docs/encoding)写入和读取所选类型的消息的方法。这些包括：

- `byte[] toByteArray();`：序列化消息并返回包含其原始字节的字节数组。
- `static Person parseFrom(byte[] data);`：解析给定字节数组中的消息。
- `void writeTo(OutputStream output);`：序列化消息并将其写入`OutputStream`。
- `static Person parseFrom(InputStream input);`：读取并解析来自的消息`InputStream`。

这些只是为解析和序列化提供的几个选项。同样，请参阅[`Message`API参考](https://developers.google.cn/protocol-buffers/docs/reference/java/com/google/protobuf/Message)以获取完整列表。

> **协议缓冲区和OO设计**：Protocol buffer类基本上是简单的数据持有者（如C中的structs）。他们没有在对象模型中成为完备的类。如果要向生成的类添加更丰富的行为，最好的方法是将生成的协议缓冲区类包装在特定于应用程序的类中。如果无法控制`.proto`文件的设计，则包装Protocol buffer也是一个好主意（例如，重用另一个项目中的文件）。在这种情况下，可以使用包装器类来设计一个更适合的独特环境的接口：隐藏一些数据和方法，公开便利函数等。**永远不应通过从它们继承来向生成的类添加行为**。这将破坏内部机制，无论如何都不是一个好的面向对象的实践。



## 写一条Message

将地址簿应用程序能够做的第一件事是将个人详细信息写入地址簿文件，先创建并填充Protocol buffer类的实例，然后将它们写入输出流。

从文件中读取一个`AddressBook`，然后根据用户输入向其中添加一个新`Person`文件，然后再次将新`AddressBook`文件写回到文件中。

```java
import com.example.tutorial.AddressBookProtos.AddressBook;
import com.example.tutorial.AddressBookProtos.Person;
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.InputStreamReader;
import java.io.IOException;
import java.io.PrintStream;

class AddPerson {
  // This function fills in a Person message based on user input.
  static Person PromptForAddress(BufferedReader stdin,
                                 PrintStream stdout) throws IOException {
    Person.Builder person = Person.newBuilder();

    stdout.print("Enter person ID: ");
    person.setId(Integer.valueOf(stdin.readLine()));

    stdout.print("Enter name: ");
    person.setName(stdin.readLine());

    stdout.print("Enter email address (blank for none): ");
    String email = stdin.readLine();
    if (email.length() > 0) {
      person.setEmail(email);
    }

    while (true) {
      stdout.print("Enter a phone number (or leave blank to finish): ");
      String number = stdin.readLine();
      if (number.length() == 0) {
        break;
      }

      Person.PhoneNumber.Builder phoneNumber =
        Person.PhoneNumber.newBuilder().setNumber(number);

      stdout.print("Is this a mobile, home, or work phone? ");
      String type = stdin.readLine();
      if (type.equals("mobile")) {
        phoneNumber.setType(Person.PhoneType.MOBILE);
      } else if (type.equals("home")) {
        phoneNumber.setType(Person.PhoneType.HOME);
      } else if (type.equals("work")) {
        phoneNumber.setType(Person.PhoneType.WORK);
      } else {
        stdout.println("Unknown phone type.  Using default.");
      }

      person.addPhones(phoneNumber);
    }

    return person.build();
  }

  // Main function:  Reads the entire address book from a file,
  //   adds one person based on user input, then writes it back out to the same
  //   file.
  public static void main(String[] args) throws Exception {
    if (args.length != 1) {
      System.err.println("Usage:  AddPerson ADDRESS_BOOK_FILE");
      System.exit(-1);
    }

    AddressBook.Builder addressBook = AddressBook.newBuilder();

    // Read the existing address book.
    try {
      addressBook.mergeFrom(new FileInputStream(args[0]));
    } catch (FileNotFoundException e) {
      System.out.println(args[0] + ": File not found.  Creating a new file.");
    }

    // Add an address.
    addressBook.addPerson(
      PromptForAddress(new BufferedReader(new InputStreamReader(System.in)),
                       System.out));

    // Write the new address book back to disk.
    FileOutputStream output = new FileOutputStream(args[0]);
    addressBook.build().writeTo(output);
    output.close();
  }
}
```

## 读一条Message

示例读取由以上示例创建的文件，并打印其中的所有信息。

```java
import com.example.tutorial.AddressBookProtos.AddressBook;
import com.example.tutorial.AddressBookProtos.Person;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.PrintStream;

class ListPeople {
  // Iterates though all people in the AddressBook and prints info about them.
  static void Print(AddressBook addressBook) {
    for (Person person: addressBook.getPeopleList()) {
      System.out.println("Person ID: " + person.getId());
      System.out.println("  Name: " + person.getName());
      if (person.hasEmail()) {
        System.out.println("  E-mail address: " + person.getEmail());
      }

      for (Person.PhoneNumber phoneNumber : person.getPhonesList()) {
        switch (phoneNumber.getType()) {
          case MOBILE:
            System.out.print("  Mobile phone #: ");
            break;
          case HOME:
            System.out.print("  Home phone #: ");
            break;
          case WORK:
            System.out.print("  Work phone #: ");
            break;
        }
        System.out.println(phoneNumber.getNumber());
      }
    }
  }

  // Main function:  Reads the entire address book from a file and prints all
  //   the information inside.
  public static void main(String[] args) throws Exception {
    if (args.length != 1) {
      System.err.println("Usage:  ListPeople ADDRESS_BOOK_FILE");
      System.exit(-1);
    }

    // Read the existing address book.
    AddressBook addressBook =
      AddressBook.parseFrom(new FileInputStream(args[0]));

    Print(addressBook);
  }
}
```



## 扩展Protocol buffer

若希望新的Protocol buffer向后兼容，而旧的缓冲区向后兼容，需要遵循一些规则。在新版本的协议缓冲区中：

- *不得*更改任何现有字段的标签号。
- *不得*添加或删除任何必填字段。
- *可以*删除可选或重复的字段。
- *可以*添加新的可选或重复字段，但必须使用新的标签号（即该Protocol buffer中从未使用过的标签号，甚至删除的字段也从未使用过）。

（这些规则有[一些例外](https://developers.google.cn/protocol-buffers/docs/proto.html#updating)，但很少使用。）

如果遵循这些规则，旧代码将兼容新message，而忽略任何message。对于旧代码，已删除的可选字段将仅具有其默认值，而删除的重复字段将为空。新代码还将透明地读取旧消息。但是，请记住，新的可选字段不会出现在旧消息中，因此需要明确检查它们是否设置为`has_`，或使用以下命令在`.proto`文件中提供合理的默认值：`[default = value]`标签编号之后。如果未为可选元素指定默认值，则使用特定于类型的默认值：对于字符串，默认值为空字符串。对于布尔值，默认值为false。对于数字类型，默认值为零。还要注意，如果添加了一个新的重复字段，则由于没有`has_`标记，因此新代码将无法分辨它是空的（由新代码）还是根本没有设置（由旧代码）。

## 高级用法

Protocol buffer的用途不只是简单的访问器和序列化。确保浏览[Java API参考，](https://developers.google.cn/protocol-buffers/docs/reference/java/index.html)以了解还可以使用它们做什么。

Protocol buffer类提供的一项关键功能是*反射*。您可以遍历消息的字段并操纵它们的值，而无需针对任何特定的消息类型编写代码。使用反射的一种非常有用的方法是将协议消息与其他编码（例如XML或JSON）相互转换。反射的一种更高级的用法可能是查找相同类型的两条消息之间的差异，或者开发一种“协议消息的正则表达式”，在其中可以编写与某些消息内容匹配的表达式。可以将协议缓冲区应用于比最初预期的范围更广的问题。

反射是[`Message`](https://developers.google.cn/protocol-buffers/docs/reference/java/com/google/protobuf/Message)和[`Message.Builder`](https://developers.google.cn/protocol-buffers/docs/reference/java/com/google/protobuf/Message.Builder)接口的一部分。



**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)

