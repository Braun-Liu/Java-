# 一. 引子

一般的类和方法，只能使用具体类型：要么是基本类型，要么是自定义类型。如果要编写可以应用于多种类型的代码，这种刻板的限制对代码的束缚就会很大。

多态算是一种泛化机制，但对代码的约束还是太强（要么继承父类，要么实现接口）。**有许多原因促成了泛型的出现，而最重要的一个原因，就是为了\**更安全友好的使用容器类\** : \**用来指定容器要持有什么类型的对象，而且由编译器来保证类型的正确性。\****

例如：在 Java 加入泛型特性前，ArrayList 只维护一个 Object 类型的数组：

```java
public class ArrayList{



   private Object[] elementData;



   ...



   public Object get(int i){ ... }



   public void add(Object o){ ... }



   ...



}
```

显然，这样的实现存在两个问题： 
　　 
\1. 当获取一个值时，必须进行强制类型转换：

```java
ArrayList list = new ArrayList();



...



String str = (String)list.get(0);
```

\2. 该 ArrayList 没有错误检查，即可以向数组中添加任何类型的对象：

```java
list.add(new Integer(1));
```

向上述的动态数组中添加一个整型对象，程序在编译时和运行时都不会出错。但当我们强制将 get 结果转型时，就会抛出 ClassCastException 异常，程序出错。对于容器类，它是可以存储任何对象的，但我们在使用时，一次只想也只应该向其中放入一种对象。基于这种需求，提出了 **类型参数化** 的概念，即**泛型**。　

- **引入泛型后的 ArrayList 源码**

```java
public class ArrayList<E> extends AbstractList<E>



                          implements List<E>, RandomAccess, Cloneable, java.io.Serializable{



		



		// ...



}
```

------

#  二、泛型基础

## 1. 概念

1、**术语：**适用于许多许多的类型;

2、**本质：**实现类型参数化的概念，使代码可以应用于多种类型;

3、**核心：**告诉编译器想使用什么类型（指定具体类型参数或对其进行限制），然后编译器帮你处理一切细节(类型安全检查等);

4、**泛型初衷：**

(1). **希望类或方法具备最广泛的表达能力，即通过解耦类或方法与所使用的类型之间的约束；**

(2). **对容器类而言，泛型在保证容器类可以存储任何类型对象的同时，又保证了容器类一旦声明自己将要保存的元素类型时，就不可再保存其他类型了**，例如：

```java
ArrayList<Fruit> list = new ArrayList<Fruit>();



 



fruits.add(new Fruit());       // OK



fruits.add(new Apple());       // OK



fruits.add(new Orange());      // OK



 



fruits.add(new Object());      // Error
```

上述代码表明了该容器只能保存 Fruit 类型的对象，由于 Apple 也是一种 Fruit，所以其也可以保存 Apple 类型 对象，但对于不属于 Fruit类型 的对象，编译器杜绝将其放入列表中。

简单地说：**泛型 = 编译时的类型检查 + 编译时的类型擦除(编译器插入 checkcast 等) + 运行时的自动类型转换。****所以，我们在理解和应用泛型时，一定要从 编译期 和 运行时 两个视角去分析。**

- 例1：

在 Java 1.7 以后，我们可以这样创建一个 ArrayList：

```java
ArrayList<String> list = new ArrayList<>();
```

 list 变量的类型就决定了它引用的动态数组所能存储的元素类型，即后者的类型参数可以从变量中推断出。

- 例2：

```java
public class TypeInference {



 



	public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {



		Set<E> result = new HashSet<E>(s1);



		result.addAll(s2);



		return result;



	}



 



	public static void main(String[] args) {



		Set<Integer> integers = new HashSet<Integer>();



		Set<Double> doubles = new HashSet<Double>();



		Set<Number> numbers = null;



 



		// 编译器的类型推断能力有限



		numbers = TypeInference.union(integers, doubles);           // Error



		numbers = TypeInference.<Number>union(integers, doubles);   // OK



	}



}
```

如例 2 所示，为 numbers 赋值时需要显式对类型参数实例化。

## 2. 定义与语法

- **泛型类（参数化类）**

```java
public class Holder<T>{}
```

- **泛型接口（参数化接口）**

```java
public interface Generator<T>{}
```

-  **泛型方法（参数化方法：所在类可以是泛型类，也可以不是；能够独立于类而产生变化；细粒度）**

```java
public <T> void fun(T x){}
```

## 3. 注意事项

### 1. 泛型与多态

只有当你希望使用的参数类型比某个具体类型（以及它的所有子类型）更加泛化时，也就是说，**当你的代码能够跨多个类工作时，使用泛型才有所帮助（比如开发中我们写的一个公共类，可以设置为泛型类）；否则，使用多态就可以满足要求。**

```java
public class HasF {



	public void f(){...}



}



 



// 以下两种实现方式所取得的效果是一样的



 



// 泛型实现



class Manipulator1<T extends HasF>{



	private T obj;



	public Manipulator1(T x){



		this.obj = x;



	}



	public void manipulate(){



		obj.f();



	}



}



 



// 多态实现



class Manipulator2{



	private HasF obj;



	public Manipulator2(HasF x){



		this.obj = x;



	}



	public void manipulate(){



		obj.f();



	}



}
```

### 2. 泛型类的识别（误区）

先看下面两段代码：

```java
// 第一段代码



public class Pair<T> {



    private T first;



    private T second;



 



    public Pair(T first, T second){



        this.first = first;



        this.second = second;



    }



    public void setFirst(T first){



        this.first = first;



    }



    public T getFirst(){



        return first;



    }



    public void setSecond(T second){



        this.second = second;



    }



    public T getSecond(){



        return second;



    }



}



 



// 第二段代码  时间间隔类



public class DateInterval extends Pair<Date> {     



   



    public DateInterval(Date first, Date second){



        super(first, second);



    }



 



    @Override



    public void setSecond(Date second) {



        super.setSecond(second);



    }



    @Override



    public Date getSecond(){



        return super.getSecond();



    }



}
```

由泛型类的定义可知，Pair<T> 是一个泛型类，因为在类名后面有类型参数；类 DateInterval 后面没有跟类型参数列表，因此该类就是一个 T 被替换为 Date 的实体类，其从 Pair<Date> 继承得到的方法列表，与泛型彻底无关。

### 3. 对泛型类 LinkedList<T> 的类型参数 T 实例化所得到的不同泛型类型的理解

下图中，LinkedList<String>、LinkedList<Point> 和 LinkedList<PolyLine> 是三种不同的类型，就像 Integer 和 String 一样，是两种互不相同的类型。但是，三者共享同一个 Class 对象。换句话说，三者在运行期的类型是一样的，但在编译期根据类型参数的不同成为截然不同的类型。下面代码可为例证。 

![img](Java%E6%B3%9B%E5%9E%8B.assets/20190119180518195.png)

