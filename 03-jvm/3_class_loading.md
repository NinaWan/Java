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

JVM会动态地加载、连接和初始化类和接口，所谓动态，是指在类/接口第一次被使用时才会去加载，而不是在JVM启动的初始就一次性加载。

### 1. 加载 Loading

> Loading is the process of finding the binary representation of a class or interface type with a particular name and
> creating a class or interface from that binary representation.

加载阶段，将类/接口对应的.class文件中的二进制字节流读入到JVM中，并在方法区中为要加载的类/接口创建对应的java.lang.Class对象。

### 2. 连接 Linking

> Linking is the process of taking a class or interface and combining it into the run-time state of the Java Virtual
> Machine so that it can be executed.
>
> Linking a class or interface involves verifying and preparing that class or interface, its direct superclass, its
> direct superinterfaces, and its element type (if it is an array type), if necessary. Linking also involves resolution
> of
> symbolic references in the class or interface, though not necessarily at the same time as the class or interface is
> verified and prepared.

#### 2.1 验证 Verification

验证阶段，确保读入的.class文件中的内容满足当前JVM的[约束要求][Constraints on Java Virtual Machine Code]。

#### 2.2 准备 Preparation

准备阶段，为要加载的类/接口中的静态变量在方法区中分配内存空间，且赋予默认值。可以在loading和初始化之间的任意时间执行。

#### 2.3 解析 Resolution

> Resolution is the process of dynamically determining one or more concrete values from a symbolic reference in the
> run-time constant pool. Initially, all symbolic references in the run-time constant pool are unresolved.

解析阶段，对运行时常量池中的符号引用进行解析。

### 3. 初始化 Initialization

> Initialization of a class or interface consists of executing the class or interface initialization method
> &lt;clinit&gt;.

初始化阶段，执行类/接口的初始化方法[&lt;clinit&gt;][Class Initialization Methods]。

&lt;clinit&gt;方法由编译器自动生成，其中主要包括类/接口中定义的对静态变量的赋值语句和静态代码块中的语句。

JVM负责保证类初始化过程的线程安全。

**触发类/接口初始化的情况：**

1. 调用new, getstatic, putstatic, 或者invokestatic中的任意一条Java虚拟机指令，都会触发对应的类/接口的初始化。

2. 对2 (REF_getStatic), 4 (REF_putStatic), 6 (REF_invokeStatic), 或者8 (REF_newInvokeSpecial)
   进行方法句柄解析所得到的java.lang.invoke.MethodHandle实例第一次被调用时。

3. 调用类库中的某些反射方法时会触发对应类/接口的初始化，例如Class.forname("...")方法或者newInstance()方法等。

4. 如果子类被初始化，则会触发父类的初始化。

5. 声明了non-abstract, non-static方法的接口，如果其直接或者间接实现类被初始化，则会触发该接口的初始化。

6. JVM启动的时候会创建和初始化initial类/接口，并调用其中的main方法。

**不会触发类/接口初始化的情况：**

1. 通过子类调用父类的静态变量，不会导致子类的初始化，但父类会初始化。
2. 通过定义数组来引用类，不会导致该类的初始化。
3. 调用类中定义的常量，不会导致该类的初始化。

**初始化顺序：**

1. 父类的静态变量和静态代码块，按照声明的顺序
2. 子类的静态变量和静态代码块，按照声明的顺序
3. 父类的成员变量和非静态代码块，按照声明顺序
4. 父类的构造函数
5. 子类的成员变量和非静态代码块，按照声明顺序
6. 子类的构造函数

### 4. 使用 Using

### 5. 卸载 Unloading

类的卸载是指回收该类的Class对象。

卸载需要满足的条件：

1. 该类所有的实例都已经被回收；
2. 该类没有在其他地方被引用；
3. 该类的类加载器已经被回收。

## 类加载器

### JVM提供的类加载器

#### BootstrapClassLoader 启动类加载器

最高一层的类加载器，由C++实现，属于JVM的一部分，负责加载JAVA_HOME/lib目录或者-Xbootclasspath参数指定的目录中的能被JVM识别的类。

#### ExtensionClassLoader 扩展类加载器

在sun.misc.Launcher$ExtClassLoader类中由Java实现，不属于JVM的一部分，负责加载JAVA_HOME/lib/ext目录或者java.ext.dirs系统变量指定的目录中的jar包和类。

#### AppClassLoader 应用程序类加载器

在sun.misc.Launcher$AppClassLoader类中由Java实现，不属于JVM的一部分，负责加载应用程序classpath路径下的jar包和类，可以通过调用ClassLoader.getSystemClassLoader()
方法获得。

### 用户自定义的类加载器

### 双亲委派模型 Parents Delegation Model

当一个子类加载器收到类加载请求时，首先会尝试把该请求委派给父类加载器，依次往上委派直到最高层的类加载器。如果父类加载器无法完成请求，则子类加载器才会尝试自己去处理。

双亲委派模型中，类加载器之间的父子关系是通过组合(composition)实现的，而不是通过继承。

# Refs

1. [Chapter 4. The class File Format](https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html)
2. [Chapter 5. Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-5.html)

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

[Constraints on Java Virtual Machine Code]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-4.html#jvms-4.9

[Class Initialization Methods]: https://docs.oracle.com/javase/specs/jvms/se19/html/jvms-2.html#jvms-2.9.2