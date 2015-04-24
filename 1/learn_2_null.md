# null的使用，NullPointException的预防，Guava Option<>的使用 #
----------

# 一、null的含糊语义 #

Null的含糊语义让人很不舒服。Null很少可以明确地表示某种语义，例如，Map.get(key)返回Null时，可能表示map中的值是null，亦或map中没有key对应的值。Null可以表示失败、成功或几乎任何情况。使用Null以外的特定值，会让你的逻辑描述变得更清晰。

	Map<String, String> map = new HashMap<String, String>();
	map.put("loull", null);
	System.out.println(map.get("loull"));
	System.out.println(map.get("alipay"));

两个都输出`null`，我们无法得知是不存在这个`key`，还是这个`key`对应的`value`值是`null`。

----------

# 二、null对象的使用 #

Null确实也有合适和正确的使用场景，如在性能和速度方面Null是廉价的，而且在对象数组中，出现Null也是无法避免的。但相对于底层库来说，在应用级别的代码中，Null往往是导致混乱，疑难问题和模糊语义的元凶。

### (1).常见使用场景： ###

有时候，我们定义一个引用类型变量，在刚开始的时候，无法给出一个确定的值，但是不指定值，程序可能会在try语句块中初始化值。这时候，我们下面使用变量的时候就会报错。这时候，可以先给变量指定一个null值，问题就解决了。例如：  

	Connection conn = null;
	try {
	　　conn = DriverManager.getConnection("url", "user", "password");
	} catch (SQLException e) {
	　　 e.printStackTrace();
	}
	String catalog = conn.getCatalog();

如果刚开始的时候不指定conn = null，则最后一句就会报错。

### (2).容器类型与null： ###

- List：允许重复元素，可以加入任意多个null。
- Set：不允许重复元素，最多可以加入一个null。
- Map：Map的key最多可以加入一个null，value字段没有限制。
-  数组：基本类型数组，定义后，如果不给定初始值，则java运行时会自动给定值。引用类型数组，不给定初始值，则所有的元素值为null。

### (3).null的其他作用: ###

1. 判断一个引用类型数据是否null。 用==来判断。
2. 释放内存，让一个非null的引用类型变量指向null。这样这个对象就不再被任何对象应用了。等待JVM垃圾回收机制去回收。

### (4).null的使用建议： ###

1. 在Set或者Map中使用null作为键值指向的value，千万别这么用。很显然，在Set和Map的查询操作中，将null作为特殊的例子可以使查询结果更浅显易懂。
2. 在Map中包含value是null值的键值对，应该**把这种键值对移出map**，使用一个独立的Set来包含所有null或者非null的键。最好的办法就是把这类key值分立开来，并且好好想想一个value是null的键值对对于程序来说到底意味着什么。
3. 在列表中使用null，并且这个列表的数据是稀疏的，或许最好应该使用一个Map<Integer,E>字典来代替这个列表。因为字典更高效，并且也许更加精确的符合你潜意识里对程序的需求。
4. 想象一下如果有一种自然的“空对象”可以使用，比方说对于枚举类型，添加一个枚举常数实例，这个实例用来表示你想用null值所表示的情形。比如：Java.math.RoundingMode有一个常数实例UNNECESSARY来表示“不需要四舍五入”，任何精度计算的方法若传以RoundingMode.UNNECESSARY为参数来计算，必然抛出一个异常来表示不需要舍取精度。

### (5).问题和困惑: ###

首先，对于null的随意使用会一系列难以预料的问题。大概95%以上的集合类默认并不接受null值，如果有null值将被放入集合中。

另外，null值是一种令人不满的模糊含义。有的时候会产生二义性，这时候我们就很难搞清楚具体的意思，如果程序返回一个null值，其代表的含义到底是什么。用其它一些值(而不是null值)可以让代码表述的含义更清晰。

----------

# 三、NullPointerException原因 #

**对象未初始化而直接引用对象值或者方法到的**

**对象引用已经不存在或者被JDBC关闭**


- 一个经典的例子是JDBC connection已经关闭，ResultSet对象仍然被使用中，这个时候NullPointerException就被抛出。

**违反某些Java容器的限制，读写Null值**

