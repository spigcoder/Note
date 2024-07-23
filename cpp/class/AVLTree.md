# AVLTree

* AVLTree是一种特殊的搜索二叉树（左子树的值小于root，右子树的值大于root），AVLTree解决了普通的搜索二叉树可能退化为单支树导致的搜索元素效率低下的问题

> AVLTree: 他的左右子树都是AVLTree，左右子树的高度差（平衡因子）的绝对值不超过1（1、0、-1）在本博客中以右子树的高度减左子树的高度作为平衡因子



### AVLTree实现

* AVLTree通过使用平衡因子的方法来实现左右高度差不超过1，如果插入新节点之后AVLTree的左右高度差超过了1，就要对这个AVLTree进行旋转处理

##### AVLTree节点的定义

```c++
template<typename K, typename V>
struct AVLNode{
    AVLNode(pair<K, V> kv = {K(), V()})
    :_kv(kv),
    _bt(0), _left(nullptr),
    _right(nullptr),_parent(nullptr)
    {}

   pair<K, V> _kv;
   int _bt; //balance factor
   AVLNode<K, V>* _left; 
   AVLNode<K, V>* _right; 
   AVLNode<K, V>* _parent; 
};
```

##### AVLTree的插入

​	AVLTree就是在普通的二叉搜索树中加入平衡因子，所以AVLTree也可以看作是一个二叉搜索树，所以AVLTree的插入可以分为两步

> 1. 通过二叉搜索树的性质找到要插入的位置
>
> 2. 根据平衡因子的改变对树进行调整
>
>    > 注：由于我们定义的平衡因子是右子树的高度减左子树的高度，所以如果插入的新的节点在parent的右边的化parent->bt++ 否则 parent->bt--

* 完整代码

```c++
template<typename K, typename V>
    bool AVLTree<K, V>::insert(const pair<K, V>& kv){
/////////////////////////////////////////////////////////////////////////////////////////////
/*找到要插入的位置并将节点插入*/
        if(_root == nullptr){
            _root = new Node(kv);
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
/////////////////////////////////////////////////////////////////////////////////////////////
/*调整平衡因子并根据平衡因子进行旋转操作*/
        while(parent){
            //renewal balance factor
            if(node == parent->_left) parent->_bt--;
            else parent->_bt++;
            if(parent->_bt == 0) break;
///////////////////////////////////////////////////////////////////////////////////////////////*旋转操作*/
            if(parent->_bt == 2 || parent->_bt == -2){
                if(parent->_bt == 2 && node->_bt == 1){
                    RotateL(parent);
                }else if(parent->_bt == -2 && node->_bt == -1){
                    RotateR(parent);
                }else if(parent->_bt == 2 && node->_bt == -1){
                    Node* SubR = parent->_right; 
                    Node* SubRL = SubR->_left;
                    int bt = SubRL->_bt;
                    RotateRL(parent);
                    //renewal balance factor
                    if(bt == 0){
                        SubR->_bt = parent->_bt = 0;
                    }else if(bt == 1){
                        parent->_bt = -1;
                        SubR->_bt = SubRL->_bt = 0;
                    }else if(bt == -1){
                        SubRL->_bt = parent->_bt = 0;
                        SubR->_bt = 1;
                    }

                }else if(parent->_bt == -2 && node->_bt == 1){
                    Node* SubL = parent->_left;
                    Node* SubLR = SubL->_right;
                    int bt = SubLR->_bt;
                    RotateLR(parent);
                    //renewal balance factor
                    if(bt == 0){
                        SubL->_bt = parent->_bt = 0;
                    }else if(bt == 1){
                        SubL->_bt = -1;
                        SubLR->_bt = parent->_bt = 0;
                    }else if(bt == -1){
                        SubL->_bt = SubLR->_bt = 0;
                        parent->_bt = 1;
                    }
                }
                break;
            }
            node = parent;
            parent = parent->_parent;
        }
    }
```

##### AVLTree平衡因子的更新

