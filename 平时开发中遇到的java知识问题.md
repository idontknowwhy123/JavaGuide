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
调用新的list的add方法添加元素就可以达到目的。<br>

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
  
