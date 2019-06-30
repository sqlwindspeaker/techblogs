# API设计建议

## 两个警句

### 1. 让接口容易被正确使用，不易被误用

### 2. 任何接口，如果要求用户必须记得做某些事情，就是有不正确使用的倾向

**举例子**：

```java
public class Date {
    public Date(int month, int day, int year) {}
}
```

接口的问题：

用户需要记住构造函数的参数顺序，这很容易被一个不太仔细的开发者误用；

即使现在的IDE可以方便的提示参数，但你仍可能在一个没有IDE的情况下进行debug，比如线上的终端环境；

**好的范例**：使用builder模式

```java
public class Date {
    private Date(int month, int day, int year) {}
    
    public static class Builder {
        public Builder() {}
        public Builder setMonth(int month) {}
        public Builder setYear(int year) {}
        public Builder setDay(int day) {}
        public Date build() {}
    }
}
```



## 基本原则

参考：https://coolshell.cn/articles/18024.html

### 好API的4个特点

#### 最简 vs. 完备

极简：减少公有的class，以及每个class中公有的方法，这样的API便于使用和记忆。

完备有两方面，一方面是说应该有的接口都已经包含了，当然这会和极简冲突；另一方面是一个接口应该存在在合适的地方。比如一个方法被放在一个错误的类中，就可能被用户所找不到。

#### 语义清晰且简单

这常常是一个容易出问题的地方，要注意以下三点：

1. 使用简单，直观，容易理解的算法实现功能
2. 没有明确需求的时候，不要考虑过度通用化的实现方案
3. 没有明确性能问题的时候，不要过早的考虑优化方案

#### 符合直觉

尽管每个人的直觉是不同的，但是我们应该努力假设对方不熟悉我们的系统，因此检验标准就是:

1. <span style="color:red;font-weight:bold">经验不丰富的用户也不用阅读文档就能看懂API</span>
2. <span style="color:red;font-weight:bold">开发人员不用查看API文档就可以理解使用API的代码</span>

#### 一致性和容易记忆

API 命名的一致性对于容易记忆是很有意义的。

反例:

```java
array.length
string.length()
list.size()
```





















