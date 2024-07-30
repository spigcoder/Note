# RBTree



## 红黑树概念

* 红黑树是一种平衡二叉搜索树，但在每个节点上增加一个存储位表示节点的颜色，可以是Red或则Black。通过对一条从根到叶子的路径上各个节点着色方式的限制，红黑树确保没有一条路径会比其他的路径长两倍，因而是接近平衡的。

### 红黑树的性质

> 	1. 每个节点不是黑色就是红色
> 	1. 根节点是黑色的
> 	1. 红色节点的孩子必须是黑色的
> 	1. 每一条路径下的黑色节点数量相同

根据最后两条性质就可以确定红黑树最长路径不会超过最短路径的两倍。

### 红黑树节点的定义

```c++
enum Colour{
    RED,
    BLACK
};

template<typename K, typename V>
struct RBNode{
    RBNode(pair<K, V> kv = {K(), V()})
    :_kv(kv),
    _col(RED),
     _left(nullptr),
    _right(nullptr),
    _parent(nullptr)
    {}

   pair<K, V> _kv;
   Colour _col;
   RBNode<K, V>* _left; 
   RBNode<K, V>* _right; 
   RBNode<K, V>* _parent; 
};
```

同样的三叉链结构，只是将AVLTree的bt变成col

* 这里新插入的节点的默认颜色是红色 -> 因为如果是黑色，那么每插入一个新的节点都会破坏**性质4**，但是如果新插入的节点是红色节点则有可能会破坏**性质3**，有可能不破坏

## 红黑树的插入操作

红黑树的插入操作主要分为两步：

> 1. 根据二叉搜索树的性质找到要插入的节点并插入
>
> 2. 检测插入新节点之后，红黑树的性质是否遭到破坏，这里如果新插入节点的父亲是黑色节点就不破坏任何性质，如果父亲是红色节点就破坏**性质3**需要根据情况进行旋转或变色。
>
> 
>
> ​	注：红黑树是否发生旋转主要与叔叔的颜色以及是否存在有关

* 1. **cur为红，p为红，g为黑，u存在且为红**![QQ20240723-205757](C:/Users/Administrator/Desktop/QQ20240723-205757.png)

> ​	这时不需要进行旋转，只要将 p->col = black    u->col = black  g->col = red
>
> ​	然后让cur=g  parent = cur->parent继续向上进行判断即可	

```c++
			if(uncle && uncle->_col == RED){
                    //更改颜色
                    parent->_col = uncle->_col = BLACK;
                    grandfather->_col = RED;
                    //向上更新
                    node = grandfather;
                    parent = node->_parent;
             }
```

* 2. cur为红，p为红，g为黑，u不存在/u存在且为黑

  这里主要有两种情况，如果是一边倒（参考AVLTree），就是单旋，如果是折线型的，就是双旋，具体情况可自行画图理解，现在给出参考代码![QQ20240723-210239](C:/Users/Administrator/Desktop/QQ20240723-210239.png)

​	

```c++
			else if(uncle == nullptr || uncle->_col == BLACK){
                    //需要进行旋转
                    if(node == parent->_left){
                        //          g
                        //      p       u
                        // n -> 进行右单旋
                        RotateR(grandfather);
                        parent->_col = BLACK;
                        grandfather->_col = RED;
                        break;
                    }else if(node == parent->_right){
                        //          g
                        //      p       u
                        //          n -> 进行左右双旋
                        RotateL(parent);
                        RotateR(grandfather);
                        node->_col = BLACK;
                        grandfather->_col = RED;
                        break;
                    }
```

### 完整插入代码

```c++
    template<typename K, typename V>
    bool RBTree<K, V>::insert(const pair<K, V>& kv){
        if(_root == nullptr){
            _root = new Node(kv);
            _root->_col = BLACK;
            return true;
        }
        //find position to insert
        Node *cur = _root, *parent = _root;
        while(cur){
            parent = cur;
            if(kv.first > cur->_kv.first){
                cur = cur->_right;
            }else if(kv.first < cur->_kv.first){
                cur = cur->_left;
            }else{
                return false;
            }
        }

        //link
        Node* node = new Node(kv);
        if(kv.first > parent->_kv.first){
            parent->_right = node;
        }else{
            parent->_left = node;
        }
        node->_parent = parent;

        while(parent && parent->_col == RED){
            Node* grandfather = parent->_parent;
            //      g
            //    p   u 
            if(parent == grandfather->_left){
                Node* uncle = grandfather->_right;
                //uncle 存在且是红色
                if(uncle && uncle->_col == RED){
                    //更改颜色
                    parent->_col = uncle->_col = BLACK;
                    grandfather->_col = RED;
                    //向上更新
                    node = grandfather;
                    parent = node->_parent;
                }else if(uncle == nullptr || uncle->_col == BLACK){
                    //需要进行旋转
                    if(node == parent->_left){
                        //          g
                        //      p       u
                        // n -> 进行右单旋
                        RotateR(grandfather);
                        parent->_col = BLACK;
                        grandfather->_col = RED;
                        break;
                    }else if(node == parent->_right){
                        //          g
                        //      p       u
                        //          n -> 进行左右双旋
                        RotateL(parent);
                        RotateR(grandfather);
                        node->_col = BLACK;
                        grandfather->_col = RED;
                        break;
                    }else{
                        assert(false);
                    }
                }else{
                    assert(false);
                }
            }else{
                Node* uncle = grandfather->_left;
                if(uncle && uncle->_col == RED){
                    uncle->_col = parent->_col = BLACK;
                    grandfather->_col = RED;
                    node = grandfather;
                    parent = node->_parent;
                }else if(uncle == nullptr || uncle->_col == BLACK){
                    if(node == parent->_right){
                        //          g
                        //      u       p
                        //                  n -> 进行左单旋
                        RotateL(grandfather);
                        parent->_col = BLACK;
                        grandfather->_col = RED;
                        break;
                    }else if(node == parent->_left){
                        //          g
                        //      u       p
                        //           n -> 进行右左单旋
                        RotateR(parent);
                        RotateL(grandfather);
                        node->_col = BLACK;
                        grandfather->_col = RED;
                        break;
                    }else {
                        assert(false);
                    }
                }else{
                    assert(false);
                }
            }
        }

        _root->_col = BLACK;
    }
```

 