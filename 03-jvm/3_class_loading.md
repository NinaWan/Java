# 类加载 Class Loading

## class文件

### The ClassFile Structure

```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

#### Items

**magic**

class文件的前4个字节为magic number，被Java虚拟机用来标识当前文件是否为有效的class文件。它的值为`0xCAFEBABE`。

**minor_version, major_version**

magic number之后的4个字节为class file的版本号，前两个字节为minor version, 后两个字节为major version。

高版本的JVM可以执行低版本编译器生成的class file，反之则不可以。

通过javap -v命令可以查看class文件的版本信息。

**constant_pool_count**

版本号之后的2个字节为constant_pool_count，表示constant_pool table中entry的个数加一。

**constant_pool[]**

constant_pool是一个由cp_info组成的table，用来记录class中的各种常量，e.g. 字符串常量、类和接口名称、字段名称等等，下标从1开始。

cp_info的结构如下:

```
cp_info {
    u1 tag;
    u1 info[];
}
```

其中，tag用来标识当前的cp_info属于哪种格式，即info中应该存储哪些信息。详见 [The Constant Pool]

**access_flags**

常量池信息之后的两个字节是access_flags，用来标识当前类/接口的访问权限和其他设置。
flag的详细说明见 [Table 4.1-B. Class access and property modifiers]

**this_class**

this_class记录一个类索引值，不能为0，必须指向constant_pool中的一个tag为 [CONSTANT_Class_info] 的entry。

**super_class**

`1. 如果当前class文件标识的是一个类：`

super_class记录一个父类索引值，可以为0，或者指向constant_pool中的一个tag为 [CONSTANT_Class_info] 的entry。

因为Java中没有多继承，每个类至多有一个直接父类，所以只有一个父类索引值。

* 当父类索引值为0时，表示当前类没有父类，则当前类只可能是Object类；
* 当父类索引值不为0时，则索引值指向其父类的相关信息。

`2. 如果当前class文件标识的是一个接口：`

super_class记录一个父类索引值，不可以为0，必须指向constant_pool中的一个tag为 [CONSTANT_Class_info]
的entry，且该entry标识的是Object类的相关信息。

**interfaces_count**

interfaces_count表示的是当前类/接口直接实现的接口的数量。

**interfaces[]**

interfaces[]是一个数组，记录当前类/接口直接实现的接口的索引值，
指向constant_pool中tag为 [CONSTANT_Class_info] 且表示接口的entry，按照代码中实现的接口从左到右的顺序。

**fields_count**

fields_count表示的是当前类/接口中定义的变量的个数。

**fields[]**

fields是一个由field_info构成的table，用来记录当前类/接口中定义的每个变量的详细信息，包括类变量和实例变量，不包括方法中定义的局部变量，也不包括在其父类/接口中定义的变量。

field_info的结构如下：

```
field_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

其中：

* `access_flags`用来标识当前变量的访问权限和其他设置，详见 [Table 4.5-A. Field access and property flags]

* `name_index`指向constant_pool中一个结构为 [CONSTANT_Utf8_info] 的常量，用来标识当前变量的名称。

* `descriptor_index`指向constant_pool中一个结构为 [CONSTANT_Utf8_info] 的常量，用来标识当前变量的描述符。

* `attributes_count`表示的是当前变量的其他属性的个数。

* `attributes[]`是一个由[attribute_info]构成的table，用来记录每个属性的详细信息。

**methods_count**

methods_count表示的是当前类/接口中定义的方法的个数。

**methods[]**

methods是一个由method_info构成的table，用来记录当前类/接口中定义的每个方法的详细信息，包括实例方法、类方法、[实例初始化方法][Instance Initialization Methods]
和[类/接口初始化][Class Initialization Methods]方法，不包括从父类/接口中继承来的方法。

method_info的结构如下：

```
method_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

其中：

* `access_flags`用来标识当前方法的访问权限和其他设置，详见 [Table 4.6-A. Method access and property flags]

* `name_index`指向constant_pool中一个结构为 [CONSTANT_Utf8_info] 的常量，用来标识当前方法的名称。

* `descriptor_index`指向constant_pool中一个结构为 [CONSTANT_Utf8_info] 的常量，用来标识当前方法的描述符。

* `attributes_count`表示的是当前方法的其他属性的个数。

* `attributes[]`是一个由[attribute_info]构成的table，用来记录每个属性的详细信息。

**attributes_count**

attributes_count表示的是当前类的属性的个数。

**attributes[]**

attributes是一个由attribute_info构成的table，用来记录当前类/接口的每个属性的详细信息。

attribute_info的结构如下：

```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

其中：

* `attribute_name_index`指向constant_pool中一个结构为[CONSTANT_Utf8_info]的常量，用来表示当前属性的名称。

* `attribute_length`表示随后的属性信息的长度，单位为byte，不包括attribute_name_index和attribute_length自身所占的6个字节。

* `info`是一个byte array，用来记录属性的详细信息。

## 类加载机制

## 类加载器

<!-------------------------- Links --------------------------->

[The Constant Pool]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.4

[Table 4.1-B. Class access and property modifiers]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.1-200-E.1

[CONSTANT_Class_info]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.4.1

[Table 4.5-A. Field access and property flags]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.5-200-A.1

[CONSTANT_Utf8_info]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.4.7

[attribute_info]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.7

[Instance Initialization Methods]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-2.html#jvms-2.9.1

[Class Initialization Methods]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-2.html#jvms-2.9.2

[Table 4.6-A. Method access and property flags]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.6-200-A.1