```java
public class TestClassTypes {



  public static void main(String[] args) {



 



  LinkedList<String> proverbs = new LinkedList<>();



  LinkedList<Object> numbers = new LinkedList<>();



 



   System.out.println("numbers class name: " + numbers.getClass().getName());   // Output: java.util.LinkedList



   System.out.println("proverbs class name: " + proverbs.getClass().getName()); // Output: java.util.LinkedList



   System.out.println("Compare Class objects: " + numbers.getClass().equals(proverbs.getClass()));     // Output:true



 



 



   // 由于 LinkedList<String> 与 LinkedList<Object> 在编译期根本就是不同类型，所以下面代码编译不能通过：



   // 类似于：把 Integer 类型实例强制转型为 String实例 赋给 String引用



   proverbs = (LinkedList<String>)numbers;   



 



   // 每个类都是 Object 的子类



   Object obj = (Object)numbers;



   System.out.println("obj class name " + obj.getClass().getName());  // Output: java.util.LinkedList



 



   // 会有转型安全的异常



   proverbs = (LinkedList<String>)obj;



   System.out.println("obj in proverbs class name " + obj.getClass().getName()); // Output:java.util.LinkedList



  }



}
```

### 4. 泛型与 static

**在泛型类中，static 域或方法无法访问泛型类的类型参数(类型实例化所得的所有泛型类型共享同一个Class对象)；若静态方法需要使用泛型能力，就必须使其成为泛型方法（不与泛型类共享类型参数）。**

**在一个类中，static 域或方法都是该类的 Class 对象的成员，而我们知道泛型所创造出来的所有类型都共享一个 Calss对象**， 因此实质上不受泛型参数限制，所以如下代码根本不能通过编译：

```java
public class Test<T> {



 



	public static T one;           // 编译错误



	



	public static T show(T one){   // 编译错误



		return null;



	}



}
```

看下在 Eclipse 中的编译报错提示：

```java
// 不能将一个静态的引用指向一个非静态的类型



Cannot make a static reference to the non-static type T
```

![img](Java%E6%B3%9B%E5%9E%8B.assets/20190119181337328.png)

但是要注意区分下面的一种情况：

```java
public class Test<T> {



 



	// 这是正确的



	public static <T> T show(T one){



		return null;



	}



}
```

因为这是一个泛型方法，在泛型方法中使用的 类型参数T 是自己在方法中定义的 T，而不是泛型类中的 T。

### 5. 限制泛型可用的类型

**在定义泛型类时，预设可以使用任何的类型来实例化泛型类中的类型。**但是，**如果想要限制使用泛型的类型时，即要求只能使用某个特定类型或其子类型才能实例化该类型时，使用 extends 关键字指定这个类型必须是继承或者实现某个接口。****一般地，当没有指定泛型继承的类型或实现的接口时，\**默认等价于使用 T extends Object\**，因此，默认情形下任何类型都可以作为参数插入。**特别地，**为类型参数设定的第一个边界可以是类类型或接口类型，类型参数的第一个边界之后的任意额外边界都只能是接口类型，同时，一般将标记性接口放到靠后位置，这些类型参数之间有 & 相连接。**

```java
publc class MyClass<T extends Number & Serilizable>{



    ...



}
```

- **在调用泛型方法的时候，可以指定泛型，也可以不指定泛型**

**在不指定泛型的情况下，泛型变量的类型为该方法中的几种类型的同一个父类的最小级，直到 Object。在指定泛型的时候，该方法中的几种类型必须是该泛型实例类型或者其子类。**

```java
// 代码示例



public class Test2{  



	public static void main(String[] args) {  



 



		/**不指定泛型的时候*/ 



		//这两个参数都是Integer，所以T为Integer类型



		Integer i = Test2.add(1, 2);   



		// 这两个参数一个是Integer，一个是Double，所以取同一父类的最小级，为Number  



		Number f = Test2.add(1, 1.2);  



		// 这两个参数一个是Integer，一个是String，所以取同一父类的最小级，为Object 



		Object o = Test2.add(1, "asd");   



 



		System.out.println(i.getClass().getName());   // 输出： java.lang.Integer



		System.out.println(f.getClass().getName());   // 输出： java.lang.Double



		System.out.println(o.getClass().getName());   // 输出： java.lang.String



 



		/**指定泛型的时候*/ 



		// 指定了Integer，所以只能为Integer类型或者其子类



		int a = Test2.<Integer>add(1, 2);   



		// 编译错误，指定了Integer，不能为Double



		int b = Test2.<Integer>add(1, 2.2); 



		// 指定为Number，所以可以为Integer和Double



		Number c = Test2.<Number>add(1, 2.2);    



	}  



 



	// 这是一个简单的泛型方法  



	public static <T> T add(T x, T y){  



		return y;  



	}  



} 
```

注意，这个例子中的两个输出是 java.lang.Double 和 java.lang.String，而不是 java.lang.Number 和 java.lang.Object：

```java
System.out.println(f.getClass().getName());   // 输出： java.lang.Double



System.out.println(o.getClass().getName());   // 输出： java.lang.String
```

实际上，这个问题涉及泛型机制和多态两点。在例子中，类型参数 T 被编译器用 Number 替换，这是没问题的，因为无论整形还是浮点型都属于数型，这是由多态机制保证的。但是，无论 x 还是 y，它们本质上还是各自的类型不会发生任何改变。要注意的是，**这里的 getClass() 方法返回的变量的实际类型，即运行时类型而非编译时类型，因此返回 y 的类型是double 而非 number。**

### 6. 泛型的兼容性

- **从泛型类型生成的任何类型的引用都能存储到对应的原生类型的变量中**

```java
LinkedList list = new LinkedList<String>();
```

这样编写代码是合法兼容的，但是，不应该将这作为日常编程习惯的一部分，因为这种实践存在固有的风险：由于 **类型安全性检查是针对引用的**，所以上述写法和如下写法实质上是一样的：

```java
LinkedList list = new LinkedList();
```

- **从原生类型生成的引用能存储到任何类型的泛型类型的变量中**

```java
LinkedList<String> list1 = new LinkedList();



LinkedList<Integer> list2 = new LinkedList();
```

这样编写代码是合法兼容的，但是，由于我们可以将一个已经原生的 LinkedList 对象 直接赋值此类引用，虽然在之后在添加元素是会进行类型安全检查，但之前的 LinkedList 对象 所存储的元素可能五花八门，给程序带来隐患。具体请参照下图：

![img](Java%E6%B3%9B%E5%9E%8B.assets/20190119183935622.png)

### 7. primitive 类型不可以作为类型参数（八大类型）

### 8. 若使用泛型方法可以取代将整个类泛型化，那么就应该使用泛型方法

### 9. 泛型方法与可变参数列表可以很好的共存

```java
public static <T> void f(T... args){}
```

------

# 三. 通配符及泛型的逆变和协变

## 1、 通配符

### (1).无界通配符

我们知道，通过为泛型类的每个类型形参提供类型实参，可以表达由这个泛型类定义的集合中的特定类型。例如，**为了指定存储 String 的 ArrayList，就需要将类型参数设定为 String，所以动态数组类型就是 ArrayList<String>。若不想为泛型类的类型参数提供具体类型，可以将参数设定为 “?”，这就是通配符的作用，通配符类型可以表示任何类或接口类型。**

```java
ArrayList<?> list  = new ArrayList<String>();



list  = new ArrayList<Double>();



 



list.add(e);     // e cannot be resolved to a variable  



System.out.println(list1.size());  
```

