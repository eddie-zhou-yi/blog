# Java字符串常量池

### 引言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们知道在java中有八种基本类型(此处不考虑void)，分别是byte, char, chort, int, long, float, double, boolean。这8种基本类型都是存储在栈空间中，因此他们的存取速度都比存储在堆中的引用类型来的更快。而除了这8中基本类型之外，还有一个String类型是我们在java编程中经常使用的一个类，这个类的对象用来保存字符串类型的变量。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在使用String类型的时候我们常常是使用`String var = "test var"`的形式来声明一个String对象，但是我们知道，java是一门面向对象的语言，若要生成一个类型对应的对象，则需要对这个这个类型进行实例化。那么String类为什么可以不使用new 来实例化，而直接赋值一个字符串的字面量，就可以生成一个String类型的对象了，这里的原因是JDK在实现的时候针对String类型做了优化，为字符串创建了一个字符串常量池，并且在显示声明一个String类型的字面量时，JDK在加载类时会自动调用intern方法，将此字面量保持至字符串常量池中，并返回此字面量的引用地址。

​         

