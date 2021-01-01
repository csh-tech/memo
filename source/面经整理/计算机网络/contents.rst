计算机网络
==========

.. code-block:: c
    :linenos:

    #ifndef AVL_TREE_H_
    #define AVL_TREE_H_

    #include <iostream>
    #include <queue>
    #include <cassert>
    #include <algorithm>

    template<typename Key, typename Value>
    class AVL {
    public:
      AVL() : root_(nullptr), count_(0) {}

      ~AVL() { Destroy(root_); }

      inline int Size() { return count_; }

      inline bool IsEmpty() { return count_ == 0; }

      void Insert(Key key, Value value) {
        root_ = Insert(root_, key, value);
      }

      bool Contain(Key key) { return Contain(root_, key); }

      // 思考：返回值应该是什么类型？
      // 如果设计成Node*的话，那么Node结构体必须为public，破坏了封装性
      // 如果涉及成Value的话，那么如何处理找不到的情形将是一个问题
      // 如果涉及成Value*的话，找不到时可以返回nullptr，找到的话，用户会得到一个
      // 指向target的指针，方便用户修改value
      Value* Search(Key key) { return Search(root_, key); }

      // 深度优先遍历  O(n)
      // 先根遍历
      void PreOrder() { PreOrder(root_); }

      // 中根遍历
      void InOrder() { InOrder(root_); }

      // 后根遍历
      void PostOrder() { PostOrder(root_); }

      // 广度优先遍历  O(n)
      // 层序遍历
      void LevelOrder() {
        std::queue<Node*> q;
        q.push(root_);
        while (!q.empty()) {
          Node* node = q.front();
          q.pop();
          std::cout << node->key << std::endl;
          if (node->left) q.push(node->left);
          if (node->right) q.push(node->right);
        }
      }

      // 寻找最小值
      Key Minimum() {
        assert(count_ != 0);
        Node* min_node = Minimum(root_);
        return min_node->key;
      }

      // 寻找最小值
      Key Maximum() {
        assert(count_ != 0);
        Node* max_node = Maximum(root_);
        return max_node->key;
      }

      // 从二叉搜索树中删除最小值所在节点
      void RemoveMin() { if (root_) root_ = RemoveMin(root_); }

      // 从二叉搜索树中删除最大值所在节点
      void RemoveMax() { if (root_) root_ = RemoveMax(root_); }

      // 从二叉搜索树中删除键值为key的节点
      void Remove(Key key) {
        root_ = Remove(root_, key);
      }

      // 判断当前的二叉树是否是一棵(升序)二分搜索树
      bool IsBST() {
        std::vector<Key> keys;
        InOrder(root_, keys);
        for (int i = 1; i < keys.size(); i++) {
          if (keys[i - 1] > keys[i]) return false;
        }
        return true;
      }

      // 判断当前的二叉树是否是一棵平衡二叉树
      bool IsBalanced() { return IsBalanced(root_); }

    private:
      struct Node {
        Key key;
        Value value;
        Node* left;
        Node* right;
        int height;  // 记录节点的高度值

        Node(Key key, Value value)
          : key(key), value(value), left(nullptr), right(nullptr), height(1) {}

        Node(Node* node)
          : key(node->key), value(node->value), left(node->left), right(node->right), height(1) {}
      };

      Node* root_;
      int count_;

      // 新加函数
      int GetHeight(Node* node) {
        if (node == nullptr) return 0;
        return node->height;
      }

      // 获得node节点的平衡因子
      int GetBanceFactor(Node* node) {
        if (node == nullptr) return 0;
        return GetHeight(node->left) - GetHeight(node->right);
      }

      bool IsBalanced(Node* node) {
        if (node == nullptr) return true;
    
        int balance_factor = GetBanceFactor(node);
        if (std::abs(balance_factor > 1)) return false;
        return IsBalanced(node->left) && IsBalanced(node->right);
      }

      // 向以node为根的二叉搜索树中插入节点(key, value)，返回插入新节点后的二叉搜索树的根
      // TODO: 非递归实现
      Node* Insert(Node* node, Key key, Value value) {
        if (node == nullptr) {
          count_++;
          return new Node(key, value);
        }
    
        if (node->key == key) {
          node->key = key;
        } else if (node->key > key) {
          node->left = Insert(node->left, key, value);
        } else {
          node->right = Insert(node->right, key, value);
        }

        // 维持平衡需要更新 height
        node->height = 1 + std::max(GetHeight(node->left), GetHeight(node->right));

        // 计算平衡因子
        int balance_factor = GetBanceFactor(node);
    
        // 处理不平衡情况

        // (1) LL情况: 右旋转
        //           y                           x
        //          / \                        /   \
        //         x  T4      右旋            z     y
        //        / \      ----------->     /  \   / \
        //       z  T3                    T1   T2 T3 T4
        //      / \
        //    T1  T2
        if (balance_factor > 1 && GetBanceFactor(node->left) >= 0) {
          return RightRotate(node);
        }

        // (2) RR情况: 左旋转
        //       y                            x
        //      / \                         /   \
        //    T1   x         左旋          y     z
        //        / \    ----------->    /  \   / \
        //      T2   z                 T1   T2 T3 T4
        //          / \
        //        T3  T4
        if (balance_factor < -1 && GetBanceFactor(node->right) <= 0) {
          return LeftRotate(node);
        }

        // (3) LR情况: 先左旋转后右旋转
        //       y                        y                      z
        //      / \                     /  \                   /   \
        //     x  T4    左旋           z    T4    右旋        x      y
        //    / \     ------->       /  \       ------->    / \    / \
        //  T1   z                  x   T3                T1  T2  T3 T4
        //      / \                / \
        //     T2  T3            T1  T2
        if (balance_factor > 1 && GetBanceFactor(node->right) < 0) {
          node->left = LeftRotate(node->left);
          return RightRotate(node);
        }

        // (4) RL情况: 先右旋转后左旋转
        //       y                      y                         z
        //      / \                   /  \                      /   \
        //    T1   x      右旋      T1    z        左旋        y      x
        //        / \   ------->        /  \     ------->    / \    / \
        //       z  T4                T2    x              T1  T2  T3 T4
        //     / \                         / \
        //   T2  T3                      T3  T4
        if (balance_factor < -1 && GetBanceFactor(node->right) > 0) {
          node->right = RightRotate(node->right);
          return LeftRotate(node);
        }
    
        // 其他情况下(一定要注意不止上述几种情况)树仍然是平衡的，所以不需要处理，直接返回即可
        // 上述写法的 if 代码段是极好的，不要写成 if (...) {...} else if (...) {...} else {...}
        return node;  // don't forget
      }

      // 对节点 y 进行右旋转操作，返回旋转后的新的根节点 x
      //           y                           x
      //          / \                        /   \
      //         x  T4      右旋            z     y
      //        / \      ----------->     /  \   / \
      //       z  T3                    T1   T2 T3 T4
      //      / \
      //    T1  T2
      // 假设 T1 和 T2 二者中最大的高度值为 h，那么：
      // z 的高度值为: h + 1
      // T3 的高度值为: h + 1 或者 h
      // x 的高度值为: h + 1
      // T4 的高度值为: h
      // 所以，右侧的二叉树是一棵平衡二叉树并且是一棵二分搜索树
      Node* RightRotate(Node* y) {
        Node* x = y->left;
        Node* T3 = x->right;
    
        // 向右旋转
        x->right = y;
        y->left = T3;

        // 更新 height，只有 x 和 y 的高度值发生了改变，要先更新 y 的高度值，因为 y
        // 高度值对 x 的高度值有影响
        y->height = 1 + std::max(GetHeight(y->left), GetHeight(y->right));
        x->height = 1 + std::max(GetHeight(x->left), GetHeight(x->right));

        // 返回旋转过的树的根节点
        return x;
      }

      // 对节点 y 进行左旋转操作，返回旋转后的新的根节点 x
      //       y                            x
      //      / \                         /   \
      //    T1   x         左旋          y     z
      //        / \    ----------->    /  \   / \
      //      T2   z                 T1   T2 T3 T4
      //          / \
      //        T3  T4
      Node* LeftRotate(Node* y) {
        Node* x = y->right;
        Node* T2 = x->left;
    
        // 向左旋转
        x->left = y;
        y->right = T2;

        // 更新高度值
        y->height = 1 + std::max(GetHeight(y->left), GetHeight(y->right));
        x->height = 1 + std::max(GetHeight(x->left), GetHeight(x->right));
    
        // 返回旋转后的树的根节点
        return x;
      }

      // 查找以node为根的二叉搜索树中是否包含键值为key的节点
      bool Contain(Node* node, Key key) {
        if (node == nullptr) {
          return false;
        }
    
        if (node->key == key) {
          return true;
        } else if (node->key > key) {
          return Contain(node->left, key);
        } else {
          return Contain(node->right, key);
        }
      }

      // 以node为根的二叉搜索树中查找键值为key的节点
      Value* Search(Node* node, Key key) {
        if (node == nullptr) return nullptr;
    
        if (node->key == key) {
          return &(node->value);
        } else if (node->key > key) {
          return Search(node->left, key);
        } else {
          return Search(node->right, key);
        }
      }

      // TODO: 重构遍历接口
      // 增加一个函数指针类型的参数代替cout，例如Delete函数就可以直接调用后根遍历，并在调
      // 用的同时传入相应的delete操作
      // 对以node为根的二叉搜索树进行先根遍历
      void PreOrder(Node* node) {
        if (node != nullptr) {
          std::cout << node->key << std::endl;
          PreOrder(node->left);
          PreOrder(node->right);
        }
      }

      // 对以node为根的二叉搜索树进行中根遍历
      void InOrder(Node* node) {
        if (node != nullptr) {
          InOrder(node->left);
          std::cout << node->key << std::endl;
          InOrder(node->right);
        }
      }

      // for IsBST check
      void InOrder(Node* node, std::vector<Key>& keys) {
        if (node != nullptr) {
          InOrder(node->left, keys);
          keys.push_back(node->key);
          InOrder(node->right, keys);
        }
      }

      // 对以node为根的二叉搜索树进行后根遍历
      void PostOrder(Node* node) {
        if (node != nullptr) {
          PostOrder(node->left);
          PostOrder(node->right);
          std::cout << node->key << std::endl;
        }
      }

      void Destroy(Node* node) {
        if (node != nullptr) {
          Destroy(node->left);
          Destroy(node->right);
          delete node;
          count_--;
        }
      }

      // 返回以node为根的二叉搜索树的最小键值得节点
      // TODO: 非递归实现
      Node* Minimum(Node* node) {
        if (node->left == nullptr) {
          return node;
        }
        return Minimum(node->left);
      }

      // 返回以node为根的二叉搜索树的最大键值得节点
      // TODO: 非递归实现
      Node* Maximum(Node* node) {
        if (node->right == nullptr) {
          return node;
        }
        return Maximum(node->rihgt);
      }

      // 删除以node为根的二分搜索树中的最小节点
      // 返回删除节点后新的二分搜索树的根
      // TODO: 非递归实现
      Node* RemoveMin(Node* node) {
        if (node->left == nullptr) {  // 表示node节点就是当前的最小节点，将其删除
          Node* right_node = node->right;  // 右孩子为空时也ok
          delete node;
          count_--;
          return right_node;
        }

        node->left = RemoveMin(node->left);
        return node;
      }

      // 删除以node为根的二分搜索树中的最小节点
      // 返回删除节点后新的二分搜索树的根
      // TODO: 非递归实现
      Node* RemoveMax(Node* node) {
        if (node->right == nullptr) {  // 表示node节点就是当前的最大节点，将其删除
          Node* left_node = node->left;  // 左孩子为空时也ok
          delete node;
          count_--;
          return left_node;
        }

        node->right = RemoveMax(node->right);
        return node;
      }

      // 删除以node为根的二叉搜索树中的键值为key的节点
      // 返回删除节点后的新的二分搜索树的根  O(log(n))，主要耗时在找到目标节点
      Node* Remove(Node* node, Key key) {
        if (node == nullptr) return nullptr;
    
        Node* ret_node;
        if (key < node->key) {
          node->left = Remove(node->left, key);
          ret_node = node;
        } else if (key > node->key) {
          node->right = Remove(node->right, key);
          ret_node = node;
        } else {  // key == node->key
          if (node->left == nullptr) {
            Node* right_node = node->right;
            delete node;
            count_--;
            ret_node = right_node;
          } else if (node->right == nullptr) {
            Node* left_node = node->left;
            delete node;
            count_--;
            ret_node = left_node;
          } else {
            // node->left != nullptr && node->right != nullptr
            // 陷阱，要复制一次最小节点，因为RemoveMin会把这个最小节点删除
            Node* successor = new Node(Minimum(node->right));
            count_++;
            // successor->right = RemoveMin(node->rihgt);  // RemoveMin内会发生一次count_--
            // 处理AVL树时，上一行需要改写成如下形式
            // 因为RemoveMin操作没有添加维护平衡的功能，可能会打破树的平衡特性
            // 而Remove可以维护平衡，可以直接调用
            // Question: 下面这句会不会有问题，递归能不能跳出去？
            // 可以跳出去，最开始设有出口
            successor->right = Remove(node->rihgt, successor->key);
            successor->left = node->left;
            delete node;
            count_--;
            ret_node = successor;
          }
        }

        // 当删除的是叶子节点时，ret_node可能会出现为空的情况，此时，若不处理的话，后续的
        // ret_node->height操作就会报错
        if (ret_node == nullptr) return nullptr;  

        // 维持平衡需要更新 height
        ret_node->height = 1 + std::max(GetHeight(ret_node->left), GetHeight(ret_node->right));

        // 计算平衡因子
        int balance_factor = GetBanceFactor(ret_node);
    
        // 处理不平衡情况

        // (1) LL情况: 右旋转
        if (balance_factor > 1 && GetBanceFactor(ret_node->left) >= 0) {
          return RightRotate(ret_node);
        }

        // (2) RR情况: 左旋转
        if (balance_factor < -1 && GetBanceFactor(ret_node->right) <= 0) {
          return LeftRotate(ret_node);
        }

        // (3) LR情况: 先左旋转后右旋转
        if (balance_factor > 1 && GetBanceFactor(ret_node->right) < 0) {
          ret_node->left = LeftRotate(ret_node->left);
          return RightRotate(ret_node);
        }

        // (4) RL情况: 先右旋转后左旋转
        if (balance_factor < -1 && GetBanceFactor(ret_node->right) > 0) {
          ret_node->right = RightRotate(ret_node->right);
          return LeftRotate(ret_node);
        }
    
        return ret_node;  // don't forget
      }

      // TODO: 采用第二种策略，用节点的左子树的最大值代替节点，从而达到删除的目的

    };

    #endif  // AVL_TREE_H_