**list 变量是 ArrayList<?> 类型，所以能将指向任意类型的 ArrayList<> 对象的引用存储在其中。**但由于 list 是通配符类型参数的结果，所以存储引用的实际类型并不知道，因而无法使用这个变量调用任何与类型参数有关的方法。**特别地，在 Java 集合框架中，对于参数值是未知类型的容器类，只能读取其中元素，不能向其中添加元素。因为其类型未知，所以编译器无法识别添加元素的类型和容器的类型是否兼容，唯一的例外是 null (对 null 而言，无所谓类型)。**

### (2). 深入理解无界通配符

　　我们有必要对以下三种类型进行区分：

> List ：持有任何 Object 类型 的 原生 List，编译器不会对原生类型进行安全检查；
>
> List<?> ：具有某种特定类型 的 非原生List，编译器会进行安全检查；
>
> List<Object> ： 编译器认为 List<Object> 是 List<?> 的子类型；

```java
public class Wildcards {



    // Raw argument:



    static void rawArgs(Holder holder, Object arg) {



         holder.set(arg);  // Warning:



         holder.set(new Wildcards()); // Same warning



 



        // OK, but type information has been lost:



        Object obj = holder.get();



    }



 



    // Similar to rawArgs(), but errors instead of warnings:



    static void unboundedArg(Holder<?> holder, Object arg) {



        // holder.set(arg); // Error:



        // holder.set(new Wildcards()); // Same error



 



        // OK, but type information has been lost:



        Object obj = holder.get();



    }



}
```

## 2、 向上转型 / 通配符的上界 / 协变

在引入通配符的上界这一概念时，我们先看一下数组的一种特殊行为：基类型的数组引用可以被赋予导出类型的数组,如下面的代码所示：

```java
class Fruit {} 



class Apple extends Fruit {} 



class Jonathan extends Apple {} 



class Orange extends Fruit {} 



 



public class CovariantArrays {



	public static void main(String[] args) {



 



		Fruit[] fruit = new Apple[10];



		// 编译期、运行期都 OK



		fruit[0] = new Apple();  



		// 编译期、运行期都 OK



		fruit[1] = new Jonathan();



		//  编译期 OK、运行期抛出 java.lang.ArrayStoreException(因为 fruit 的运行时类型是 Apple[], 而不是 Fruit[] 或 Orange[]) 



		fruit[3] = new Fruit();  



		// 说明 Fruit[] 是 Apple[] 的父类型



		System.out.println(Fruit[].class.isAssignableFrom(Apple[].class));   // true 



	}



}
```

由此可以说明：

由 13 行可知，该行代码编译期正常，则进一步说明：编译器的类型检查是针对引用的(Fruit 型数组可以放入 Fruit 及其子类型对象)；但在运行时，由于 fruit 引用实际上指的是一个 Apple 数组，而作为 Apple 数组则只可以向其中放入 Apple 及其子类型对象，因此当放入 Fruit 对象时，抛出异常。

由 16 行可知，Fruit[ ] 是 Apple[ ] 的父类型，因此根据 Java 多态特性，前者可以指向后者对象。

我们知道，**泛型的主要目标之一就是将这种错误检查移到编译期**，那么，如果我们用泛型容器代替数组，那将会发生什么呢？

```java
public class NonCovariantGenerics { 



	



	// Compile Error: Type Mismatch 类型不匹配



	List<Fruit> flist = new ArrayList<Apple>();  



}
```

由以上代码可以知道，编译期根本不允许我们这么做。试想，如果编译期允许我们这样做，该容器就允许存入任何类型的对象，只要它是一种 Fruit，而不像数组那样会抛出运行时异常，违背了泛型的初衷（泛型保证容器的类型安全检查）。**所以，在编译期看来，List<Fruit> 和 List<Apple> 根本就是两种不同的类型，并无任何继承关系。** 

但是，有时你**想要在以上两个类型之间建立某种向上转型关系，这就引出了通配符的上界**。例如：

```java
public class GenericsAndCovariance {



	public static void main(String[] args) {



 



		// 允许我们向上转型，像数组那样



		List<? extends Fruit> flist = Arrays.asList(new Apple());



 



		// Compile Error: can’t add any type of object:



		flist.add(new Apple());      // Compile Error



		flist.add(new Fruit());      // Compile Error



		flist.add(new Object());     // Compile Error



 



		flist.add(null); // Legal but uninteresting



 



		// We know that it returns at least Fruit:



		Fruit f = flist.get(0);



		Object o = flist.get(0);



		Apple a = flist.get(0);   // Compile Error:Type mismatch



 



		flist.contains(new Apple());   // OK



		flist.indexOf(new Apple());    // OK



	}



}
```

对于上述例子，flist 的类型就是 List<? extends Fruit> 了，但这并不意味着可以向这个 List 可以添加任何类型的 Fruit，甚至于不能添加 Apple。虽然编译器知道这个 List 持有的是 Fruit，但并不知道其具体持有哪种特定类型(可能是List<Fruit>，List<Apple>，List<Orange>，List<Jonathan>)，所以编译器不知道该添加那种类型的对象才能保证类型安全（add 方法的参数为 “? extends Fruit” ），因而编译器杜绝任何添加任何类型的 Fruit。但是，对于诸如 get(int index)【我们进行读取操作时，编译器是允许的，而且编译器还知道 List 中的任何一个对象至少具有 Fruit 类型】、**contains(Object o) 和 indexof(Object o) 等操作，由于其参数类型不涉及通配符，因此编译器允许调用这些操作。** 

**因此，一旦执行这种向上转型，我们就丢掉向其中添加任何对象的能力。更一般地，编译器会直接拒绝对参数列表中涉及通配符的方法的调用。因此，这意味着将由泛型类的设计者来决定哪些调用地安全的，并使用 Object类型 作为其参数类型，例如 contains 方法和 indexof 方法。**例如,

```java
public class Holder<T> {



	



        private T value;



 



	public Holder() {



	}



 



	public Holder(T val) {



		value = val;



	}



 



	public void set(T val) {



		value = val;



	}



 



	public T get() {



		return value;



	}



 



	public boolean equals(Object obj) {



		return value.equals(obj);



	}



 



	public static void main(String[] args) {



		Holder<Apple> Apple = new Holder<Apple>(new Apple());



		Apple d = Apple.get();



		Apple.set(d);



 



 



		Holder<? extends Fruit> fruit = Apple; // OK



		Fruit p = fruit.get();



		d = (Apple) fruit.get(); // Returns ‘Fruit’，类型擦除，返回上界



 



		// No warning,运行时异常 java.lang.ClassCastException



		Orange c = (Orange) fruit.get(); 



 



 



		// fruit.set(new Apple()); // Cannot call set()，参数列表含通配符



		// fruit.set(new Fruit()); // Cannot call set()，参数列表含通配符



 



		fruit.equals(d); // OK，参数列表不含通配符



	}



}
```

## 3、超类型通配符 / 通配符的下界 / 逆变

我们可以使用超类型通配符指定通配符的下界, 例如 <? super MyClass>，这使得我们可以安全的传递一个对象到泛型类型中。有了超类型通配符，就可以向 Collection 写入了，如下图所示：

![img](Java%E6%B3%9B%E5%9E%8B.assets/20190119190941593.png)