- 这个方面首推的就java.util.HashTable,它不接受Null 作为Key或者Value，如果试图用Null作为Key去读取HashTable将会得到NullPointerException

**几种相对比较难的Java NullPointerException异常**

1. Socket连接丢失导致IO流的Java NullPointerException


2. 资源文件加载错误导致的NullPointerException


		InputStream in =this.getClass().getResourceAsStream(PropertiesName); 
		props.load(in);  // throw NullPointerException if xml/property files missing


3. 多线程导致的NullPointerException


----------


# 四、避免NullPointerException的一些实践 #

### (1)从已知的String对象中调用equals()方法，而非未知对象 ###

	Object unknownObject = null;
	 
	//错误方式 – 可能导致 NullPointerException
	if(unknownObject.equals("knownObject")){
	   System.err.println("This may result in NullPointerException if unknownObject is null");
	}
	 
	//正确方式 - 即便 unknownObject是null也能避免NullPointerException
	if("knownObject".equals(unknownObject)){
	    System.err.println("better coding avoided NullPointerException");
	}

### (2)当valueOf()和toString()返回相同的结果时，宁愿使用前者 ###

	BigDecimal bd = getPrice();
	System.out.println(String.valueOf(bd)); //不会抛出空指针异常
	System.out.println(bd.toString()); //抛出 "Exception in thread "main" java.lang.NullPointerException"

### (3)使用null安全的方法和库 ###

有很多开源库已经为您做了繁重的空指针检查工作。其中最常用的一个的是Apache commons 中的`StringUtils`。支付宝内部使用`StringUtil`也是类似的。

	//StringUtils方法是空指针安全的，他们不会抛出空指针异常
	System.out.println(StringUtils.isEmpty(null));
	System.out.println(StringUtils.isBlank(null));
	System.out.println(StringUtils.isNumeric(null));
	System.out.println(StringUtils.isAllUpperCase(null));
	 
	Output:
	true
	true
	false
	false

### (4)避免从方法中返回空指针，而是返回空collection或者空数组。 ###

通过返回一个空collection或者空数组，在调用如size(),length()的时候不会因为空指针异常崩溃。Collections类提供了方便的空List，Set和Map: Collections.EMPTY_LIST，Collections.EMPTY_SET，Collections.EMPTY_MAP。

	public List getOrders(Customer customer){
	    List result = Collections.EMPTY_LIST;
	    return result;
	}

### (5)使用annotation@NotNull 和 @Nullable ###

通过使用像`@NotNull`和`@Nullable`之类的annotation来声明一个方法是否是空指针安全的。现代的编译器、IDE或者工具可以读此annotation并添加忘记的空指针检查，或者提示出不必要的乱七八糟的空指针检查。IntelliJ和findbugs已经支持了这些annotation。

*介个暂时我也不会用*

###  (6)遵从Contract并定义合理的默认值。 ###

在Java中避免空指针异常的一个最好的方法是简单的定义contract并遵从它们。

大部分空指针异常的出现是因为使用不完整的信息创建对象或者未提供所有的依赖项。

如果不允许创建不完整的对象并优雅地拒绝这些请求，可以在接下来的工作中预防大量的空指针异常。

如果对象允许创建，需要给他们定义一个合理的默认值。

例如一个Employee对象不能在创建的时候没有id和name，但是是否有电话号码是可选的。现在如果Employee没有电话号码，可以返回一个默认值（例如0）来代替返回null。但是必须谨慎选择，有时候检查空指针比调用无效号码要方便。同样的，通过定义什么可以是null，什么不能为null，调用者可以作出明智的决定。failing fast或接受null同样是一个你需要进行选择并贯彻的，重要的设计决策

### (7)定义数据库中的字段是否可为空 ###

如果在使用数据库来保存域名对象，如Customers，Orders 等，需要在数据库本身定义是否为空的约束。因为数据库会从很多代码中获取数据，数据库中有是否为空的检查可以确保数据健全。在数据空中维护null约束同样可以帮助你减少Java代码中的空指针检查。当从数据库中加载一个对象是明确哪些字段是可以为null的，而哪些不能，这可以使代码中不必要的!= null检查最少化。


更多参见：

