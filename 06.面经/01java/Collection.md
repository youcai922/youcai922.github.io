# ArrayList和LinkedList区别

ArrayList底层数组，插入删除慢，查找快；ArrayList主要控件开销在于需要在lList列表预留一定空间；

LinkedList底层链表（双向链表），查找慢，插入删除快；LinkList主要控件开销在于需要存储结点信息以及结点指针信息。

# hashmap是线程安全的吗

线程不安全，不安全的情况主要是resize()扩容方法导致的。