由图片可知，参数 apples 是 Apple 或 Apple的某种基类型 (例如：Fruit，Object，…) 的 List，也就是说，该 List 可以是 List<Apple>, List<Fruit> 或 List<Object>等。但无论具体指的是哪一种，我们向其中添加 Apple 或 Apple的子类型 总是安全的。**但编译器不允许向该 List 放入一个 Fruit 对象， 因为该List的类型可能是List<Apple> , 这样将会违背泛型的本意。**对于List<? super Apple>，在读取容器元素时，由于该容器所包含的元素可能是 Object类型、 Fruit类型 和 Apple类型，因此，从容器所读取到的元素只能确定是Object类型的，如下面图片所示： 

![img](Java%E6%B3%9B%E5%9E%8B.assets/20190119191047194.png)

## 4、协变与逆变

逆变与协变用来描述类型转换（type transformation）后的继承关系，其定义：如果 A,B 表示类型，f(⋅)表示类型转换，≤ 表示继承关系（比如，A ≤ B 表示A是B的子类）；

> f(⋅) 是逆变（contravariant）的，当 A≤B 时有 f(B)≤f(A) 成立；
>
> f(⋅) 是协变（covariant）的，当 A≤B 时有 f(A)≤f(B) 成立；
>
> f(⋅) 是不变（invariant）的，当 A≤B 时上述两个式子均不成立，即f(A)与f(B)相互之间没有继承关系。 

接下来，我们看看 Java 中的常见类型转换的协变性、逆变性或不变性:

### (1).泛型

令f(A) = ArrayList<A>，那么f(⋅) 是逆变、协变还是不变的呢？如果是逆变，则ArrayList<Integer>是ArrayList<Number>的父类型；如果是协变，则ArrayList<Integer>是ArrayList<Number>的子类型；如果是不变，二者没有相互继承关系。由于实际上ArrayList<Number>和ArrayList<Integer>无关，所以泛型是不变的。

### (2).数组

令f(A) = A[]，容易证明数组是协变的：

## 5、实现泛型的协变与逆变

　　我们知道Java 中的泛型是不变的，可我们有时需要实现泛型的逆变与协变，怎么办呢？这时通配符 ? 派上了用场：

- **<? extends >实现了泛型的协变**，比如：

```java
ArrayList<? extends Apple> l3 = new ArrayList<>();



ArrayList<? extends Fruit> l4 = new ArrayList<>();



l4 = l3;
```

对于ArrayList<? extends Apple>类型，我们知道其表示某种具体类型(只是没有确定下来)，但是无论其具体指的是ArrayList<Apple>类型还是ArrayList<Jonathan>类型都是可以赋给ArrayList<? extends Fruit>类型的引用的，反之则不可以。因此，我们可以认为ArrayList<? extends Fruit>类型是ArrayList<? extends Apple> 类型的父类型，故<? extends>实现了泛型的协变。

- **<? super>实现了泛型的逆变**，比如：

```java
ArrayList<? super Apple> l1 = new ArrayList<>();



ArrayList<? super Fruit> l2 = new ArrayList<>();



l1 = l2;
```

对于 ArrayList<? super Fruit>类型，我们知道其表示某种具体类型(只是没有确定下来)，但是无论其具体指的是ArrayList<Fruit>类型还是ArrayList<Object>类型都是可以赋给ArrayList<? super Apple>类型的引用的，反之则不可以。因此，我们可以认为ArrayList<? super Apple>类型是ArrayList<? super Fruit>类型的父类型，故 <? super>实现了泛型的逆变。

## 6、PECS 准则 (producer-extends, consumer-super)

我们知道 <?> 表示：我想使用 Java 泛型来编写代码，而不是用原生类型；但是在当前这种情况下，我并不能确定下泛型参数的具体类型，因此用?表示任何某种类型。因此，根据我们对通配符的了解，使用无界通配符的泛型类不能够写数据，而在读取数据时，所赋值的引用也只能是 Object 类型。那么，我们究竟如何向泛型类写入、读取数据呢？

《Effective Java2》给出了答案。 PECS: producer(读取)-extends, consumer(写入)-super。换句话说，如果输入参数表示一个 T 的生产者，就使用<? extends T>；如果输入参数表示一个 T 的消费者，就使用<? super T>。总之，通配符类型可以保证方法能够接受它们应该接受的参数，并拒绝那些应该拒绝的参数。 比如，一个简单的 Stack API ：

```java
public class  Stack<E>{



    public Stack();



    public void push(E e);



    public E pop();



    public boolean isEmpty();



}
```

现在要实现 pushAll(Iterable<E> src) 方法，将实现 Iterable 接口的 src 的元素逐一入栈：

```java
public void pushAll(Iterable<E> src){



    for(E e : src)



        push(e)



}
```

那么问题就来了：假设有一个实例化Stack<Number>的对象stack(类型参数被实例化为Number)，显然， 我们向这个 stack 中加入Integer型或Float型元素都是可以的，因为这些元素本来就是Number型的。因此，src 就包括但不限于 Iterable<Integer>与Iterable<Float>两种可能。这时，在调用上述pushAll方法时，编译器就会产生type mismatch错误。原因是显而易见的，因为Java中泛型是不变的，Iterable<Integer> 与 Iterable<Float> 都不是 Iterable<Number>及其子类型中的一种。所以，我们对pushAll方法的设计就存在逻辑上的问题。因此，应改为：

```java
// Wildcard type for parameter that serves as an E producer



public void pushAll(Iterable<? extends E> src) {



    for (E e : src)



        push(e);



}
```

这样，我们就可以实现将实现Iterable接口的E类型的容器中的元素读取到我们的 Stack<E> 中。

那么，如果现在要实现 popAll(Collection<E> dst) 方法，将 Stack 中的元素依次取出并添加到 dst 中，如果不用通配符实现：

```java
// popAll method without wildcard type - deficient!



public void popAll(Collection<E> dst) {



    while (!isEmpty())



        dst.add(pop());   



}
```

同样地，假设有一个实例化Stack<Number> 的对象 stack ， dst 为 Collection<Object>，显然，这是合理的。但如果我们调用上述的 popAll(Collection<E> dst)方法，编译器会报出type mismatch错误，编译器不允许我们进行这样的操作。原因是显而易见的，因为Collection<Object>不是Collection<Number>及其子类型的一种。所以，我们对popAll方法的设计就存在逻辑上的问题。因此，应改为：

```java
// Wildcard type for parameter that serves as an E consumer



public void popAll(Collection<? super E> dst) {



    while (!isEmpty())



        dst.add(pop());



}
```

这样，我们就可以实现将 Stack<E> 中的元素读取到我们的 Collection 中。在上述例子中，在调用 pushAll方法时，src生产了E实例（produces E instances）；在调用popAll方法时 dst 消费了E实例（consumes E instances）。Naftalin与Wadler将PECS称为Get and Put Principle。

此外，我们再来学习一个例子：java.util.Collections的copy方法(JDK1.7)，它的目的是将所有元素从一个列表(src)复制到另一个列表(dest)中。显然，在这里，src是生产者，它负责产生T类型的实例；dest是消费者，它负责消费T类型的实例。这完美地诠释了PECS：

