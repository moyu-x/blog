---
title: 编写一个代码生成器（1）
date: 2017-04-01 22:45:15
tags:
  - 代码生成器
categories:
  - 代码生成器
---

最近在写代码的时候使用了IDEA上面的几个代码生成器的插件，例如GsonFormat以及MyBatisCodeHelper，由此就想到了的自己如何实现一个代码生成器。使用代码生成器，可以有效的介绍在开发过程中编写重复代码的可能性，当然也可以保证代码的质量和代码注释的质量。

# 原理

在整个代码生成的过程中，个人认为最核心的地方就是解析格式的设计和文件的读写。

1. 对于数据的读写，可以从一个文件或者从控制台读入一个数据，并根据此来生成相关的类。
2. 对于数据格式的设计，这个是代码生成的设计的核心，如果设计的好的话，可以构建一个代码质量非常高的代码生成器，主要的选择可以使用JSON格式也可以自己设计一个

# 简单实现

这里实现一个读取三个数据，一个为类名，一个为属性类型，一个为属性名称，生成一个简单类的代码生成器，生成的文件当然是放在同级目录下。

这个简单的实现其实最主要的问题是对文件的写入，其实对getter和setter的编写来说应该是十分的简单的，整个文件生成的核心代码如下，不过这些代码有点丑陋，这些里面只是使用字符串拼接的方法实现了一个类的主体模板，没有其他的东西，只要花时间都能做成：

```java
    /**
     * 生成getter方法.
     *
     * @param attributeType 属性类型
     * @param attributeName 属性名
     * @return 生成好的getter方法
     */
    public static String generateGetter(String attributeType, String attributeName) {
        return "\tpublic " + attributeType + " get" + CodeGeneration.toUpperCaseFirstOne(attributeName) + "(){\r" +
                "\t\treturn " + attributeName + ";\r" +
                "\t}\r";
    }


    /**
     * 生成setter方法.
     *
     * @param attributeType 属性类型
     * @param attributeName 属性名
     * @return 生成好的setter方法 string
     */
    public static String generateSetter(String attributeType, String attributeName) {
        return "\tpublic void set" + CodeGeneration.toUpperCaseFirstOne(attributeName) + "(" + attributeType + " " + attributeName + ")\r" +
                "\t\tthis." + attributeName + " = " + attributeName + ";\r" +
                "\t}\r";
    }

    /**
     * 生成类的首部.
     *
     * @param className 类名
     * @return 类首部
     */
    public static String generateClassHeader(String className) {
        return "public class " + CodeGeneration.toUpperCaseFirstOne(className) + "{\r";
    }

    /**
     * 生成类的属性.
     *
     * @param attributeType 属性的类型
     * @param attributeName 属性的名称
     * @return 生成的属性
     */
    public static String generateClassAttribute(String attributeType, String attributeName) {
        return "\tprivate " + attributeType + " " + attributeName + ";\r";
    }

    /**
     * 生成的类的底部.
     *
     * @return 类底部
     */
    public static String generateClassFoot() {
        return "}";
    }
```

其实就是编写一个模板代码，然后就可以根据此来生成大量的代码了，真个完整的代码在Github上可以看到。