当我们插入节点时并不是每次都要进行旋转的，要看我们插入的位置以及对于整个树高度的影响

> 	1. 如果插入之后parent-bt == 0证明这个子树高度不变，直接可以退出
> 	1. 如果插入之后parent->bt == 2/-2那么就要根据情况来进行合适的旋转
> 	1. 否则就要继续向上更新，将node = parent; parent = node->_parent

##### AVLTree的旋转

​	如果在一个原本是平衡的AVL树中插入一个新节点，可能造成不平衡，此时必须要调整树的结构，使之平衡化。根据节点插入位置的不同，AVL的旋转有四种

* 1. 新插入的节点在较高左子树的左侧 -- 左左： 右单旋

     

     > ![QQ_1721658915557](C:/Users/ADMINI~1/AppData/Local/Temp/QQ_1721658915557.png)
     >
     > 这种情况是较为简单的，我们可以设置30为SubL， 60为parent， 30的右节点为SubLR，只要将SubL->right = parent, parent->right = SubLR, 同时更新他们的父节点就可以了（这里要注意保存parent的父节点，并判断是否为nullptr如果是nullptr，那就要让SubL变成root）
     >
     > * 代码
     >
     > ```c++
     > template<typename K, typename V>
     >     void AVLTree<K, V>::RotateR(Node* parent){
     >         Node* SubL = parent->_left;
     >         Node* SubLR = SubL->_right;
     >         Node* Parent_parent = parent->_parent;
     > 
     >         //link
     >         SubL->_right = parent;
     >         parent->_parent = SubL;
     >         parent->_left = SubLR;
     >         if(SubLR) SubLR->_parent = parent;
     >         if(Parent_parent == nullptr){
     >             _root = SubL;
     >         }else{
     >             if(Parent_parent->_left == parent){
     >                 Parent_parent->_left = SubL;
     >             }else{
     >                 Parent_parent->_right = SubL;
     >             }
     >         }
     >         SubL->_parent = Parent_parent;
     > 
     >         //renewal balance factor
     >         parent->_bt = SubL->_bt = 0;
     >     }
     > ```

* 2. 新插入的节点在较高右子树的右侧 -- 右右：左单选

     >![QQ20240722-224155](C:/Users/Administrator/Desktop/QQ20240722-224155.png)
     >
     >具体内容与左左单旋一致，直接上代码
     >
     >```c++
     >    template<typename K, typename V>
     >    void AVLTree<K, V>::RotateL(Node* parent){
     >        Node* SubR = parent->_right;
     >        Node* SubRL = SubR->_left;
     >        Node* Parent_parent = parent->_parent;
     >
     >        //link
     >        SubR->_left = parent;
     >        parent->_parent = SubR;
     >        parent->_right = SubRL;
     >        if(SubRL) SubRL->_parent = parent; 
     >        if(Parent_parent == nullptr){
     >            _root = SubR;
     >        }else{
     >            if(parent == Parent_parent->_left){
     >                Parent_parent->_left = SubR;
     >            }else{
     >                Parent_parent->_right = SubR;
     >            }
     >        }
     >        SubR->_parent = Parent_parent;
     >
     >        //renewal balance factor
     >        parent->_bt = SubR->_bt = 0;
     >    }
     >```

