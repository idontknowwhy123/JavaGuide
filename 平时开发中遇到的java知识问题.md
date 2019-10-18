1.java中需要用equals来判断两个字符串值是否相等

但在java中，这个代码即使在两个字符串完全相同的情况下也会返回false<br> 
Java中必须使用string1.equals(string2)来进行判断<br> 
eg:<br> 
string s1="Hello";<br>
string s2="Hello";<br>
则(s1==s2)=true;<br>
因为他们指向的同一个对象。<br>
 eg:<br>
String s1=new String("Hello");<br> 
String s2=new String("Hello");<br> 
则(s1==s2)=false<br>
如果把其他变量的值赋给s1和s2，即使内容相同，由于不是指向同一个对象，也会返回false。所以建议使用equals()，因为equals比较的才是真正的内容<br> 
例如：<br> 
String string1=new String( "aaa" );<br> 
String string2=new String( "aaa" );<br> 
这两个字符串当然应该是相等的。<br> 
如果用表达式string1==string2，则该表达式的值为false<br> 
如果用表达式string1.equals(string2)，则该表达式的值为true <br>
因此应该用string1.equals(string2)，在if语句中就是 <br>
if(string1.equals(string2)==true) //字符串相等，……<br>

string1==string2,是值相等,而且内存地址也相等,是完全的相等<br> 
string1.equals(string2)为true,只是值相等<br>

总结:1）对于==，如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等；

　　　　如果作用于引用类型的变量，则比较的是所指向的对象的地址

　　2）对于equals方法，注意：equals方法不能作用于基本数据类型的变量

　　　　如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址；

　　　　诸如String、Integer、Double、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。

2.List en=null，定义了Entity的集合变量，并且实例化为null，与前面一个不同的是他可以被使用，但仅限于equals、==等判断或者其它非取值等操作；<br>
想用的话也是需要实例化或者里面已经有值了,否则会报错空指针<br>

    List<MoRecord> mos=null;<br>
    mts = new ArrayList<MtRecord>();<br>
    mts.addAll(mtArrList);<br>
Listen=new ArrayList() 定义并且实例化为Arraylist，这个时候就可以做所有的List和ArrayList的操作，比如添加值、取值、迭代等等操作。<br> 


3.Exception in thread "main" java.util.ConcurrentModificationException<br>
  问题场景<br>
 1     public void test1()  {<br>
 2         ArrayList<Integer> arrayList = new ArrayList<>();<br>
 3         for (int i = 0; i < 20; i++) {<br>
 4             arrayList.add(Integer.valueOf(i));<br>
 5         }
 6 
 7         // 复现方法一
 8         Iterator<Integer> iterator = arrayList.iterator();<br>
 9         while (iterator.hasNext()) {<br>
10             Integer integer = iterator.next();<br>
11             if (integer.intValue() == 5) {<br>
12                 arrayList.remove(integer);<br>
13             }<br>
14         }<br>
15 
16         // 复现方法二
17         iterator = arrayList.iterator();<br>
18         for (Integer value : arrayList) {<br>
19             Integer integer = iterator.next();<br>
20             if (integer.intValue() == 5) {<br>
21                 arrayList.remove(integer);<br>
22             }<br>
23         }<br>
24     }<br>

解决办法:<br>
如果要在foreach循环中删除list中的元素，要使用itrator迭代器，借助itrator的remove方法删除元素，若使用list的remove方法则会抛出异常<br>

·如果要在foreach循环中添加list元素，则要另外new一个list。因为直接对list使用add，会抛出异常，而itrator并没有刻意向list中添加元素的方法。<br>
所以也无法借助iterator。所以可以采取另外new一个list，然后借助list接口的addAll方法，将原来的list整个加入到新list中，此时循环旧的list，<br>
调用新的list的add方法添加元素就可以达到目的（hashmap是一样的处理方式）。<br>

