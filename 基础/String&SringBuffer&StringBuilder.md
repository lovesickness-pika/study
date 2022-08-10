### String & StringBuffer & StringBuilder



- String、StringBuffer、StringBuilder都是用来操作字符串的对象，但是在使用上有些区别

- String是不可变对象，因为String对象是用final修饰的，并且它的底层数据char数组是用final修饰的
- StringBuffer和StringBuilder是可变对象，前者线程安全，后者线程不安全





#### 面试题：大量字符串拼接

使用StringBuilder，更快，更省内存

循环拼接不能使用String，因为会创建多个StringBuilder和String对象

- 字面量拼接编译器会直接优化为拼接后的字符串并放入常量池



#### 编译器对String字符串连接做的优化

- 对于字符串，变量与字面量或变量与变量的字符串连接，编译器会将其优化为 new StringBuilder，使用StringBuilder的append进行拼接，再调用toString返回String对象；对于常量与常量的拼接会直接优化为连接之后的字符串