# Hash



## Hash结构

 ` 比较`

* 在我们之前学习的顺序结构和平衡树中，**元素关键码与其存储位置之间没有对应的关系**，因此当我们要查找一个元素时，要进行多次关键码的比较。顺序表的时间复杂度是O(N)，搜索树为树的高度，平衡搜索树为O(log_2 N)，它们搜索的效率取决与元素的比较次数。

* 那有没有一种理想的存储结构：不经过任何比较，直接根据关键码与存储位置之间的关系来确定存储位置  -> 哈希表：

当向该元素中：

> * 插入元素：根据元素的关键码，以此函数计算出该元素的存储位置并按此位置进行存放
> * 搜索元素：对元素的关键码进行同样的计算，把求得的函数值当作元素的存储位置，在结构中按此位置取元素比较，若关键码相同，则搜索成功。

## 哈希函数

* 哈希函数就是通过一个元素关键码来计算出它在哈希表中的存储位置，哈希函数设计的不合理很可能会增加哈希冲突的次数

### 哈希函数的设计原则：

> * 哈希函数的定义域必须包括需要存储的全部关键码，而如果散列表允许有m个地址时，其值域必须在0到m-1之间
> * 哈希函数计算出来的地址能均匀分布在整个空间
> * 哈希函数应该比较简单

### 常见的哈希函数

* 直接定址法

> 取关键字的某个线性函数为散列地址：Hash（Key）= A*Key + B
>
> 优点：简单、均匀
>
> 缺点：需要事先知道关键字的分布情况
>
> 使用场景：适合查找比较小且**连续**的情况

* 除留余数法

> 设散列表中允许的地址数为m，取一个不大于m，但最接近或者等于m的质数p作为除数，按照哈希函数：Hash(key) = key% p(p<=m),将关键码转换成哈希地址



## 负载因子

* 哈希表中的负载因子决定了什么时间进行扩容操作

>  散列表的负载因子 = 填入表中的元素个数 / 散列表的长度
>
> 负载因子越大，表明表中的元素越多，哈希冲突的概率越大，负载因子越小，产生冲突的可能性就越小。



## 哈希冲突

对于两个数据元素的关键字$k_i$和 $k_j$(i != j)，有$k_i$ != $k_j$，但有：Hash($k_i$) == 

Hash($k_j$)，即：不同关键字通过相同哈希哈数计算出相同的哈希地址，

**该种现象称为哈希冲突或哈希碰撞**。

### 解决哈希冲突

解决哈希冲突主要有两种方法：闭散列和开散列

### 闭散列

* 闭散列：也叫开放定址法，当发生哈希冲突时，如果哈希表未被填满，说明哈希表必然有着空闲的位置，那么就可以把要插入的元素放到空闲的位置上去，这就是开放地址法，那么如何找到下一个空闲的位置呢？主要有线性探测和二次探测

1. 线性探测：从发生冲突的位置开始依次向后探测，直到下一个空位置为止。

> * 插入：通过哈希函数获取待插入元素在哈希表中的位置，如果这个位置满了，就使用线性探测找到下一个为空的位置（可以是空，也可以是delete）
> * 删除：通过哈希函数找到要删除的位置，如果这个位置不是就一直向后使用线性探测进行寻找，直到找到空位置为止，将这个为止标记为delete

代码实现：

```c++
template <typename T>
struct HashFunc{
    size_t operator()(const T& key){
        return (size_t)key;
    }
};

//模板的特化
template<> 
struct HashFunc<string>{
    size_t operator()(const string& key){
        size_t num = 0;
        for(auto e : key){
            num += e;
            num *= 31;
        }
        return num;
    }
};
//使用开放定址法（闭散列）实现哈希表
namespace openaddress{
    enum State{
        EXIST,
        EMPTY,
        DELETE
    };

    template<typename K, typename V> struct HashNode{
        HashNode() = default;
        HashNode(const pair<K, V>& kv)
        :_kv(kv),
        _state(EMPTY)
        {}

        pair<K, V> _kv;
        State _state = EMPTY; 
    };

    template<typename K, typename V, typename hash = HashFunc<K>>
    class Hash{
    private:
        using Node = HashNode<K, V>;
    public:
        Hash(){ _hash.resize(10); }
        bool erase(const K& key);
        Node* find(const K& key);
        bool insert(const pair<K, V>& kv);

    private:
        vector<Node> _hash;
        size_t _n = 0; //c++的默认构造函数对于自定义类型是不会进行初始化操作的，所以这个要自己来进行初始化
    };

    template<typename K, typename V, typename hash>
    bool Hash<K, V, hash>::insert(const pair<K, V>& kv){
        //去重
        if(find(kv.first)) return false;
        //使用除留余数法进行数据的定位
        hash hs;
        if(_n / (double)_hash.size() >= 0.7){
            //进行扩容，扩容时不能直接对_hash进行resize因为扩容之后sz发生了变化映射关系也就发生了变化
            //所以合适的扩容方法是新建一个表，然后把旧的数据重新插入到新表当中，然后进行vector的交换
            Hash<K,V, hash> new_hash;
            new_hash._hash.resize(2*_hash.size());
            for(int i = 0; i < _hash.size(); ++i){
                if(_hash[i]._state == EXIST){
                    pair<K,V> l_kv = _hash[i]._kv;
                    new_hash.insert(l_kv);
                }
            } 
            _hash.swap(new_hash._hash);
        }

        //插入操作
        size_t sz = _hash.size();
        size_t hashi = hs(kv.first) % sz;
        while(_hash[hashi]._state == EXIST){
            hashi = (hashi+1)%sz;
        }
        _hash[hashi]._kv = kv;
        _hash[hashi]._state = EXIST;
        ++_n;
        return true;
    }

    template<typename K, typename V, typename hash>
    typename Hash<K, V, hash>::Node* Hash<K, V, hash>::find(const K& key){
        hash hs;
        size_t sz = _hash.size();
        size_t hashi = hs(key) % sz;
        while(_hash[hashi]._state != EMPTY){
            if (_hash[hashi]._state == EXIST && _hash[hashi]._kv.first == key){
                return &_hash[hashi];
            }
            hashi = (++hashi)%sz;
        }
        return nullptr;
    }

};
```

