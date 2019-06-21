[TOC]







# 前言





#  一、`ArrayList` 的容量伸缩机制

为了解决数据结构中数组定长的缺点，Java 提供了具有容量伸缩机制的 `ArrayList` 。



## 1.节点插入前的扩容

> 节点插入的步骤：
>
> （1）检查索引是否合法
>
> （2）确保容量充足（扩容）
>
> （3）元素后移
>
> （4）元素插入



在节点插入前，通过`ensureCapacityInternal`方法来确保容量充足且符合要求

```java

    /**
     * 尾插
     * @param e 待插入元素
     * @return 是否插入成功
     */
    public boolean add(E e) {
        // 确保内容容量
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 数组末尾插入元素e
        elementData[size++] = e;
        return true;
    }


    /**
     * 插入到指定位置
     * @param index
     * @param element
     */
    public void add(int index, E element) {
        // 检查索引（index）是否合法
        rangeCheckForAdd(index);

        // 确保内容容量
        ensureCapacityInternal(size + 1);  // Increments modCount!!

        // 将 index及其之后的元素后移一位
        System.arraycopy(elementData, index, elementData, index + 1,
                size - index);

        // 将元素插入到index位置
        elementData[index] = element;

        //
        size++;
    }
```























# 参考资料

## 1.精选资料

1. [Java集合源码分析（一）ArrayList](https://www.cnblogs.com/zhangyinhua/p/7687377.html)
2. [java集合框架综述](https://www.cnblogs.com/xiaoxi/p/6089984.html)



## 2.参考资料

1. [Java源码解读——ArrayList（二）](https://iamxi.iteye.com/blog/1452278)
2. [Java 位运算(移位、位与、或、异或、非）](<https://blog.csdn.net/xiaochunyong/article/details/7748713>)
3. [二进制]([https://zh.wikipedia.org/wiki/%E4%BA%8C%E8%BF%9B%E5%88%B6](https://zh.wikipedia.org/wiki/二进制))
4. 



