## Java集合中的Fail-Fast快速失败机制

### 1 简介

`fail-fast`快速失败机制，是java集合中的一种错误的检测的机制，当在迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生`fail-fast`机制，即抛出`ConcurrentModificationException`异常。

> 发生这种情况的根本原因是我们使用集合的迭代器遍历集合的时候，我们会有一个`expectedModCount`值，这个值在创建我迭代器的时候会使用ArrayList的`modCount`这个值设置成为`expectModCount`，但是如果我们在使用迭代器遍历的过程中我们调用`ArrayList`集合的`remove`方法的话就会修改`modCount`这个值，当调用`iter.next`获取元素的时候，我们首先会调用`checkForComodification();`这个方法，这个方法里面会比较迭代器当中的`expectedModCount`值和list当中`modCount`这两个值是否相等，如果相等的话，就会抛出`ConcurrentModificationException`异常，这就是我们所说的快速失败的场景。

### 2 fail-fast快速失败的场景

    当我们使用线程不安全的`ArrayList`或者`HashMap`的时候，都会出现快速失败的场景。

    使用`ArrayList`集合的时候出现的场景：

```java
public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(i + "");
        }
        Iterator<String> iterator = list.iterator();
        int i =0;

        while (iterator.hasNext()){
            if(i == 3){
                //会修改ArrayList集合当中的modCount值
                list.remove(i);
            }
            //此时会比较modCount和expectModCount
            System.out.println(iterator.next());
            i++;
        }
    }
```

       `ArrayList`集合中的`remove()`方法源码如下：

```java
public E remove(int index) {
        rangeCheck(index);
           
        //修改了modCount的值
        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

> **疑问1：初始化创建的迭代器的时候迭代器的`expectedModCount`值是多少？**
> 
> 在调用集合的方法创建集合的迭代器的时候，初始的`expectModCount`值就是集合的`modCount`值。
> 
> **疑问2：初始化后创建后的迭代器的`expectedModCount`值就不改变了吗？**
> 
> 是的，一旦迭代器初始化完成，迭代器的`exceptModCount`值就不再改变。
> 
> ```java
>         //指向当前元素的指针，用来获取当前元素
>         int cursor;       
>         //指向上一个元素
>         int lastRet = -1; 
>         //从这儿就可以看到初始化的时候是将expectedModCount设置为modCount
>         int expectedModCount = modCount;
> ```

    当我们的集合执行了一次`remove`方法之后，`modCount`的值就改变了，后面我们使用迭代器方法`iterator.next()`获取元素的时候,该方法会首先校验`modCount`和`expectModCount`值是否相等，如果不相等就抛出`ConcurrentModification`异常。

```java
        public E next() {
            //先校验modCount的值和expectModCount值是否相等
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        //校验方法
        final void checkForComodification() {
            //不想等就抛出异常
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

    至此，我们就直到抛出`ConcurrentModificationException`异常的原因了。同样的，不仅仅是`ArrayList`，`HashMap`也存在这样的问题。在下面的代码中也会抛出`ConcurentModificationException`,原因是和`ArrayList`一样的，也是我们获取到了map的`entry`结构的迭代器，将迭代器当中的`exceptModCount`值设置为了`modCount`，但是后续在调用`map.remove()`方法的时候会将map当中的`modcount`值修改，后面在调用的时候也会比较`modCount`和`exceptModCount`的值，此时发现不一样，就会抛出`ConcurrentModificationException`异常。

```java
@Test
    public void testHashMap(){
        Map<String, String> map = new HashMap<>();
        for (int i = 0; i < 10; i++) {
            map.put(i + "", "" + i);
        }
        Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();
        int i = 0;
        while (iterator.hasNext()){
            if(i == 3){
                map.remove(3 + "");
            }
            Map.Entry<String, String> res = iterator.next();
            System.out.println(res);
            i++;
        }
    }
```

    以上是单线程情况下出现`ConcurrentModificationException`异常的情况，此外多线程下最有可能出现这种异常，以下是代码示例。

```java
public class FailFastTest {
     public static List<String> list = new ArrayList<>();

     private static class MyThread1 extends Thread {
           @Override
           public void run() {
                Iterator<String> iterator = list.iterator();
                while(iterator.hasNext()) {
                     String s = iterator.next();
                     System.out.println(this.getName() + ":" + s);
                     try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                }
                super.run();
           }
     }

     private static class MyThread2 extends Thread {
           int i = 0;
           @Override
           public void run() {
                while (i < 10) {
                     System.out.println("thread2:" + i);
                     if (i == 2) {
                           list.remove(i);
                     }
                     try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                     i ++;
                }
           }
     }

     public static void main(String[] args) {
           for(int i = 0 ; i < 10;i++){
            list.add(i+"");
        }
           MyThread1 thread1 = new MyThread1();
           MyThread2 thread2 = new MyThread2();
           thread1.setName("thread1");
           thread2.setName("thread2");
           thread1.start();
           thread2.start();
     }
}
```

#### 3 快速失败的解决方法

    上面我们了解到`ArrayList`和`HashMap`的快速失败的原因是`modCount`和`expectModCount`值不想等引起的，那么我们只要不修改`ArrayList`的`modCount`值的然后进行删除就行了，这样的话`modCOunt`和`expectModCount`的值就是一样的了。所以我们可以调用迭代器的`remove()`方法进行删除从该方法的源码当中我们可以看出来他是没有修改`modCount`值的。

1、    不抛出`ConcurrentModificationException`的解决方法。

```java
public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(i + "");
        }
        Iterator<String> iterator = list.iterator();
        int i =0;
        while (iterator.hasNext()){
            if(i == 3){
                iterator.remove();
            }
            System.out.println(iterator.next());
            i++;
        }
        System.out.println(list);
}

//迭代器的remove方法
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```

2、使用`JUC`下的集合安全的类来操作

    除了使用迭代器的`remove`方法来进行删除之外（使用迭代器的方法进行删除是有一定局限的，你只能删除你遍历到的元素），还可以使用`JUC`包下的线程安全的集合`CopyOnWriteList`和`ConcurrentHashMap`集合进行操作，`CopyOnWriteList`使用了写时复制的技术，当我们进行对该集合进行`add`、`remove`操作的时候，我们不是对原来的集合进行操作的，我们采用写时复制的技术将原来的数据拷贝一份，在新的数组上进行修改，待完成之后，我们才会将引用指向新的数组，但是如果是读取操作还是读的是老数组上的元素。所以`CopyOnWriteArrayList`数组并不会有快速失败的机制产生。但是这种情况导致的就是`CopyOnWriteArrayList`不能保证数据的强一致性，只能保证最终一致性，因为你在对一个集合修改的时候，其他的线程读取还是读的老数组。

    `CopyOnWriteArrayList`集合的`remove()`、`add()`方法的源码如下：

```java
    //删除方法
    public E remove(int index) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }

     //add方法源码
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

    对于`ConcurrentHashHashMap`也可以解决这种问题，`ConcurrentHashMap`采用了锁机制，是线程安全的。`ConcurrentHashMap`的迭代器是弱一致性的迭代器，在这种迭代方式中，当`iterator`被创建之后集合再发生改变就不再是抛出`ConcurrentModificationException`,取而代之的是改变的时候new新的数据从而不影响原有的数据，`iterator`完成之后再将头指针替换为新的数据，这样`iterator`线程可以使用原来的老的数据，而写线程可以并发的完成修改。
