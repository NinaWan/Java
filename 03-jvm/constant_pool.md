# 常量池

## 常量

被final修饰的变量，初次赋值之后就不能再改变。

## Java中常量池

## Class常量池 (静态常量池)

.class文件中的常量池，主要包含字面量 (literal) 和符号引用 (symbolic references)。

符号引用包括以下常量：

1. 类和接口的全限定名；
2. 字段的名称和描述符；
3. 方法的名称和描述符；

## 运行时常量池 Runtime Constant Pool

JVM在类加载过程中会把.class文件中的常量池载入到方法区中。运行时常量池具备动态性，可以在JVM运行期间动态添加新的常量，例如调用String.intern()
方法时会将对应的字符串常量添加到常量池中。

## 字符串常量池

当创建字符串常量时，JVM首先会检查字符串常量池，如果字符串已存在，则直接返回字符串常量池中的实例引用；如果不存在，则会实例化该字符串并放到字符串常量池里。