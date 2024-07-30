# BitMap and BloomFilter

* BitMap 和 BloomFilter都是对哈希思维的一种运用，它们在处理海量数据时有它们各自的优势

## 位图

* 所谓位图就是使用每一位来存放某种状态，适用于海量数据，数据无重复的场景。通常使用来判断某个数据是否存在

### 位图的实现

```c++
constexpr size_t RANGE = 32;

template<size_t N>
class BitMap{
public:
    BitMap(){
        _bit_map.resize(N/RANGE+1);
    }

    //将x对应的位设置为0
    void set(size_t x){
        //找出x是在第i个int中的第j位
        int i = x / RANGE;
        int j = x % RANGE;

        _bit_map[i] |= (1<<j); 
    }

    void unset(size_t x){
        //将x对应的位致0        
        int i = x / RANGE;
        int j = x % RANGE;

        _bit_map[i] &= ~(1<<j); 
    }

    bool test(size_t x){
        //判断x所在的位是不是1
        int i = x / RANGE;
        int j = x % RANGE;

        return _bit_map[i] & (1<<j);
    }

private:
    vector<int> _bit_map;
};
```

### 位图的应用

1. 快速查找某个数据是否还在一个集合中

2. 排序 + 去重

3. 求两个集合的交集                                     

   ->  如果使判断在不在用一个位图就可以了，但判断出现次数可以使用两个位图  

   00表示没出现    01表示出现1次  10表示出现两次  11表示出现3次及以上

4. 操作系统中磁盘块的标记

* 位图的缺点 -> 只能存储整型
* 位图的优点 -> 效率高，使用的空间较少

## 布隆过滤器

* 布隆过滤器是由布隆（Burton Howard Bloom）在1970年提出的 一种紧凑型的、比较巧妙的概 率型数据结构，特点是高效地插入和查询，可以用来告诉你 “**某样东西一定不存在或者可能存在**”，它是**用多个哈希函数，将一个数据映射到位图结构中**。此种方式不仅可以提升查询效率，也 可以节省大量的内存空间  -> 总的来说，布隆过滤器就是一个哈希与位图的结合，可以用来判断非整型一定不存在或者可能存在，至于为什么我们了解了布隆过滤器就知道了

### 什么是布隆过滤器

* 布隆过滤器是一个位图结构，如：

  ![QQ20240728-184726](C:/Users/Administrator/Desktop/QQ20240728-184726.png)

然后根据多个哈希函数对要存储的值进行计算，然后得到多个哈希值，再将对应位置的值置1

eg:

> ​	如我想存储百度，就用三个哈希函数计算出3个哈希值，然后将其置1，再插入Tencent，在进行计算和置1

![QQ20240728-185027](C:/Users/Administrator/Desktop/QQ20240728-185027.png)

这时在进行查询时就将要查询的值同样进行计算然后判断是不是所有的结果都为1，**如果是那么就可能存在（因为可能你想存的值也是其他几个数据想存储的，这样就会导致误判），如果有一个不为1就一定不存在**

### 实现

```c++
struct BKDRHash

{
 size_t operator()(const string& s)
 {
 // BKDR

 size_t value = 0;
 for (auto ch : s)
 {
 value *= 31;
 value += ch;
 }
 return value;
 }
};

struct APHash

{
 size_t operator()(const string& s)
 {
 size_t hash = 0;
 for (long i = 0; i < s.size(); i++)
 {
 if ((i & 1) == 0)
 {
 hash ^= ((hash << 7) ^ s[i] ^ (hash >> 3));
 }
 else

 {
 hash ^= (~((hash << 11) ^ s[i] ^ (hash >> 5)));
 }
 }
 return hash;
 }
};

struct DJBHash
{
 size_t operator()(const string& s)
 {
 size_t hash = 5381;
 for (auto ch : s)
 {
 hash += (hash << 5) + ch;
 }
 return hash;
 }
};

template<size_t N,

size_t X = 5,

class K = string,

class HashFunc1 = BKDRHash,

class HashFunc2 = APHash,

class HashFunc3 = DJBHash>

class BloomFilter

{

public:
 void Set(const K& key)
 {
 	size_t len = X*N;
 	size_t index1 = HashFunc1()(key) % len;
 	size_t index2 = HashFunc2()(key) % len;
 	size_t index3 = HashFunc3()(key) % len;
 	/* cout << index1 << endl;

 	cout << index2 << endl;

 	cout << index3 << endl<<endl;*/
	
 	_bs.set(index1);
 	_bs.set(index2);
 	_bs.set(index3);
 }
 bool Test(const K& key)
 {
 	size_t len = X*N;
 	size_t index1 = HashFunc1()(key) % len;
 	if (_bs.test(index1) == false)
 		return false;
 	size_t index2 = HashFunc2()(key) % len;
 	if (_bs.test(index2) == false)
 		return false;
 	size_t index3 = HashFunc3()(key) % len;
 	if (_bs.test(index3) == false)
 		return false;
 	return true;  // 存在误判的

 }
 // 不支持删除，删除可能会影响其他值。

 void Reset(const K& key);
 private:
 	bitset<X*N> _bs;
};   
```

### 布隆过滤器的查找

> ​	布隆过滤器的思想是将一个元素用多个哈希函数映射到一个位图中，因此被映射到的位置的比特 位一定为1。所以可以按照以下方式进行查找：**分别计算每个哈希值对应的比特位置存储的是否为 零，只要有一个为零，代表该元素一定不在哈希表中，否则可能在哈希表中。**
>
> 注意：布隆过滤器如果说某个元素不存在时，该元素一定不存在，如果该元素存在时，该元素可 能存在，因为有些哈希函数存在一定的误判。
>
> 比如：在布隆过滤器中查找"alibaba"时，假设3个哈希函数计算的哈希值为：1、3、7，刚好和其 他元素的比特位重叠，此时布隆过滤器告诉该元素存在，但实该元素是不存在的。

### 布隆过滤器的删除

> 布隆过滤器不能直接支持删除工作，因为在删除一个元素时，可能会影响其他元素。
>
> 比如：删除上图中"tencent"元素，如果直接将该元素所对应的二进制比特位置0，“baidu”元素也 被删除了，因为这两个元素在多个哈希函数计算出的比特位上刚好有重叠。 

#### 优点

> 1. 增加和查询元素的时间复杂度为:O(K), (K为哈希函数的个数，一般比较小)，与数据量大小无 关
> 2. 哈希函数相互之间没有关系，方便硬件并行运算
> 3. 布隆过滤器不需要存储元素本身，在某些对保密要求比较严格的场合有很大优势
> 4. 在能够承受一定的误判时，布隆过滤器比其他数据结构有这很大的空间优势
> 5. 数据量很大时，布隆过滤器可以表示全集，其他数据结构不能
> 6. 使用同一组散列函数的布隆过滤器可以进行交、并、差运算

#### 不足

> 1. 有误判率，即存在假阳性(False Position)，即不能准确判断元素是否在集合中(补救方法：再 建立一个白名单，存储可能会误判的数据) 2. 不能获取元素本身
> 2. 一般情况下不能从布隆过滤器中删除元素