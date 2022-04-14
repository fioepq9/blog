---
title: go面试题
date: 2022-04-06 19:07:10
tags:
  - golang
categories:
  - 面试题
---

## Golang的GC算法了解吗？

go使用的是三色标记法，属于标记清除法。这里的三色，对应了垃圾回收过程中对象的三种状态：灰色，代表对象在标记队列中；黑色，代表对象已被标记；白色，代表对象未被标记。在垃圾回收开始时，会将根对象标记为灰色，也就是放入标记队列中，然后会从标记队列中取出一个灰色对象，将它引用的对象加入队列中后将它自身变为黑色，直到队列为空。最终黑色对象保留，白色对象被回收。三色标记法实际就是对根对象进行一次BFS，最后没有遍历到的，即为需要回收垃圾。

### 根对象是什么？

<details>
    <summary>Answer</summary>
    <ol>
        <li>全局变量</li>
        <li>执行栈</li>
        <li>寄存器</li>
    </ol>
</details>

### 没有STW会出现什么问题？

<details>
    <summary>Answer</summary>
    <p>
        内存误回收：考虑一个已经扫描完毕的黑色对象，让它引用一个新建的白色对象。最终这个白色对象会被回收。
    </p>
</details>

### 常见的垃圾回收算法

<details>
    <summary>Answer</summary>
    <ul>
        <li>引用计数法：对每个对象维护一个引用计数，当计数为0时回收该对象。</li>
        <ul>
            <li>优点：对象可以很快被回收，不会出现内存耗尽或达到某个阈值时才回收。</li>
            <li>缺点：不能很好地处理循环引用，而且实时维护引用计数，空间开销比较大。</li>
        </ul>
        <li>标记清除法：从根变量开始遍历所有引用的对象并标记，最后回收所有没有被标记的对象。</li>
        <ul>
            <li>优点：可以处理循环引用，并且不用实时维护引用计数，减少了空间上的开销。</li>
            <li>缺点：需要STW，即暂停程序运行。</li>
        </ul>
        <li>分代收集法：按照对象生命周期长短划分，生命周期长的放入老年代，而短的放入新生代，不同代有不同的回收算法和回收频率。</li>
        <ul>
            <li>优点：回收性能好。</li>
            <li>缺点：算法复杂。</li>
        </ul>
    </ul>
</details>

## GMP模型

<details>
    <summary>Answer</summary>
    <ul>
        <li>G：指goroutine。</li>
        <li>M：工作线程，在go中被称为Machine。</li>
        <li>P：处理器，是go中定义的一个概念，它包含运行go代码的必要资源。</li>
    </ul>
    <ul>
        <p>goroutine调度策略</p>
        <ul>
            <li>每个P维护一个包含G的本地队列，P周期性地从本地队列头部取出G调度到M中执行一小段时间后将上下文保存下来，然后将G放到本地队列尾部。除了本地队列以外，还有一个全局队列。每个P都会周期性地查看全局队列。如果一个P的本地队列中的G已全部执行完毕，那么会从全局队列中拿一批G放入本地队列；如果全局队列中也没有G，那么它会尝试将其它P的本地队列中的G偷取一半放入自身的本地队列。</li>
            <li>当G进入系统调用时，所属的M会释放P，P会被其它M获取（可能是空闲的M，如果没有空闲的M，那么新建一个M），原本的M由于系统调用而阻塞。</li>
        </ul>
    </ul>
</details>

## 内存逃逸

<details>
    <summary>Answer</summary>
    <p>
        在一段程序中，每一个函数都会有自己的内存区域存放自己的局部变量、返回地址等，这些内存会由编译器在栈中进行分配，每一个函数都会分配一个栈桢，在函数运行结束后进行销毁，但是有些变量我们想在函数运行结束后仍然使用它，那么就需要把这个变量在堆上分配，这种从"栈"上逃逸到"堆"上的现象就成为内存逃逸。
    </p>
</details>