多线程下的解决方案：<br>
 public static void test5() {<br>
 2         ArrayList<Integer> arrayList = new ArrayList<>();<br>
 3         for (int i = 0; i < 20; i++) {<br>
 4             arrayList.add(Integer.valueOf(i));<br>
 5         }<br>
 6 
 7         Thread thread1 = new Thread(new Runnable() {<br>
 8             @Override<br>
 9             public void run() {<br>
10                 synchronized (arrayList) {<br>
11                     ListIterator<Integer> iterator = arrayList.listIterator();<br>
12                     while (iterator.hasNext()) {<br>
13                         System.out.println("thread1 " + iterator.next().intValue());<br>
14                         try {<br>
15                             Thread.sleep(100);<br>
16                         } catch (InterruptedException e) {<br>
17                             e.printStackTrace();<br>
18                         }<br>
19                     }<br>
20                 }<br>
21             }<br>
22         });<br>
23 
24         Thread thread2 = new Thread(new Runnable() {<br>
25             @Override<br>
26             public void run() {<br>
27                 synchronized (arrayList) {<br>
28                     ListIterator<Integer> iterator = arrayList.listIterator();<br>
29                     while (iterator.hasNext()) {<br>
30                         Integer integer = iterator.next();<br>
31                         System.out.println("thread2 " + integer.intValue());<br>
32                         if (integer.intValue() == 5) {<br>
33                             iterator.remove();<br>
34                         }<br>
35                     }<br>
36                 }<br>
37             }<br>
38         });<br>
39         thread1.start();<br>
40         thread2.start();<br>
41     }<br>

4.某个微服务A依赖于另外一个微服务B，则应该先编译微服务B，则微服务A才能正常调用微服务B<br>

5.基本数据类型与字符串之间的转换<br>
5.1.int转为String<br>
    整型转换成字符型<br>
　　String num = Integer.toString(int n);或者是String.valueOf(int n)<br>
　  Long型转换成字符型<br>
　　String num = Long.toString(long n);<br>或者是String.valueOf(long n)<br>
　　Short型转换成字符型<br>
　　String num = Short.toString(Short n);<br>或者是String.valueOf(Short n)<br>
　　Float型转换成字符型<br>
　　String num = Float.toString(Float n);<br>或者是String.valueOf(Float n)<br>
　　Double型转换成字符型<br>
　　String num = Double.toString(Double n);<br>或者是String.valueOf(Double n)<br>
  
5.2.String转换成int<br>
　　转换成Int型<br>
　　int/Integer num = Integer.parseInt(String str);<br>
　　转换成long型<br>
　　Long/long num = Long.parseLong(String str);<br>
　　转换成short型<br>
　　short/Short num = Short.parseShort(String str);<br>
　　转换成float型<br>
　　float/Float num = Float.parseFloat(String str);<br>
　　转换成double型<br>
　　double/Double num = Double.parseDouble(String str);<br>
  
 6.java对象转化为String类型<br>
  方法一：object.toString()（常用）<br>
  方法二：（String）object<br>
  方法三：String.valueOf(Object)<br>
  
 7.
 1.1   throw是语句抛出一个异常。<br>
       语法：throw (异常对象);<br>
       throw e;<br>

1.2   throws是方法可能抛出异常的声明。(用在声明方法时，表示该方法可能要抛出异常)<br>
      语法：[(修饰符)](返回值类型)(方法名)([参数列表])[throws(异常类)]{......}<br>
      public void doA(int a) throws Exception1,Exception3{......}<br>
      
8.java.lang.NoSuchMethodError的解决办法<br>
错误可能的原因：<br>

有这个类，该类真的没有这个方法<br>
有这个类，而且有好几个，他们之间发生了冲突<br>
方法本身是存在的，方法所在类也是存在的，那么在运行时还会出现这个错误，就只能说明运行时引用的类里面没有这个方法。<br>
这说起来有点拗口，简单的说，就是存在至少两个类名一样的类A和B，其中A有一个need方法，B则没有这个方法。编译时，编译器发现依赖路径下有需要的类A或者B，
则编译通过。<br>
但是执行时，在要调用A.need()方法时，因为A和B同名，错误的调用了B.need()方法，这个方法本身是不存在的，自然就出现了NoSuchMethodError这个错误
解决办法<br>
通过分析可以发现，其实问题产生的根本原因是类有冲突，也就是存在多个满足条件的类A,B,C..<br>
这种情况一般出现在引用某个库或者jar时，同时引用了多个版本而导致的。<br>