```java
// List<? extends T> 类型的 src 囊括了所有 T类型及其子类型 的列表  



// List<? super T> 类型的 dest 囊括了所有可以将 src中的元素添加进去的 List种类 



public static <T> void copy(List<? super T> dest, List<? extends T> src) {



    // 将 src 复制到 dest 中



    int srcSize = src.size();



    if (srcSize > dest.size())



        throw new IndexOutOfBoundsException("Source does not fit in dest");



 



    if (srcSize < COPY_THRESHOLD ||



        (src instanceof RandomAccess && dest instanceof RandomAccess)) {



        for (int i=0; i<srcSize; i++)



            dest.set(i, src.get(i));



    } else {



        ListIterator<? super T> di=dest.listIterator();



        ListIterator<? extends T> si=src.listIterator();



        for (int i=0; i<srcSize; i++) {



            di.next();



            di.set(si.next());



        }



    }



}
```

**因此，输入参数是生产者时，用 ? extends T；输入参数是消费者时，用 ? super T；输入参数既是生产者又是消费者时，那么通配符类型没什么用了：因为你需要的是严格类型匹配，这是不用任何通配符而得到的。**无界通配符<?> 既不能做生产者(读出来的是Object),又不能做消费者(写不进去)。

------

# 四.编译器如何处理泛型？

 通常情况下，一个编译器处理泛型有两种方式：

## 1、Code Specialization

在实例化一个泛型类或泛型方法时都产生一份新的目标代码（字节码or二进制代码）。例如，针对一个泛型list，可能需要针对string，integer，float产生三份目标代码。

## 2、Code Sharing

**对每个泛型类只生成唯一的一份目标代码；该泛型类的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。**

- ### C++中的模板（template）是典型的Code specialization实现

C++编译器会为每一个泛型类实例生成一份执行代码。执行代码中integer list和string list是两种不同的类型。这样会导致代码膨胀（code bloat），不过有经验的C＋＋程序员可以有技巧的避免代码膨胀。另外，在引用类型系统中，这种方式会造成空间的浪费。因为引用类型集合中元素本质上都是一个指针，没必要为每个类型都产生一份执行代码，而这也是Java编译器中采用Code sharing方式处理泛型的主要原因。

- ### Java 是典型的Code sharing实现

Java编译器通过Code Sharing方式为每个泛型类型创建唯一的字节码表示，并且将该泛型类型的实例都映射到这个唯一的字节码表示上。将多种泛型类形实例映射到唯一的字节码表示是通过类型擦除（type erasue）实现的。

------

# 五. 类型擦除

## 1、要点

**类型擦除，通过移除泛型类定义的类型参数并将定义中每个类型变量替换成对应类型参数的非泛型上界(第一个边界)，得到原生类型(raw type)。**类型擦除是 Java 泛型实现的一种折中，以便在不破坏现有类库的情况下，将泛型融入Java并且保证兼容性（泛型出现前后的Java类库互相兼容）。类型擦除指的是通过类型参数合并，将泛型类型实例关联到同一份字节码(Class 对象)上。编译器只为泛型类型生成一份字节码，并将其实例关联到这份字节码上。**类型擦除的关键在于从泛型类型中清除类型参数的相关信息，并且在必要的时候添加类型检查和类型转换的方法。**

擦除是在编译期完成的。类型擦除可以简单的理解为将泛型java代码转换为普通java代码，只不过编译器更直接点，将泛型java代码直接转换成普通java字节码。**泛型类型只有在静态类型检查期间才会出现，在此之后，程序中的所有泛型类型都将被擦除，并替换为它们的非泛型上界。因此，在泛型代码内部，无法获得任何有关泛型参数类型的信息。**

## 2、编译器是如何配合类型擦除的？

![img](Java%E6%B3%9B%E5%9E%8B.assets/20190119192103402.png)

## 3、类型擦除的主要过程

- ### **对于Pair<>：**

```java
// 代码示例 A



class Pair<T> {  



	private T value;  



	public T getValue() {  



		return value;  



	}  



	public void setValue(T  value) {  



		this.value = value;  



	}  



}  
```

- ### **Pair<>的原始类型为：**

```java
// 代码示例 B



class Pair {  



	private Object value;  



	public Object getValue() {  



		return value;  



	}  



	public void setValue(Object  value) {  



		this.value = value;  



	}  



}
```

以下类型擦除示例：

```java
// 代码示例 1



interface Comparable <A> { 



	public int compareTo( A that); 



} 



 



//.............................类型擦除后........................................



 



// 代码示例 1



interface Comparable { 



	public int compareTo( Object that); 



} 
// 代码示例 2



final class NumericValue implements Comparable <NumericValue> { 



	priva byte value;  



	public NumericValue (byte value) { this.value = value; }  



	public byte getValue() { return value; }  



	public int compareTo( NumericValue that) { return this.value - that.value; } 



} 



 



//.............................类型擦除后........................................



 



// 代码示例 2



final class NumericValue implements java.lang.Comparable{



	// 域



	private byte value;



	// 构造器



	public NumericValue(byte);



	// 方法



	public int compareTo(NumericValue);



	public volatile int compareTo(java.lang.Object); //桥方法



	public byte getValue( );



}
// 代码示例 3



class Collections {  



	public static <A extends Comparable<A>> A max(Collection <A> xs) { 



		Iterator<A> xi = xs.iterator(); 



		A w = xi.next(); 



		while(xi.hasNext()) { 



		    A x = xi.next(); 



		    if(w.compareTo(x) < 0) 



			   w = x; 



		} 



		return w; 



	} 



} 



 



//.............................类型擦除后........................................



 



// 代码示例 3



class Collections {  



	public static Comparable max(Collection xs) { 



		Iterator xi = xs.iterator(); 



		Comparable w = (Comparable) xi.next(); 



		while (xi.hasNext()) { 



		    Comparable x = (Comparable) xi.next(); 



		    if (w.compareTo(x) < 0) w = x; 



		} 



		return w; 



	} 



}  
// 代码示例 4



final class Test { 



	public static void main (String[] args) { 



		LinkedList<NumericValue> numberList = new LinkedList<NumericValue> (); 



		numberList.add(new NumericValue((byte)0));  



		numberList.add(new NumericValue((byte)1));  



		NumericValue y = Collections.max( numberList );  



	} 



}



 



//.............................类型擦除后........................................



 



// 代码示例 4



final class Test { 



	public static void main (String[ ] args) { 



		LinkedList numberList = new LinkedList(); 



		numberList.add(new NumericValue((byte)0));  ，



		numberList.add(new NumericValue((byte)1));  



		NumericValue y = (NumericValue) Collections.max( numberList );  



	} 



}
```

第一个泛型类被擦除后, A被替换为最左边界 Object。由于Comparable是一个泛型接口，所以Comparable<NumericValue>的类型参数NumericValue被擦除掉并将相关参数置换为Object，但是这直接导致NumericValue没有实现接口（重写）Comparable的compareTo(Object that)方法，于是编译器自动添加了一个桥方法（由编译器在编译时自动添加）。第二个示例中限定了类型参数的边界，A必须为Comparable的子类，按照类型擦除的过程，先将所有的类型参数替换为最左边界Comparable，得到最终的擦除后结果。

------

# 六. 泛型带来的问题及解决方法

## 1、类型检查所针对的对象

