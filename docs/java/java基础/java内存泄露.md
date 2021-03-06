http://www.i3geek.com/archives/1241

# Java内存泄露

> 内存泄露是指当不再使用的对象没有得到释放，还占有内存，从而造成内存浪费的情况。

在C++中，内存是由程序员进行管理的，从内存的创建、使用和释放都是程序员去操作。很多时候因为疏忽忘记对对象的释放，从而导致无用对象不断增加，导致内存不足，产生内存泄露的现象。

## 原因

一般产生内存泄露的原因主要有如下两种：第一种，没有释放掉不需要的内存；第二种，内存对象明明已经不需要，但还保留着这块内存和它的访问引用。

在java中，对于对象的回收是由垃圾回收器进行处理的，并不需要程序员的干涉，所以基本不存在第一种情况。那产生内存泄漏的原因主要是第二种情况，因为程序员的操作导致明明不需要的、该被回收的对象，没有被认定为垃圾，没有回收掉，则产生了内存泄露。

### 根本原因

根本原因就是，长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄露，尽管短生命周期对象已经不再需要，但是因为长生命周期对象持有它的引用而导致不能被回收。

说的比较抽象 看看下面的例子：

	Vector v = new  Vector( 100 );  
	for  ( int  i = 1 ;i < 100 ; i ++ ){  
		Object o = new  Object();  
		v.add(o);  
		o = null ;  //认为此时想回收o
	}

此时将o置为null后仍然不被进行回收。因为短生命周期（Object）对象虽然被置空，但是长生命周期的Vector仍然持有。所以导致不需要的对象，仍然存在，没有被回收，产生内存泄露。此时的解决办法是，将v也置null

## 常见场景

### 集合类

1、如上例，不同生命周期

2、在HashMap中，加入数据后，修改数据内容，导致hashcode不同，从而无法根据hashcode删除元素，产生内存溢出

	Set<Person> set = new HashSet<Person>(); 
	Person p1 = new Person("张三","pwd1",25); 
	set.add(p1); //设：此时p1的hashcode为25，HashSet根据25存储
	p1.setAge(2); //修改对象，此时p1的hashcode为1
	set.remove(p1);//根据hashcode删除时，找不到对象，删除失败
 
### 单例模式

使用单例模式的类，长期持有单例的对象，可能会导致内存泄露。

## 避免办法

使用java中的多种引用，比如弱引用、软引用和虚引用等，在不同需求下进行使用，从而避免内存泄漏