9.Collections.emptyList()<br>
先说明一下好处有哪些：<br>
1，如果你想 new 一个空的 List ，而这个 List 以后也不会再添加元素，那么就用 Collections.emptyList() 好了。
new ArrayList() 或者 new LinkedList() 在创建的时候有会有初始大小，多少会占用一内存。
每次使用都new 一个空的list集合，浪费就积少成多，浪费就严重啦，就不好啦
2，为了编码的方便。<br>
比如说一个方法返回类型是List，当没有任何结果的时候，返回null,有结果的时候，返回list集合列表。
那样的话，调用这个方法的地方，就需要进行null判断。使用emptyList这样的方法，可以方便方法调用者。返回的就不会是null，省去重复代码。
注意的地方：<br>
这个空的集合是不能调用.add（），添加元素的。因为直接报异常。因为源码就是这么写的：直接抛异常。

10.方法外定义的变量，如果是基本类型变量（或者String），作为传参传入方法里面进行处理后，该基本类型变量在方法外还是不会改变，如果是引用类型，
作为传参传入方法里面进行处理后，该引用类型变量会改变。
https://blog.csdn.net/fenglllle/article/details/81389286

11.将一个引用类型对象在函数外进行定义，作为传参写入函数，如果对值进行修改，则函数外的值会修改，因为引用地址会指向这个修改后的值，
如果要对值进行赋值则不会被修改，因为引用地址还是指向原来那个值<br>
例如：List list = new Arraylist();<br>
     setValue(list);//list不会变<br>
     setValue2(list);//list会变<br>
public static void setValue(List list){<br>
   list = getValue1(String s1);<br>
}<br>
public static void setValue2(List list){<br>
   list.add(1);<br>
}<br>

12.try catch编程习惯
1.在写程序时，对可能会出现异常的部分通常要用try{…}catch{…}去捕捉它并对它进行处理；

2.用try{…}catch{…}捕捉了异常之后一定要对在catch{…}中对其进行处理，那怕是最简单的一句输出语句，或栈输入e.printStackTrace();用try{…}catch{…}捕捉了异常之后一定要对在catch{…}中对其进行处理，那怕是最简单的一句输出语句，或栈输入e.printStackTrace();

3.如果是捕捉IO输入输出流中的异常，一定要在try{…}catch{…}后加finally{…}把输入输出流关闭；如果是捕捉IO输入输出流中的异常，一定要在try{…}catch{…}后加finally{…}把输入输出流关闭；

4.如果在函数体内用throw抛出了某种异常，最好要在函数名中加throws抛异常声明，然后交给调用它的上层函数进行处理。如果在函数体内用throw抛出了某种异常，最好要在函数名中加throws抛异常声明，然后交给调用它的上层函数进行处理。

13.Java 中一共有四种访问权限控制，其权限控制的大小情况是这样的：public > protected > default(包访问权限) > private ,具体的权限控制看下面表格，列所指定的类是否有权限允许访问行的权限控制下的内容：

访问权限	本类	本包的类	子类	非子类的外包类
public	  是	    是    是	    是
protected	是	   是	   是	    否
default	 是	    是	   否	    否
private	 是	    否	   否	    否


14、JAVA中String类以形参传递到函数里面，修改后外面引用不能获取到更改后的值
这个问题真正原因是因为String类的存储是通过final修饰的char[]数组来存放结果的。不可更改。所以每次当外部一个String类型的引用传递到方法内部时候，只是把外部String类型变量的引用传递给了方法参数变量。对的。外部String变量和方法参数变量都是实际char[]数组的引用而已。所以当我们在方法内部改变这个参数的引用时候，因为char[]数组不可改变，所以每次新建变量都是新建一个新的String实例。很显然外部String类型变量没有指向新的String实例。所以也就不会获取到新的更改。