```java
public class Test10 {  



	public static void main(String[] args) {  



 



		ArrayList<String> arrayList1=new ArrayList();  



		arrayList1.add("1");  // 编译通过  



		arrayList1.add(1);    // 编译错误  



		String str1=arrayList1.get(0);  // 返回类型就是 String  



 



		ArrayList arrayList2=new ArrayList<String>();  



		arrayList2.add("1");  // 编译通过  



		arrayList2.add(1);    // 编译通过  



		Object object=arrayList2.get(0);  // 返回类型就是 Object  



 



		new ArrayList<String>().add("11");  // 编译通过  



		new ArrayList<String>().add(22);    // 编译错误  



		String string=new ArrayList<String>().get(0);  // 返回类型就是 String  



	}  



}
```

因此我们可以得出结论：**类型检查就是针对引用的，谁是一个引用，用这个引用调用泛型方法，就会对这个引用调用的方法进行类型检测，而无关它真正引用的对象。**

## 2、所有动作都发生在边界处（对传递进来的值，编译器进行额外的检查；对真正传递出去的值，编译器自动插入的转型）

因为类型擦除的问题，所以所有的泛型类型最后都会被替换为原始类型。这样就引起了一个问题，既然都被替换为原始类型，那么为什么我们在获取的时候，不需要进行强制类型转换呢？先看下面非泛型示例：

```java
// 代码片段1



public class SimpleHolder {



	private Object obj;



	public void setObj(Object obj) {



		this.obj = obj;



	}



	public Object getObj() {



		return obj;



	}



	public static void main(String[] args) {



		SimpleHolder holder = new SimpleHolder();



		holder.setObj("Item");



		String s = (String)holder.getObj();



	}



} 
```

反编译这个类，得到下面代码片段：

```java
public void setObj(java.lang.Object);



  Code:



   0:   aload_0



   1:   aload_1



   2:   putfield        #2; //Field obj:Ljava/lang/Object;



   5:   return



 



public java.lang.Object getObj();



  Code:



   0:   aload_0



   1:   getfield        #2; //Field obj:Ljava/lang/Object;



   4:   areturn



 



public static void main(java.lang.String[]);



  Code:



   0:   new     #3; //class SimpleHolder



   3:   dup



   4:   invokespecial   #4; //Method "<init>":()V



   7:   astore_1



   8:   aload_1



   9:   ldc     #5; //String Item



   11:  invokevirtual   #6; //Method setObj:(Ljava/lang/Object;)V



   14:  aload_1



   15:  invokevirtual   #7; //Method getObj:()Ljava/lang/Object;



   18:  checkcast       #8; //class java/lang/String



   21:  astore_2



   22:  return
```

现将泛型应用到上述代码，如下：

```java
// 代码片段 2



public class GenericHolder<T> {



	private T obj;



	public void setObj(T obj) {



		this.obj = obj;



	}



	public T getObj() {



		return obj;



	}



	public static void main(String[] args) {



		GenericHolder<String> holder = new GenericHolder<String>();



		holder.setObj("Item");



		String s = holder.getObj();



	}



}
```

反编译这个类，得到下面代码片段：

```java
public void setObj(java.lang.Object);



  Code:



   0:   aload_0



   1:   aload_1



   2:   putfield        #2; //Field obj:Ljava/lang/Object;



   5:   return



 



public java.lang.Object getObj();



  Code:



   0:   aload_0



   1:   getfield        #2; //Field obj:Ljava/lang/Object;



   4:   areturn



 



public static void main(java.lang.String[]);



  Code:



   0:   new     #3; //class GenericHolder



   3:   dup



   4:   invokespecial   #4; //Method "<init>":()V



   7:   astore_1



   8:   aload_1



   9:   ldc     #5; //String Item



   11:  invokevirtual   #6; //Method setObj:(Ljava/lang/Object;)V



   14:  aload_1



   15:  invokevirtual   #7; //Method getObj:()Ljava/lang/Object;



   18:  checkcast       #8; //class java/lang/String



   21:  astore_2



   22:  return
```

在上述应用泛型的代码中，将：

```java
String s = holder.getObj();
```

替换为：

```java
holder.getObj();
```

反编译后，有代码片段：

```java
public void setObj(java.lang.Object);



  Code:



   0:   aload_0



   1:   aload_1



   2:   putfield        #2; //Field obj:Ljava/lang/Object;



   5:   return



 



public java.lang.Object getObj();



  Code:



   0:   aload_0



   1:   getfield        #2; //Field obj:Ljava/lang/Object;



   4:   areturn



 



public static void main(java.lang.String[]);



  Code:



   0:   new     #3; //class GenericHolder



   3:   dup



   4:   invokespecial   #4; //Method "<init>":()V



   7:   astore_1



   8:   aload_1



   9:   ldc     #5; //String Item



   11:  invokevirtual   #6; //Method setObj:(Ljava/lang/Object;)V



   14:  aload_1



   15:  invokevirtual   #7; //Method getObj:()Ljava/lang/Object;



   18:  pop



   19:  return



}
```

首先，代码片段 1 和代码片段 2 二者所产生的字节码是相同的。看第15，它调用的是getObj()方法，返回值是Object，说明类型擦除了。然后第18，它做了一个checkcast操作，即检查类型#8， 在上面找#8引用的类型，它是一个String类型，即作String类型的强转。所以不是在get方法里强转的，**是在你调用的地方强转的**。对进入setObj()的类型进行检查是不需要的，因为这将由编译器执行。而对从getObj()返回的值进行转型仍旧是需要的，但这与你自己必须执行的操作是一样的–此处它将由编译器自动插入。也就是说，在泛型中，所有动作都发生在边界处，**对传递进来的值进行额外的编译器检查，并由编译器自动插入对传递出去的值的转型。**

其次，在未将 getObj() 的值赋给String时，由代码片段可知，编译器并未自动插入转型代码，可见所谓编译器自动插入对传递出去的值的转型的前提条件是：**其必须是真正传递出去，即必须赋值给引用。**(当然，虽然 getObj() 的返回值的类型是 Object， 但是其实质上是一个 String， 因此直接进行操作 “ getObj() instanceof String ”时，返回值也是 true。)再看一段代码：

```java
public class GenericArray<T> {



	private T[] array;



 



	public GenericArray(int sz) {



		array = (T[]) new Object[sz];



	}



 



	public void put(int index, T item) {



		array[index] = item;



	}



 



	public T get(int index) {



		return array[index];



	}



	public T[] rep() { return array; }



 



	public static void main(String[] args) {



		GenericArray<Integer> gai = new GenericArray<Integer>(10);



		gai.put(0, new Integer(4));



 



		gai.get(0);



		Integer i = gai.get(0);



 



		// This causes a ClassCastException:



		  Integer[] ia = gai.rep();



 



		// This is OK:



		Object[] oa = (Object[])gai.rep();



	}



}
```

反编译得代码段：

