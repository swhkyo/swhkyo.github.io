---
layout: post
title:  "Java 参数传递"
date:   2017-01-24 10:06 +0800
categories: jekyll update
---

# Java 参数传递 #

前几天写代码的时候掉进了一个深坑，作为一个好学生，谨记了大学老师的教(hu)导(che)，和信奉了大学教材上的内(xia)容(shuo):

当一个变量被当做参数传递给一个方法/函数(Method/Function)时候，会分为两种情况:

1. 当参数是基本类型的时候，传递的方式是**值传递(Call by value)**
2. 当参数是一个对象的时候，传递的方式是**引用传递(Call by reference)**

>值传递和引用传递分别是什么？

你一定没有好好上课，请回去自己复习或者Google

好，我们先来说说这个结(shen)论(keng):

>Java中，并没有引用传递，**所有参数传递都是值传递**

# 证实 #

江湖上有很多关于值传递和引用传递的传闻，但是百闻不如一见，还是得亲自实现：

## 值传递(Call by value) ##

首先我们来看看Java的值传递:

```Java
void changeNumberCallByValue(int number) {
    number++;
}

void main() {
    int value = 5;
    changeNumberCallByValue(value);
    System.out.println("value = " + value);
}
```
以上代码的输出结果应该是**"value = 5"**，函数changeNumberCallByValue并不会改变传入的参数的值。

这个大家都懂，我就不说了...

## 所谓的引用传递(such call by reference) ##

我们再看一段代码:

```Java
class Number {
    public int value; // 只是demo，真的写代码你千万不要public...

    public Number(int value) {
    	this.value = value;
    }
}

void changeNumberCallByRef(Number number) {
    number.value++;
}

void main() {
    Number number = new Number(5);
    changeNumberCallByRef(number);
    System.out.println("value = " + number.value);
}
```

以上代码输出的结果是**"value = 6"**，函数changeNumberCallByRef改变了number？

>刚说完Java中只有值传递就啪啪啪打脸了吗？？？number被改变了不就正正说明这是值传递吗？

## 能不能让我说完再打？ ##

我想说的是number并没有被改变，改变的是number.value...

我们修改一下上面的代码

```Java
class Number {
    public int value; // 只是demo，真的写代码你千万不要public...

    public Number(int value) {
    	this.value = value;
    }
}

void changeNumberCallByRef(Number number) {
    Number temp = new Number(0);
    temp.value = number.value;
    temp.value++;
    number = temp;
}

void main() {
    Number number = new Number(5);
    changeNumberCallByRef(number);
    System.out.println("value = " + number.value);
}
```

这段代码输出的结果是**"value = 5"**

## Why? ##

其实Java中的引用传递，传递的是一个***引用的值***，所以其实是一个值传递...(为什么这么拗口...)。

其实正确的理解，Java中的引用传递，是传递对象的指针，而并非一个对象。

>你可以修改这个指针指向的内存区域的内容，但是不能让指针指向一块新的内存区域。

举个正确例子就是:

**我家(内存区域)电视(一个变量)坏了，我把我家的钥匙(指针)给修电视的师傅，修电视的师傅拿着钥匙(指针)到我家(指向的内存区域)修好了电视(修改了这块变量)，然后我回家就发现电视修好了**

那我们再举一个错误的例子:

**我家(内存区域)电视(一个变量)坏了，我把我家的钥匙(指针)给修电视的师傅，修电视的师傅拿着钥匙(指针)回到了师傅自己家(不是我家...)修了他家的电视(修改另外一块变量)，然后告诉我电视修好了，结果我回到家投诉了师傅什么都没做...**

我们来分析一下两段代码：

```Java
void changeNumberCallByRef(Number number) {
    number.value++; // 师傅通过number指针，打开我家的大门看到value电视开始修理...
}

void changeNumberCallByRef(Number number) {
    Number temp = new Number(0); // 师傅回到自己家(或者去了别人家？？？)
    temp.value = number.value;
    temp.value++; // 师傅开始修理别人家的电视...
    number = temp; // 师傅告诉我修好了...
}
```

# 结论 #

所以结论就是:

>Java中，并没有引用传递，**所有参数传递都是值传递**

下次能不能不打脸？