* 3. 新节点插入较高右子树的左侧---右左：先右单旋再左单旋

     > ![QQ20240722-224453](C:/Users/Administrator/Desktop/QQ20240722-224453.png)
     >
     > 由于这种情况较为复杂，书面并不能很好的表达清楚，所以仅仅把图和代码贴给大家，大家可以自己根据左旋和右旋的概念自己画一遍，并根据代码写一遍可以更好的理解就是先对90进行右单旋，再对30进行左单旋，这个的难点不是代码，而是对平衡因子的掌控，因为插入在b中还是在c中对30和90的平衡因子会造成影响，所以请大家结合这个图自行画图思考
     >
     > ```c++
     > //旋转代码
     > template<typename K, typename V>
     >     void AVLTree<K, V>::RotateRL(Node* parent){
     >         Node* SubR = parent->_right;
     >         Node* SubRL = SubR->_left;
     >         
     >         RotateR(SubR);
     >         RotateL(parent);
     >     }
     > ```
     >
     > ```c++
     > 			//平衡因子更新代码
     > 			else if(parent->_bt == 2 && node->_bt == -1){
     >                     Node* SubR = parent->_right; 
     >                     Node* SubRL = SubR->_left;
     >                     int bt = SubRL->_bt;
     >                     RotateRL(parent);
     >                     //renewal balance factor
     >                     if(bt == 0){
     >                         SubR->_bt = parent->_bt = 0;
     >                     }else if(bt == 1){
     >                         parent->_bt = -1;
     >                         SubR->_bt = SubRL->_bt = 0;
     >                     }else if(bt == -1){
     >                         SubRL->_bt = parent->_bt = 0;
     >                         SubR->_bt = 1;
     >                     }
     > ```
     >
     > 

* 4. 新节点插入较高左子树的右侧---左右：先左单旋再右单旋

     > ![QQ20240722-225132](C:/Users/Administrator/Desktop/QQ20240722-225132.png)
     >
     > * 代码
     >
     > ```c++
     > 			//更新平衡因子
     > 			else if(parent->_bt == -2 && node->_bt == 1){
     >                     Node* SubL = parent->_left;
     >                     Node* SubLR = SubL->_right;
     >                     int bt = SubLR->_bt;
     >                     RotateLR(parent);
     >                     //renewal balance factor
     >                     if(bt == 0){
     >                         SubL->_bt = parent->_bt = 0;
     >                     }else if(bt == 1){
     >                         SubL->_bt = -1;
     >                         SubLR->_bt = parent->_bt = 0;
     >                     }else if(bt == -1){
     >                         SubL->_bt = SubLR->_bt = 0;
     >                         parent->_bt = 1;
     >                     }
     > ```
     >
     > ```c++
     >     //旋转
     >     template<typename K, typename V>
     >     void AVLTree<K, V>::RotateLR(Node* parent){
     >         Node* SubL = parent->_left;
     >         Node* SubLR = SubL->_right;
     >         
     >         RotateL(SubL);
     >         RotateR(parent);
     >     }
     > ```
     >
     > 



### 平衡二叉树的验证

1. 验证其为二叉树 (中序得到一个有序序列)
2. 验证其为平衡树 (左右子树高度差不超过1)

* 代码

```c++
    template<typename K, typename V>
    int AVLTree<K, V>::_Height(Node* root){
        if(root == nullptr) return 0;

        int left_height = _Height(root->_left);
        int right_height = _Height(root->_right);

        return left_height > right_height ? left_height+1 : right_height+1;
    }

    template<typename K, typename V>
    bool AVLTree<K, V>::_IsBalance(Node* root){
        if(root == nullptr) return true;

        int left_height = _Height(root->_left);
        int right_height = _Height(root->_right); 

        //这里是判断错误的情况，如果正确，还要递归判断其左右子树
        if(abs(left_height - right_height) >= 2 || abs(root->_bt) >= 2)
            return false;

        bool left_is_balance = _IsBalance(root->_left);
        bool right_is_balance = _IsBalance(root->_right);

        return left_is_balance && right_is_balance;
    }

```



### 总结

>总结： 假如以pParent为根的子树不平衡，即pParent的平衡因子为2或者-2，分以下情况考虑
>
>1. pParent的平衡因子为2，说明pParent的右子树高，设pParent的右子树的根为pSubR
>
>当pSubR的平衡因子为1时，执行左单旋 当pSubR的平衡因子为-1时，执行右左双旋
>
>2. pParent的平衡因子为-2，说明pParent的左子树高，设pParent的左子树的根为pSubL
>
>当pSubL的平衡因子为-1是，执行右单旋 当pSubL的平衡因子为1时，执行左右双旋 旋转完成后，原pParent为根的子树个高度降低，已经平衡，不需要再向上更新。