```java
public class GenericArray extends java.lang.Object{



public GenericArray(int);



  Code:



   0:   aload_0



   1:   invokespecial   #1; //Method java/lang/Object."<init>":()V



   4:   aload_0



   5:   iload_1



   6:   anewarray       #2; //class java/lang/Object



   9:   checkcast       #3; //class "[Ljava/lang/Object;"



   12:  putfield        #4; //Field array:[Ljava/lang/Object;



   15:  return



 



public void put(int, java.lang.Object);



  Code:



   0:   aload_0



   1:   getfield        #4; //Field array:[Ljava/lang/Object;



   4:   iload_1



   5:   aload_2



   6:   aastore



   7:   return



 



public java.lang.Object get(int);



  Code:



   0:   aload_0



   1:   getfield        #4; //Field array:[Ljava/lang/Object;



   4:   iload_1



   5:   aaload



   6:   areturn



 



public java.lang.Object[] rep();



  Code:



   0:   aload_0



   1:   getfield        #4; //Field array:[Ljava/lang/Object;



   4:   areturn



 



public static void main(java.lang.String[]);



  Code:



   0:   new     #5; //class GenericArray



   3:   dup



   4:   bipush  10



   6:   invokespecial   #6; //Method "<init>":(I)V



   9:   astore_1



   10:  aload_1



   11:  iconst_0



   12:  new     #7; //class java/lang/Integer



   15:  dup



   16:  iconst_4



   17:  invokespecial   #8; //Method java/lang/Integer."<init>":(I)V



   20:  invokevirtual   #9; //Method put:(ILjava/lang/Object;)V



   23:  aload_1



   24:  iconst_0



   25:  invokevirtual   #10; //Method get:(I)Ljava/lang/Object;



   28:  pop



   29:  aload_1



   30:  iconst_0



   31:  invokevirtual   #10; //Method get:(I)Ljava/lang/Object;



   34:  checkcast       #7; //class java/lang/Integer



   37:  astore_2



   38:  aload_1



   39:  invokevirtual   #11; //Method rep:()[Ljava/lang/Object;



   42:  checkcast       #12; //class "[Ljava/lang/Integer;"



   45:  astore_3



   46:  aload_1



   47:  invokevirtual   #11; //Method rep:()[Ljava/lang/Object;



   50:  checkcast       #3; //class "[Ljava/lang/Object;"



   53:  astore  4



   55:  return



}
```

结合上面的结论，仔细观察反编译后代码中checkcast都用在什么地方，加深对边界就是发生动作的地方和自动转型发生在调用处(需要检验两种类型时)的理解。

25显式调用后，直接pop，而31显示在调用处，还要进行 checkcast 操作；

由于类型擦除，操作39之后，进行 checkcast 操作，强转为 Ljava.lang.Integer ，但是由代码【 array = (T[]) new Object[sz]; 】可知，其 new 的是 Object 数组，是不可能成功强转到 Integer 数组的，就像 Object 对象不能成功强转到 Integer 对象一样，会在运行时抛出 ClassCastException 异常；

由于类型擦除，操作47之后，进行 checkcast 操作，由于 rep() 返回的即为 Object 数组，而其要赋给的引用也是 Object[] ,因此不会抛出任何异常。

## 3、类型擦除与多态的冲突及其解决办法

　　先看两段代码：

```java
// 第一段代码



public class Pair<T> {



	private T first;



	private T second;



 



	public Pair(T first, T second){



		this.first = first;



		this.second = second;



	}



	public void setFirst(T first){



		this.first = first;



	}



	public T getFirst(){



		return first;



	}



	public void setSecond(T second){



		this.second = second;



	}



	public T getSecond(){



		return second;



	}



}
// 第二段代码



public class DateInterval extends Pair<Date> {     // 时间间隔类



	public DateInterval(Date first, Date second){



		super(first, second);



	}



 



	@Override



	public void setSecond(Date second) {



		super.setSecond(second);



	}



	@Override



	public Date getSecond(){



		return super.getSecond();



	}



 



	public static void main(String[] args) {



		DateInterval interval = new DateInterval(new Date(), new Date());



		Pair<Date> pair = interval; //超类，多态



		Date date = new Date(2000, 1, 1);



		System.out.println("原来的日期："+pair.getSecond());



		System.out.println("set进新日期："+date);



		pair.setSecond(date);



		System.out.println("执行pair.setSecond(date)后的日期："+pair.getSecond());



	}



}
```

原本子类重写父类的方法，无可非议。但是泛型类的类型擦除造成了一个问题，Pair的原始类型中存在方法:

```java
public void setSecond(Object second);
```

DateInterval 中的方法:

```java
public void setSecond(Date second);
```

我们的本意是想重写父类Pair中的setSecond方法，但是从方法签名上看，这完全是两个不同的方法，类型擦除与多态产生了冲突。而实际情况呢？运行DateInterval的main方法，我们看到public void setSecond(Date second)的确重写了public void setSecond(Object second)方法。这是如何做到的呢？

　　使用 Java 类分析器 对其进行分析，结果：

```java
public class DateInterval extends Pair{



	// 构造器



	public DateInterval(java.util.Date, java.util.Date);



	// 方法



	public void setSecond(java.util.Date);



	public volatile void setSecond(java.lang.Object); // 方法 1



	public java.util.Date getSecond( );               // 方法 2



	public volatile java.lang.Object getSecond( );    // 方法 3，它难道不会和方法 2 冲突？



	public static void main(java.lang.String[]);



}
```

方法1和方法3是我们在源码中不曾定义的，它肯定是由编译器生成的。这个方法称为 **桥方法(bridge method)**，真正覆写超类方法的是它。语句pair.setSecond(date)实际上调用的是方法1[public volatile void setSecond(Object)]，通过这个方法再去调用public void setSecond(Date)。这个桥方法的实际内容是：

```java
public void setSecond(Object second){



    this.setSecond((java.util.Date) second);



}
```

这样的结果就符合面向对象中多态的特性了，实现了方法的动态绑定。但是，这样的做法给我们带来了一种错觉，就认为public void setSecond(Date)覆写了泛型类的public void setSecond(Object)【其实也不是重写，二者方法参数都不同】，如果我们在DateInterval中增加一个方法：

```java
public void setSecond(Object obj){



	System.out.println("覆写超类方法！");



}
```

 编译器会报如下错误：Name clash: The method setSecond(Object) of type DateInter has the same erasure as setSecond(T) of type Pair but doesn’t override it.即，同一个方法不能被重写两次。

为了实现多态，我们知道方法3也是由编译器生成的桥方法。方法擦除带来的第二个问题就是：**由编译器生成的桥方法 public volatile java.lang.Object getSecond() 方法和 public java.util.Date getSecond() 方法，从方法签名的角度看是两个完全相同的方法，它们怎么可以共存呢？** 如果是我们自己编写Java代码，这样的代码是无法通过编译器的检查的，但是虚拟机却是允许这样做的，因为虚拟机通过参数类型和返回类型来确定一个方法，所以编译器为了实现泛型的多态允许自己做这个看起来“不合法”的事情。

## 4、泛型类型变量不能是基本数据类型

**类型参数不能是基本类型。**也就是说，没有ArrayList<double>，只有ArrayList<Double>。因为当类型擦除后，ArrayList的原始类型变为Object，但是Object类型不能存储double值，只能引用Double的值。

## 5、转型和警告

**使用带有泛型类型参数的转型或 instanceof 不会有任何效果**，例如：

```java
class FixedSizeStack<T> {



	private int index = 0;



	private Object[] storage;



 



	public FixedSizeStack(int size) {



		storage = new Object[size];



	}



 



	public void push(T item) {



		storage[index++] = item;



	}



 



	public T pop() {



		// Warnning: Unchecked cast from Object to T



		return (T) storage[--index];



	}



}



 



public class GenericCast {



	public static final int SIZE = 10;



 



	public static void main(String[] args) {



		FixedSizeStack<String> strings = new FixedSizeStack<String>(SIZE);



		for (String s : "A B C D E F G H I J".split(" "))



			strings.push(s);



		for (int i = 0; i < SIZE; i++) {



			String s = strings.pop();



			System.out.print(s + " ");



		}



	}



}
```

