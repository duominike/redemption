# java集合
## list和set
### UML类图
 #### List
 无论哪种list,都实现了list接口，基于list的操作是一致的
```java
boolean add(T);
boolean remove(T);
int size();
boolean isEmpty();
boolean contains(Object o);
Iterator<E> iterator
boolean addAll(Collection<? extends E> c);
boolean containsAll(Collection<?> c);
boolean  removeAll(Collection<?> c);
Object[] toArray();
```
- ArrayList:
	底层是基于数组实现的，遍历和访问元素都比较快，但是插入和删除麻烦
- LinkedList
	底层是基于双向链表实现的，插入和删除效率比较高，但是访问不如ArrayList快捷。
	LinkedList的一些方法：
```java
public void addFirst(E e); //添加元素到队首
public E getFirst();// 返回队首元素 不删除
public E removeFirst();// 删除队首
pubic void addLast(E e);// 在队尾添加元素
public E getLast();// 返回队尾元素
public E removeLast();// 移除队尾元素
public E poll();//移除队首元素并返回
public E pollFirst();// 移除列表中的第一个元素并返回
public E pollLast();// 移除列表中的最后一个元素并返回
public E peek();// 返回队首元素 不删除
public E peekFirst();// 返回队首元素 不删除
public E peekLast();// 返回队尾元素 不删除
```
#### set
- 接口set
	set的特点是：存入set中的每个元素都必须是唯一的，因此加入set中的元素必须定义equals()方法，确保对象的唯一性。set与collection拥有完全一样的接口，set接口不保证维护元素的唯一性。

- HashSet
	HashSet是基于HashMap实现的，存入HashSet的元素必须实现hashCode()方法
- TreeSet
	保持次序的Set，底层是树结构。添加的元素必须实现Comparable接口
- LinkedHashSet
	具有HashSet的查询速度，且内部使用链表维护元素的顺序。遍历时会按插入顺序遍历。也必须实现hashCode()方法
## map
### UML类图
###
- **HashMap**
	实现一个映射，允许存储空对象，而且允许键是空（由于键必须是唯一的，当然只能有一个）。
- **LinkedHashMap**
	HashMap的一个子类，但是迭代遍历的时候，取得“键值对”的顺序是其插入顺序。
- **TreeMap**
	基于红黑树的实现。查看键或键值树的时候，它们会被排序。所得到的结果是经过排序的。TreeMap是唯一带有subMap()方法的map,可以返回一颗子树
- **HashTable**
	 实现一个映射，所有的键必须非空。为了能高效的工作，定义键的类必须实现hashcode()方法和equals()方法
## 注意事项
- equals()方法必须满足以下条件
	1. 自反性:x.equals(x)一定返回true
	2. 对称性:x.equals(y) 与 y.equals(x) 结果一直
	3. 传递性：如果 x.equals(y), y.equals(z) 则 x.equals(z)
	4. 一致性:如果x和y比较信息没有改变，则无论调多少次， x.equals(y) 结果应该保持一致
	5. 对任何不是null的x,x.equals(null)返回false
- 默认对象的 Object.equals() 比较的是对象的地址
- 设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该产生相同的值。因此hashCode()里面的方法不要依赖对象中容易改变的数据。也不应该依赖具有唯一性的对象信息，比如说this的值。因为这样做无法产生一个新的键，使之与put()中原始的键值相同。
- String作为hashmap的键：
	string有个特点，如果程序中有多个String，都包含相同的字符串序列，那么这些String对象都映射到同一块内存区域。所以new String("hello") 生成的两个实例，虽然是相互独立的，但是对他们使用hashCode()应该生成相应的结果。对于String而言，hashCode()明显是基于String内容的。有可能不同字符串内容得到的hashCode()；经常有用hashMap以字符串为键，判断结果却不正确的情况。选择的时候请慎重