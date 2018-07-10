title: ArrayList源码分析
author: Chen Mr
tags:
  - 源码分析
categories: []
date: 2018-07-10 22:33:00
---
# 学习 ArrayList 源码
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
```
## 拷贝
```java
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

```
在上面这个构造函数当中使用了 `Arrays.copyOf` 这是个拷贝数组的方法，底层实现是
new 一个数组再用 `System.arraycopy` 拷贝数组，长度使用的是新数组的长度，return 返回 copy 数组。
### 浅拷贝和深拷贝
**浅拷贝**：使用一个已知实例对新创建实例的成员变量逐个赋值，这个方式被称为浅拷贝。

**深拷贝**：当一个类的拷贝构造方法，不仅要复制对象的所有非引用成员变量值，还要为引用类型的成员变量创建新的实例，并且初始化为形式参数实例值。这个方式称为深拷贝。

也就是说浅拷贝只复制一个对象，传递引用，不能复制实例。而深拷贝对对象内部的引用均复制，它是创建一个新的实例，并且复制实例。

```
int[] array1 = {1,2};
int[] array2 = array1;
```

## 解读 modeCount
```
protected transient int modCount = 0;
```
`modCount`在代码当中顾名思义是 `ArrayList` 的修改次数。
- 不在序列化。有关键字`transient`的存在并且`ArrayList`实现了`Serializable`它也是不在序列化当中的
- 是非线程安全的。在 `add()`, `remove()`等方法当中都能看到它在自增，当多线程在添加的时候就会出错。

## 解读自动扩容
### add(E e)
当使用 `add(E e)` 时，如果 size + 1 的值大于现在容量（默认大小为 10），则需要扩容：

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

// 在方法 ensureCapacityInternal(int minCapacity) 里调用 
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 右移 1 相当于原数字除以 2，所以结果是 1.5 倍原数字
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
拷贝一个新数组，新数组大小为为原来的 1.5 倍。原数组被抛弃，等待 gc。

### add(int index, E element)
当使用 `add(int index, E element)` 则与 `add(E e)`不同。后者是在数组末尾添加，前者是数组中间添加，则需要使用`System.arraycopy`自己拷贝自己：

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // index 原数组开始位置， index + 1 目标数组开始位置，size - index 多少数据 
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}

// index=2,e=9
// [1,2,3] -> [1,2,2,3]
// [1,2,9,3]
```
在拷贝后是需要赋值`elementData[index] = element`再`size++`。
## remove(int index)
```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

// index=0 
// [1,2,3,4] -> [2,3,4,4]
// [2,3,4]
```
假如 index = 0, 数组为 [1,2,3,4] 那么需要拷贝数组,拷贝后数组为 [2,3,4,4],然后去掉数组最后一位。

## set(int index, E element) && get(int index)
```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}

public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```
`set()`,`get()`不会涉及自动扩容

# 总结
- ArrayList 是由**数组**实现的，因此在内存当中是连续的，**查询**速度很**快**。
- 但是由于**增删**时可能需要 copy 数组并额外开辟空间，所以速度很**慢**。



######  感谢龙哥投稿