由于擦除的原因，T 被擦除到它的第一个边界 Object，因此pop()实际上只是将Object转型为Object。换句话说，pop()方法实际上并没有执行任何转型。

## 6、任何在运行时需要知道确切类型信息的操作都将无法工作

> 1、instanceof 操作的右操作数不能带有泛型类型参数；
>
> 2、new 操作 : 可以 new 泛型类型(eg: ArrayList,…)，但不能 new 泛型参数(T,…)；
>
> 3、泛型数组 : **除非使用通配符**，不可以创建带有泛型类型参数的数组（若需要收集参数化类型对象，可以直接使用 ArrayList：ArrayList<Pair<String>>最安全且有效。）；
>
> 4、转型 : 带有泛型类型参数的转型不会有任何效果；

例如： 

![img](Java%E6%B3%9B%E5%9E%8B.assets/20190119194116319.png)

关于由类型擦除引起的 instance of T，new T 和创建数组T 等问题，可以引入类型标签Class<T>来解决，例如：

```java
class Building {}



 



class House extends Building {}



 



public class ClassTypeCapture<T> {



	Class<T> kind;



 



	public ClassTypeCapture(Class<T> kind) {



		this.kind = kind;



	}



 



	public boolean f(Object arg) {



		return kind.isInstance(arg);



	}



 



	public static void main(String[] args) {



 



		ClassTypeCapture<Building> ctt1 = new ClassTypeCapture<Building>(Building.class);



		System.out.println(ctt1.f(new Building()));       // true



		System.out.println(ctt1.f(new House()));          // true



 



		ClassTypeCapture<House> ctt2 = new ClassTypeCapture<House>(House.class);



		System.out.println(ctt2.f(new Building()));       // true



		System.out.println(ctt2.f(new House()));          // true



	}



}
```

------

## **7、实现参数化接口**

**一个类不能实现同一个泛型接口的两种变体，由于擦除的原因，这两个变体会成为相同的接口**，例如：

```java
public Person implements Comparable<Person>{  ...   }   // OK



 



class HonorPerson extends Person implements Comparable<HonorPerson>{  ...   }       // Error
```

HonorPerson 类不能编译，因为擦除会将 Comparable<Person>和Comparable<HonorPerson> 简化为相同的接口Comparable， 上面的代码意味着重复实现相同的接口。但是，下面的代码可以通过编译：

```java
public Person implements Comparable{  ...   }  // OK



 



class HonorPerson extends Person implements Comparable{  ...   }   // OK
```

这种差别在于：编译器对泛型的特别处理方式。

## 8、异常中使用泛型的问题

由于类型擦除的原因，将泛型应用于异常是非常受限的。catch 语句不能捕获泛型类型的异常，因为在编译期和运行时都必须知道异常的确切类型。 

### (1). 不能抛出也不能捕获泛型类的对象

事实上，泛型类扩展Throwable都不合法(Exception是Throwable的子类)。例如：下面的定义将不会通过编译:

```java
public class Problem<T> extends Exception{......}  
```

为什么不能扩展Throwable，因为异常都是在运行时捕获和抛出的，而在编译的时候，泛型信息全都会被擦除掉，那么，假设上面的编译可行，那么，再看下面的定义：

```java
try{  



}catch(Problem<Integer> e1){  



...  



}catch(Problem<Number> e2){  



...  



}   
```

　在运行时，类型信息被擦除后，那么两个地方的catch都变为原始类型Object，那么也就是说，这两个地方的catch变的一模一样,就“相当于”下面的这样：

```java
try{  



}catch(Problem<Object> e1){  



...  



}catch(Problem<Object> e2){  



...  



}  
```

这当然就是不行的, 就好像catch了两个一模一样的普通异常,编译器就不能通过编译一样。

### (2). 不能再catch子句中使用泛型变量

　　例如：

```java
public static <T extends Throwable> void doWork(Class<T> t){  



	try{  



		...  



	}catch(T e){ //编译错误  



		...  



	}  



}   
```

因为泛型信息在编译的时候已经变为原始类型，也就是说上面的 T 会变为原始类型 Throwable，那么如果可以再catch子句中使用泛型变量，那么，下面的定义呢：

```java
public static <T extends Throwable> void doWork(Class<T> t){  



	try{  



		...  



	}catch(T e){ //编译错误  



		...  



	}catch(IndexOutOfBounds e){  



	}                           



 }   
```

根据异常捕获的原则，一定是子类在前面，父类在后面，那么上面就违背了这个原则。所以java为了避免这样的情况，禁止在catch子句中使用泛型变量。

### (3). 类型变量可以使用在异常声明中

```java
public static<T extends Throwable> void doWork(T t) throws T{  



    try{  



        ...  



    }catch(Throwable realCause){  



        t.initCause(realCause);  



        throw t;   



    }  



}
```

此时，虽然 T 也会被擦除为 Throwable，但由于用在声明中，因此是合法的。

## **9、类型擦除后的冲突**

　　当泛型类型被擦除后，创建条件不能产生冲突，例如：

```java
class Pair<T>   {  



    public boolean equals(T value) {  



        return null;  



    }  



}
```

考虑Pair<>：

```java
public boolean equals(T value){}
```

擦除后变为：

```java
boolean equals(Object)
```

这与 Object.equals 方法是冲突的！当然，补救的办法是重新命名引发错误的方法。

## **10、动态类型安全**

　　先看以下代码：

```java
public class CheckedList { 



	@SuppressWarnings("unchecked") 



	static void oldStyleMethod(List probablyDogs) {   // 原生List



		probablyDogs.add(new Cat()); 



	} 



	public static void main(String[] args) { 



		List<Dog> dogs1 = new ArrayList<Dog>(); 



		oldStyleMethod(dogs1);   // Quietly accepts a Cat 



		List<Dog> dogs2 = Collections.checkedList(new ArrayList<Dog>(), Dog.class); 



 



		try { 



			oldStyleMethod(dogs2);   // Throws an exception 



		} catch(Exception e) { 



			System.out.println(e); 



		} 



 



		// Derived types work fine: 



		List<Pet> pets = Collections.checkedList( 



		new ArrayList<Pet>(), Pet.class); 



		pets.add(new Dog()); 



		pets.add(new Cat()); 



	} 



} 



 



/** Output: 



java.lang.ClassCastException: Attempt to insert class typeinfo.pets.Cat 



element into collection with element type class typeinfo.pets.Dog



*/
```

使用 Collections 的静态方法：checkedCollection(), checkedList(), checkedMap(), checkedSet(),checkedSortedMap() 和 checkedSortedSet() 可以在运行时便知道罪魁祸首在哪里，而不必等到将对象从容器中取出时。

# 七. 小结

泛型本意即为类型参数化，提供了更为广泛的表达能力，需要注意两点：

> **1、泛型只存在于编译期阶段，该阶段完成了针对引用的类型检查，将泛型参数替换为非泛型上界并由编译器在边界处自动插入转型；**
>
> **2、每个泛型类只生成唯一的一份目标代码，该泛型类的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。**