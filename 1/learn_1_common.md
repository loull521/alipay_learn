# Java 习惯用法 和 Google Guava工具使用#

----------

### 实现equals() ###

    class Cat {
		int id;
		String name;

		@Override
		public boolean equals(Object obj) {
			Cat cat = (Cat)obj;
			if (this.id == cat.id && this.name.equals(cat.name)) return true;
			return false;
		}
	}

这里有两个问题:

1. `Cat cat = (Cat)obj;`强制类型转换，如果 `obj` 不是不是`Cat`类型的，会报异常。
2. 没有对`null`进行检测，比如， `this.name` 可能为 `null`，所以调用 `equal()`方法可能会报 NPE。

比较合适的实现方式，可以借助Guava工具类：

	import com.google.common.base.Objects;
    class Person{
		private String firstname;
		private String lastname;
		private int zipCode;
		
		@Override
		public boolean equals(Object obj) {
			//如果是自己equals自己的话，这里就可以直接返回true，避免了后面可能的大量比较
			if(obj == this ) return true; 
			if (!(obj instanceof Person)) return false;
			Person p = (Person)obj;
			return Objects.equal(firstname, p.firstname)
					&& Objects.equal(lastname, p.lastname)
					&& Objects.equal(zipCode, p.zipCode);
		}
	}

`Objects.equal()`方法可以避免对null的判断，以避免NPE

    Objects.equal("a", "a"); // returns true
    Objects.equal(null, "a"); // returns false
    Objects.equal("a", null); // returns false
    Objects.equal(null, null); // returns true

### hashCode ###

接上面的例子，同样使用 `com.google.common.base.Objects`工具类：

	@Override
	public int hashCode() {
		return Objects.hashCode(firstname, lastname, zipCode);
	}

使用Objects.hashCode(field1, field2, …, fieldn)来代替手动计算散列值。

### toString ###

公司使用的是的`org.apache.commons.lang.builder.ToStringBuilder;`工具类

	@Override
    public String toString() {
        return ToStringBuilder.reflectionToString(this, ToStringStyle.SHORT_PREFIX_STYLE);
    }

### compare/compareTo ###

Guava提供了`ComparisonChain`，可以更优雅地实现比较。

	@Override
	public int compareTo(Person o) {
		return ComparisonChain.start().compare(lastname, o.lastname)
				.compare(firstname, o.firstname)
				.compare(zipCode, zipCode).result();
	}


这儿如果某个对象的`firstname`或`lastname`为`null`，会报出 NPE，如果每次都判断是否是`null`，显得很繁琐，如何避免 NPE 也是一个很大的问题。


### 使用StringBuilder或StringBuffer，构建长字符串 ###

### 使用Iterator.remove()，遍历删除  ###

否则可能回报 `ConcurrentModificationException` 异常

### 反转字符串 ###
	
	new StringBuilder(s).reverse().toString()


----------


# 工具类的使用 #

先写到这儿，懒得写了。常见用法有很多，IO、exception异常、log日志、并发等都有很多工具包和最佳实践。

常用的工具类或工具包：

	java.util.Arrays
	java.util.Collections
	Google Guaua
	Apache commons

还有公司内部使用的工具类。（欢迎补充）

