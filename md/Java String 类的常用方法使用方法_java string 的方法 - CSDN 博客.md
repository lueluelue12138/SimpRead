> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weiwenzhe/article/details/109301879)

String 常用类使用方法
--------------

String 类在 java.lang 包中，java 使用 String 类创建一个字符串变量，字符串变量属于对象。String 类对象创建后不能修改，StringBuffer & StringBuilder 类。这时我们会问，为什么我们 String 变量赋值不就是改变了吗？其实不是的，赋值后将会生成新的对像来存放新的内容，原先的对象依旧在内存中，但是 s 不在指向它，那么这个对象就会成为垃圾内存，在某一个特定的时刻有 Java 虚拟机回收。包含在一对双引号之间。

#### 文章目录

*   [String 常用类使用方法](#String_0)
*   [String 字符串变量的创建：](#String_12)
*   [声明并初始化：](#_19)
*   *   [1、int length();](#1int_length_27)
    *   [2、char charAt(值);](#2char_charAt_40)
    *   [3、char toCharArray();](#3char__toCharArray_54)
    *   [4、 int indexOf("字符")](#4_int_indexOf_72)
    *   [5、toUpperCase()； toLowerCase()；字符串大小写的转换](#5toUpperCase__toLowerCase_97)
    *   [6、String[] split("字符")](#6String_split_109)
    *   [7、boolean equals(Object anObject)](#7boolean_equalsObject_anObject_119)
    *   [8、trim(); 去掉字符串左右空格　　replace(char oldChar,char newChar); 新字符替换旧字符，也可以达到去空格的效果一种。](#8trim_E3_80_80_E3_80_80replacechar_oldCharchar_newChar_132)
    *   [9、String substring(int beginIndex,int endIndex)　　截取字符串](#9String_substringint_beginIndexint_endIndex_E3_80_80_E3_80_80_148)
    *   [10、boolean equalsIgnoreCase(String) 忽略大小写的比较两个字符串的值是否一模一样，返回一个布尔值](#10boolean_equalsIgnoreCaseString__159)
    *   [11、boolean contains(String) 判断一个字符串里面是否包含指定的内容，返回一个布尔值](#11boolean_containsString__169)
    *   [12、boolean startsWith(String)　　测试此字符串是否以指定的前缀开始。返回一个布尔值](#12boolean_startsWithString_E3_80_80_E3_80_80_181)
    *   [13.boolean endsWith(String)　　测试此字符串是否以指定的后缀结束。返回一个布尔值](#13boolean_endsWithString_E3_80_80_E3_80_80_193)
    *   [14、上面提到了 replace 方法，接下继续补充一下 String replaceAll(String,String) 将某个内容全部替换成指定内容，　　 String repalceFirst(String,String) 将第一次出现的某个内容替换成指定的内容](#14replace__String_replaceAllStringString__E3_80_80_E3_80_80_String_repalceFirstStringString__206)
*   [总结](#_215)

String 字符串变量的创建：
----------------

声明： String 变量名;

```
String str;
```

声明并初始化：
-------

String 变量名 =“初始值”;

```
String str = "淡忘_Java博客";
```

### 1、int length();

语法：字符串变量名. length();  
返回值为 int 类型。得到一个字符串的字符个数（中、英、空格、转义字符皆为字符，计入长度）

代码如下（示例）：

```
String a="淡忘_Java博客";
        int l = a.length();
        System.out.println(l);
输出的结果为9
```

### 2、char charAt(值);

语法 ：字符串名. charAt(值);　　返回值为 char 类型。从字符串中取出指定位置的字符

代码如下（示例）：

```
String str="淡忘_Java";
        char c = str.charAt(3);
        System.out.println("指定字符为：" + c);
运行结果：指定字符为：J
```

### 3、char toCharArray();

语法 ：字符串名. toCharArray(); 返回值为 char 数组类型。将字符串变成一个字符数组

代码如下（示例）：

```
String str="淡忘了"；
char c[] = str.toCharArray(); 
for (int i = 0; i < c.length; i++)
System.out.println("转为数组输出:" + c[i]);

　运行结果：
	转为数组输出:淡
	转为数组输出:忘
	转为数组输出:了
```

### 4、 int indexOf(“字符”)

语法 ：字符串名. indexOf(“字符”)；字符串名. indexOf(“字符”, 值)；查找一个指定的字符串是否存在，返回的是字符串的位置，如果不存在，则返回 - 1 。  
　 in lastIndexOf(“字符”) 得到指定内容最后一次出现的下标  
代码如下（示例）：

```
String str = "I am a good student";
        int a = str.indexOf('a');//a = 2
        int b = str.indexOf("good");//b = 7
        int c = str.indexOf("w", 2);//c = -1
        int d = str.lastIndexOf("a");//d = 5
        int e = str.lastIndexOf("a",3);//e = 2
        System.out.println(a);
        System.out.println(b);
        System.out.println(c);
        System.out.println(d);
        System.out.println(e);

　运行结果：
	转为数组输出:2
	转为数组输出:7
	转为数组输出:-1
	转为数组输出:5
	转为数组输出:2
```

### 5、toUpperCase()； toLowerCase()；字符串大小写的转换

```
String str="hello world";
System.out.println("将字符串转大写为：" + str.toUpperCase());
System.out.println("将字符串转换成小写为：" + str.toUpperCase().toLowerCase());

运算结果：

将字符串转大写为：HELLO WORLD
将字符串转换成小写为：hello world
```

### 6、String[] split(“字符”)

根据给定的正则表达式的匹配来拆分此字符串。形成一个新的 String 数组。

```
String str = "boo:and:foo";
String[] arr1 = str.split(":");
String[] arr2 = str.split("o");
运行结果：
　　arr1　　//{ "boo", "and", "foo" }
　　arr2　　//{ "b", "", ":and:f" }
```

### 7、boolean equals(Object anObject)

语法 ：字符串变量名. equals(字符串变量名);　　返回值为布尔类型。所以这里用 if 演示。比较两个字符串是否相等，返回布尔值

```
String str = "hello";
        String str1 = "world";
        if (str.equals(str1)) {
            System.out.println("这俩字符串值相等");
        } else {
            System.out.println("这俩字符串值不相等");
        }
　　运行结果：
　　　　这俩字符串值不相等
```

### 8、trim(); 去掉字符串左右空格　　replace(char oldChar,char newChar); 新字符替换旧字符，也可以达到去空格的效果一种。

```
String str = "      淡忘_Java博客         ";  
System.out.println("去掉左右空格后:" + str.trim());
 
运行结果：
　　去掉左右空格后:淡忘_Java博客

第二种：
String str = "       淡忘_Java博客         ";  
System.out.println("去掉左右空格后:" + str.replace(" ","")); 

运行结果：
　　去掉左右空格后:淡忘_Java博客
```

### 9、String substring(int beginIndex,int endIndex)　　截取字符串

```
String str = "123淡忘_Java博客456";
        System.out.println("截取后的字符为：" + str.substring(0, 3));// 截取0-3个位置的内容   不含3
        System.out.println("截取后字符为：" + str.substring(2));// 从第3个位置开始截取    含2
        
 运行结果：
　　　截取后的字符为：123
　　　截取后字符为：3淡忘_Java博客456
```

### 10、boolean equalsIgnoreCase(String) 忽略大小写的比较两个字符串的值是否一模一样，返回一个布尔值

```
String str = "123淡忘_Java博客456";
        System.out.println("截取后的字符为：" + str.substring(0, 3));// 截取0-3个位置的内容   不含3
        System.out.println("截取后字符为：" + str.substring(2));// 从第3个位置开始截取    含2
        
 运行结果：
　　　截取后的字符为：123
　　　截取后字符为：3淡忘_Java博客456
```

### 11、boolean contains(String) 判断一个字符串里面是否包含指定的内容，返回一个布尔值

```
String str = "HELLO WORLd";
        String str1 = "WO";
        if (str.contains(str1)) {
            System.out.println("str内容中存在WO");
        } else {
            System.out.println("抱歉没找着");
        }
运行结果：
　　str内容中存在WO
```

### 12、boolean startsWith(String)　　测试此字符串是否以指定的前缀开始。返回一个布尔值

```
String str = "HELLO WORLd";
        String str1 = "HE";
        if (str.startsWith(str1)) {
            System.out.println("str内容中存在HE前缀开头");
        } else {
            System.out.println("抱歉没找着");
        }
运行结果：
　　str内容中存在HE前缀开头
```

### 13.boolean endsWith(String)　　测试此字符串是否以指定的后缀结束。返回一个布尔值

```
String str = "淡忘博客";
        String str1 = "博客";
        if (str.endsWith(str1)) {
            System.out.println("str内容中存在\'博客\'后缀结束");
        } else {
            System.out.println("抱歉没找着");
        }
 运行结果：
　　str内容中存在'博客'后缀结束
```

### 14、上面提到了 replace 方法，接下继续补充一下 String replaceAll(String,String) 将某个内容全部替换成指定内容，　　 String repalceFirst(String,String) 将第一次出现的某个内容替换成指定的内容

```
String str = "，，，，，，淡忘博客，，，，， ";
        System.out.println("改变后：" + str.replaceAll("，", "a"));
  运行结果：改变后：aaaaaa淡忘博客aaaaa 
  String str = "，，，，，，淡忘博客6不，，，，， ";
        System.out.println("改变后：" + str.replaceFirst("6不", "很6"));
   运行结果：改变后：，，，，，，淡忘博客很6，，，，，
```

总结
--

大神勿喷，有什么错的希望给我留言，我会改正