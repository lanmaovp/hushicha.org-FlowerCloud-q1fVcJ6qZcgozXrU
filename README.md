**目录**

* [序](#_label0)
* [概述](#_label1)
+ [对象大小如何计算](#_lab2_1_0)
- [对象头](#_label3_1_0_0)
- [对象内容](#_label3_1_0_1)
- [8字节对齐](#_label3_1_0_2)

* [计算对象大小的方法](#_label2)
+ [方法1：基于 JDK 原生库 【推荐】](#_lab2_2_0)
+ [方法2：基于lucene\-core库](#_lab2_2_1)
+ [案例分析(基于lucene\-core库)](#_lab2_2_2)
- [Integer 对象大小分析](#_label3_2_2_0)
- [HashMap 对象大小分析](#_label3_2_2_1)

* [X 参考文献](#_label3)


[回到顶部(Back to Top)](#_labelTop)# 序


* 在Java应用程序的性能优化场景中，时常需要考虑Java对象的大小，以便观测、评估后，进一步提出优化方案：



> * 占用内存的大小。（比如 本地内存）
> * 对象数据在网络传输中占用的网络带宽
> * 对象数据在存储时占用的磁盘空间
> * ...


[回到顶部(Back to Top)](#_labelTop):[楚门加速器](https://chuanggeye.com)# 概述


## 对象大小如何计算


* **对象大小**包括俩部分的内容，对象头和对象内容：


![](https://img2024.cnblogs.com/blog/1173617/202501/1173617-20250109203514536-1485639451.png)


### 对象头



> 此处假设是64位的JVM


* 对象地址，占4个字节。
* 对象标记，占8个字节，包括锁标记，hashcode, age 等。
* 数组长度标记，占4个字节。如果对象是一个数组，会有此标记，否则没有。


### 对象内容


* 对象内部属性。如果属性是对象的话，那么记录的是对象的地址，占用4个字节。


### 8字节对齐


* `Java`对象采用的是8字节对齐。**对象大小**必须是`8`的倍数，不足需要补齐。



> 比如，计算一个对象只需要20字节，那么实际占用24字节。


[回到顶部(Back to Top)](#_labelTop)# 计算对象大小的方法


* 方法1和方法2，在 String 对象的计算上，存在差异；Integer 和 Map 的计算，经简单检验：不存在差异。


## 方法1：基于 JDK 原生库 【推荐】



> jdk 1\.8



```
import jdk.nashorn.internal.ir.debug.ObjectSizeCalculator;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class ObjectCalculatorTest {
    public static void main(String[] args) {
        String text = "Hello World!"; //12 chars
        Long objectSizeBytes = ObjectSizeCalculator.getObjectSize(text);
        log.info("objectSizeBytes: {} bytes", objectSizeBytes );//64 bytes

        Integer number = new Integer(11);
        objectSizeBytes = ObjectSizeCalculator.getObjectSize(number);
        log.info("objectSizeBytes: {} bytes", objectSizeBytes );//16 bytes

		Map map = new HashMap<>();
        objectSizeBytes = ObjectSizeCalculator.getObjectSize(number);
        log.info("objectSizeBytes: {} bytes", objectSizeBytes );//48 bytes
    }
}

```

## 方法2：基于`lucene-core`库


* 引入依赖



```
<dependency>
	<groupId>org.apache.lucenegroupId>
	<artifactId>lucene-coreartifactId>
	<version>8.7.0version>
dependency>

```

* 计算对象大小



> jdk 1\.8



```
import lombok.extern.slf4j.Slf4j;
import org.apache.lucene.util.RamUsageEstimator;

@Slf4j
public class ObjectCalculatorTest {
    public static void main(String[] args) {
        String text = "Hello World!"; //12 char = 12 byte
        objectSizeBytes = RamUsageEstimator.shallowSizeOf( text );
        log.info("objectSizeBytes: {} bytes", objectSizeBytes );//24 bytes
		
    	Integer number = new Integer(11);
        System.out.println(RamUsageEstimator.shallowSizeOf(number));//16 bytes

		Map map = new HashMap<>();
		System.out.println(RamUsageEstimator.shallowSizeOf(map));//48 bytes
    }
}

```

## 案例分析(基于`lucene-core`库)


### Integer 对象大小分析


* 它是对象，占用4个字节
* 对象标记，占用8个字节
* 查看源码，发现:



> `Integr` 内容只有以下一个**非static**的属性，是一个`int`的基本类型属性，占用`4`个字节.
> `static` 修饰的方法属性都是存储在方法区的，不占用对象空间。



```
    /**
     * The value of the {@code Integer}.
     *
     * @serial
     */
    private final int value;

```


> 故 total \= 4 \+ 8 \+ 4 \= 16


### HashMap 对象大小分析


* 它是对象，占用4个字节
* 对象标记，占用8个字节
* 查看源码，发现：



> HashMap 是继承了 AbstractMap 的，AbstractMap 中有以下的俩个属性，一共占用8个字节。因为只是存储了keySet, values 的地址



```
transient Set        keySet;
transient Collection values;

```

* HashMap 中有以下属性，共占用 6 \* 4 \= 24 个字节。



```
transient Node[] table;
transient Set> entrySet;
transient int size;
transient int modCount;
int threshold;
final float loadFactor;

```


> total \= 4 \+ 8 \+ 8 \+ 24 \= 44, 由于 java 是8字节对齐的，故一共是 48 字节。


[回到顶部(Back to Top)](#_labelTop)# X 参考文献


* [计算Java对象大小(附实际例子分析) \- CSDN](https://github.com)