[http://javarevisited.blogspot.sg/2013/05/ava-tips-and-best-practices-to-avoid-nullpointerexception-program-application.html](http://javarevisited.blogspot.sg/2013/05/ava-tips-and-best-practices-to-avoid-nullpointerexception-program-application.html "Java Tips and Best practices to avoid NullPointerException")

[http://javarevisited.blogspot.sg/2012/07/auto-boxing-and-unboxing-in-java-be.html](http://javarevisited.blogspot.sg/2012/07/auto-boxing-and-unboxing-in-java-be.html "What is autoboxing and unboxing in Java")

[http://www.importnew.com/7268.html](http://www.importnew.com/7268.html "避免Java应用中NullPointerException的技巧和最佳实践")


# 五、Guava的`Optional<T>`，使用和避免`null` #

Guava用Optional<T>表示可能为null的T类型引用。一个Optional实例可能包含非null的引用（我们称之为引用存在），也可能什么也不包括（称之为引用缺失）。它从不说包含的是null值，而是用存在或缺失来表示。但Optional从不会包含null值引用。
	
	Optional<Integer> possible = Optional.of(5);
	possible.isPresent(); // returns true
	possible.get(); // returns 5

(1). 普通青年的代码：

	public void 普通青年_sayHello(String name){
	    if(name==null){
	        name = "火星人";
	    }
	    System.out.println("普通青年说：Hello, "+name);
	}

(2). 文艺青年的代码：
	import com.google.common.base.Optional;
	 
	public void 文艺青年_sayHello(String name){
	    name = Optional.fromNullable(name).or("火星人");
	    System.out.println("文艺青年说：Hello, "+name);
	}

(3). 示例

	import com.google.common.base.Optional;

	public class GuavaTester {
	   public static void main(String args[]){
	      GuavaTester guavaTester = new GuavaTester();
	
	      Integer value1 =  null;
	      Integer value2 =  new Integer(10);
	      //Optional.fromNullable - allows passed parameter to be null.
	      Optional<Integer> a = Optional.fromNullable(value1);
	      //Optional.of - throws NullPointerException if passed parameter is null
	      Optional<Integer> b = Optional.of(value2);		
	
	      System.out.println(guavaTester.sum(a,b));
	   }
	
	   public Integer sum(Optional<Integer> a, Optional<Integer> b){
	      System.out.println("First parameter is present: " + a.isPresent());
	
	      System.out.println("Second parameter is present: " + b.isPresent());
	
	      //Optional.or - returns the value if present otherwise returns the default value passed.
	      Integer value1 = a.or(new Integer(0));	
	
	      //Optional.get - gets the value, value should be present
	      Integer value2 = b.get();
	
	      return value1 + value2;
	   }	
	}

输出：

	First parameter is present: false
	Second parameter is present: true
	10

(4). 另外一个例子：
	
	import com.google.common.base.Optional;
	
	public class OptionalDemo {
	    public static void main(String[] args) {
	        Optional<Student> possibleNull = Optional.of(null);
	        possibleNull.get();
	    }
	    public static class Student { }
	}

上面的程序，我们使用Optional.of(null)方法，这时候程序会第一时间抛出空指针异常，这可以帮助我们尽早发现问题。

(5). 再看一个例子：

	public class OptionalDemo {
	    public static void main(String[] args) {
	        Optional<Student> possibleNull = Optional.absent();
	        Student jim = possibleNull.get();
	    }
	    public static class Student { }
	}

运行上面的程序，发现出现了：Exception in thread "main" java.lang.IllegalStateException: Optional.get() cannot be called on an absent value。


这样使用也会有异常出来，那Optional到底有什么意义呢？

使用Optional除了赋予null语义，增加了可读性，最大的优点在于它是一种傻瓜式的防护。**Optional迫使你积极思考引用缺失的情况**，因为我们必须显式地从Optional获取引用。直接使用null很容易让人忘掉某些情形，尽管FindBugs可以帮助查找null相关的问题，但是我们还是认为它并不能准确地定位问题根源。

如同输入参数，方法的返回值也可能是null。和其他人一样，我们绝对很可能会忘记别人写的方法method(a,b)会返回一个null，就好像当我们实现method(a,b)时，也很可能忘记输入参数a可以为null。将方法的返回类型指定为Optional，也可以迫使调用者思考返回的引用缺失的情形。