线性探测的缺点：**一旦发生哈希冲突，所有的冲突连在一起，容易产生数据“堆积”，即：不同关键码占据了可利用的空位置，使得寻找某关键码的位置需要许多次比较，导致搜索效率降低**。

### 开散列

* **开散列又叫链地址法，首先对关键码集合用散列函数计算散列地址，具有相同地址的关键码归于同一子集合，每一个子集合成为一个桶，各个桶中的元素通过一个单链表链接起来，各链表的头节点存储在哈希表中。**

![QQ20240728-181535](C:/Users/Administrator/Desktop/QQ20240728-181535.png)

> ​	通过结构的分析我们不难看出使用开散列比开放定址法的好处在于减少了因哈希冲突而导致的额外寻找，在某中意义上实现了存储位置与关键字的真正对应，但是开散列仍然需要扩容，也就是还需要负载因子，因为可能很多数据映射在同一个桶中导致一个链表过长，扩容的目的可以降低各个桶的长度，从而增加查找速度

* **代码实现**

```c++
//使用开散列来实现hash
namespace openlist{
    template<typename K, typename V>
    struct HashNode{
        HashNode(const pair<K, V>& data)
        :_data(data),
        _next(nullptr)
        {}

        pair<K, V> _data;
        shared_ptr<HashNode> _next;
    };

    template<typename K, typename V, typename hash = HashFunc<K>>
    class Hash{
    private:
        using Node = HashNode<K,V>;
    public:
        Hash():_n(0) { _table.resize(10); }
        bool insert(const pair<K, V>& kv);
        Node* find(const K& key); 
        bool erase(const K& key);
    private:
        vector<shared_ptr<Node>> _table;
        size_t _n = 0;              //记录插入的个数，可以用来计算平衡因子
    };
    template<typename K, typename V, typename hash>
    HashNode<K,V>* Hash<K, V, hash>::find(const K& key){
        hash hs;
        size_t sz = _table.size(), hashi = hs(key) % sz; 
        //当这个桶执行的元素历遍完了，如果还是没有找到就证明没有这个元素
        shared_ptr<Node> cur = _table[hashi];
        while(cur){
            if(cur->_data.first == key) return HashIterator(this, cur);
            else cur = (cur->_next);
        }
        return HashIterator(this, nullptr);
    }

    //这里扩容时有一个效率的问题，就是我们是像开放定址法一样新创建一个_table
    //然后逐渐进行插入吗，如果是这样的化，那么效率太低，所以可以重复利用原来创建好的指针
    template<typename K, typename V, typename hash>
    bool Hash<K,V,hash>::insert(const pair<K, V>& kv){
        HashIterator node = find(kv.first);
        if(node) ;
        hash hs;
        size_t old_sz = _table.size();
        //当桶中元素的数量和桶的数量相同时进行扩容操作
        if(_n == old_sz){
            size_t new_sz = old_sz*2;
            vector<shared_ptr<Node>> new_table;
            new_table.resize(new_sz);
            for(int i = 0; i < old_sz; ++i){
                //unique<Node>
                auto cur = _table[i];
                while(cur){
                    size_t hashi = hs(cur->_data.first) % new_sz;
                    auto next = (cur->_next);
                    shared_ptr<Node> ptr = new_table[hashi];
                    cur->_next = ptr;
                    new_table[hashi] = move(cur);
                    cur = next;
                }
            }
            _table.swap(new_table);
        } 
        //插入新节点
        size_t hashi = hs(kv.first) % _table.size();
        shared_ptr<Node> old_node = _table[hashi];
        shared_ptr<Node> new_node = make_shared<Node>(kv);
        new_node->_next = move(old_node);
        _table[hashi] = move(new_node);
        _n++;
    }

    template<typename K, typename V, typename hash>
    bool Hash<K,V,hash>::erase(const K& key){
        hash hs;
        size_t sz = _table.size(), hashi = hs(key)%sz;
        shared_ptr<Node> cur = _table[hashi];
        shared_ptr<Node> prev;
        while(cur){
            if(cur->_data.first == key){
                if(!prev) {
                    _table[hashi] = move(cur->_next);
                }
                else{
                    prev->_next = move(cur->_next);
                }
                return true;
            }
            prev = cur;
            cur = cur->_next;
        } 
        return false;
    }
}
```