### 逃逸策略

<details>
    <summary>Answer</summary>
    <ol>
        <li>如果对象在函数外部没有引用，则优先放到栈中。（如果需要内存过大，则放到堆中）</li>
        <li>如果对象在函数外部存在引用，则必定放到堆中。</li>
    </ol>
</details>

## Slice的特点

<details>
    <summary>Answer</summary>
    <ul>
        <li>每个Slice都指向一个底层数组。</li>
        <li>使用<code>len()</code>计算Slice长度的时间复杂度为O(1)。</li>
        <li>使用<code>cap()</code>计算Slice容量的时间复杂度为O(1)。</li>
        <li>通过函数传递Slice时，不会拷贝底层数组。</li>
        <li>通过<code>append()</code>向切片追加元素时可能触发扩容。</li>
    </ul>
</details>

### Slice的扩容机制？

<details>
    <summary>Answer</summary>
    <ol>
        <li>如果原Slice的容量小于1024，则新Slice的容量扩大为原来的2倍。</li>
        <li>如果原Slice的容量大于等于1024，则新Slice的容量扩大为原来的1.25倍。</li>
    </ol>
</details>

## Map什么时候会触发扩容？

<details>
    <summary>Answer</summary>
    <p>满足下列两个条件之一：</p>
    <ul>
        <li>负载因子 > 6.5时，即平均每个bucket存储键值对达到6.5个以上。</li>
        <li>overflow数量 > 2^15时，即overflow的数量超过32768时。</li>
    </ul>
</details>

### Map的扩容机制？

<details>
    <summary>Answer</summary>
    <ul>
        <p>增量扩容(负载因子过大触发)</p>
        <ul>
            <li>新建一个新的buckets，容量为原来的2倍。</li>
            <li>采用渐进式rehash，在访问map时触发迁移。</li>
        </ul>
    </ul>
    <ul>
        <p>等量扩容(overflow数量过高触发)</p>
        <ul>
            <li>新建一个新的buckets，容量跟原来一样。</li>
            <li>采用渐进式rehash，在访问map时触发迁移。</li>
        </ul>
    </ul>
</details>

## defer的规则

<details>
    <summary>Answer</summary>
    <ol>
        <li>延迟函数的参数在defer语句出现时就已经确定了。</li>
        <li>延迟函数执行按后进先出顺序执行，即先出现的defer最后执行。</li>
        <li>延迟函数可能操作使用defer语句的函数的具名返回值。</li>
    </ol>
</details>

## go的多值返回是怎么实现的？

<details>
    <summary>Answer</summary>
    为了实现多值返回，go是使用栈空间来返回值的。而常见的C语言是通过寄存器来返回值的。go在调用函数的时候，会在栈中预留返回值的空位，在返回前将具体值赋值。
</details>

## 字符串类型的底层实现？

<details>
    <summary>Answer</summary>
    底层是一个字符串指针加上一个cap。是一种不可变类型，每次更换内容，实际是更换底层指针。
</details>

## for range遍历slice的过程中不断向slice进行append，会死循环吗？

<details>
    <summary>Answer</summary>
    不会。for range是go的一个语法糖，它在进入循环之前会先获取len(slice)，然后执行len(slice)次数的循环。
</details>

## go和c/c++的区别？

<details>
    <summary>Answer</summary>
    <ol>
        <li>go不支持隐式类型转换，c/c++支持。</li>
        <li>go要求统一的代码风格，并且有gofmt这个格式化工具。</li>
        <li>go通过标识符的首字母是否大写决定可见性，而c/c++通过private、public等关键字。</li>
        <li>go支持多返回值，c/c++不支持。</li>
        <li>go支持匿名函数，c/c++只存在类似的lambda表达式。</li>
        <li>go在语言层面支持并发，c/c++需要调用外部库。</li>
        <li>go有自己的垃圾回收器，而c/c++只能手动进行垃圾回收。</li>
    </ol>
</details>
