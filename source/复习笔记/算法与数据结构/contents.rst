==============
算法与数据结构
==============

Array
=======

.. code-block:: c++
    :linenos:
    
    #ifndef ARRAY_H_
    #define ARRAY_H_

    #include <stdexcept>
    #include <string>

    template <typename T>
    class Array {
    public:
      // 默认构造函数，默认容量为10
      Array() {
        data_ = new T[10];
        size_ = 0;
        capacity_ = 10;
      }

      // 构造函数
      Array(int capacity) {
        data_ = new T[capacity];
        size_ = 0;
        capacity_ = capacity;
      }

      // 析构函数
      ~Array() { delete[] data_; }

      // 获取数组中的元素个数
      inline int GetSize() { return size_; }

      // 获取数组的容量
      inline int GetCapacity() { return capacity_; }

      // 判空
      inline bool IsEmpty() { return (size_ == 0); }

      // 向指定位置添加元素  O(n)
      void Add(int index, T e) {
        if (index < 0 || index > size_) {
          throw std::invalid_argument("Add failed, required index > 0 and index <= size_");
        }

        if (size_ == capacity_){
          Resize(2 * capacity_);
        }

        for (int i = size_ - 1; i >= index; i--) {
          data_[i + 1] = data_[i];
        }

        data_[index] = e;
        size_++;
      }

      // 向最后的位置添加元素  O(1)
      void AddLast(T e) { Add(size_, e); }

      // 在头部添加元素  O(n)
      void AddFirst(T e) { Add(0, e); }

      // to string
      std::string ToString() {
        std::string res;
        res += "Array: size = " + std::to_string(size_) + ", capacity = "
              + std::to_string(capacity_) + "\n[" ;
        for (int i = 0; i < size_; i++) {
          res += std::to_string(data_[i]);
          if (i != size_ - 1) {
            res += ", ";
          }
        }
        res += "]\n";
        return res;
      }

      // 获取index索引位置的元素  O(1)
      int Get(int index) {
        if (index < 0 || index >= size_){
          throw std::invalid_argument("Get failed, index is illegal.");
        }
        return data_[index];
      }

      int GetFirst() { return Get(0); }

      int GetLast() { return Get(size_ - 1); }

      // 修改index位置的元素为e  O(1)
      void Set(int index, T e) {
        if (index < 0 || index >= size_){
          throw std::invalid_argument("Get failed, index is illegal.");
        }
        data_[index] = e;
      }

      // 查找数组中是否存在元素e  O(n)
      bool Contain(T e) {
        for (int i = 0; i < size_; i++) {
          if (data_[i] == e) {
            return true;
          }
        }
        return false;
      }

      // 查找数组中元素e所在的索引，若不存在元素e，则返回-1  O(n)
      int Find(T e) {
        for (int i = 0; i < size_; i++) {
          if (data_[i] == e) {
            return i;
          }
        }
        return -1;
      }

      // 从数组中删除index位置的元素，并返回删除的元素  O(n)
      T Remove(int index) {
        if (index < 0 || index >= size_){
          throw std::invalid_argument("Remove failed, index is illegal.");
        }
        T ret = data_[index];
        for (int i = index + 1; i < size_; i++) {
          data_[i - 1] = data_[i];
        }
        size_--;
        // 优先级 "/", "*"  大于  "==" , "!="  大于  "&&"
        if (size_ == capacity_ / 4 && capacity_ / 2 != 0) {  // lazy implementation 防止数组容量震荡
          Resize(capacity_ / 2);
        }
        return ret;
      }

      // 删除首元素  O(n)
      inline T RemoveFirst() { return Remove(0); }

      // 删除尾元素  O(1)
      inline T RemoveLast() { return Remove(size_ - 1); };

      // 删除数组中的元素e  O(n)
      void RemoveElement(T e) {
        int index = Find(e);
        if (index != -1) {
          Remove(index);
        }
      }
  
    private:
      // 改变数组容量  O(n)
      void Resize(int new_capacity) {
        T* new_data = new T[new_capacity];
        for (int i = 0; i < size_; i++) {
          new_data[i] = data_[i];
        }
        delete[] data_;
        data_ = new_data;
        capacity_ = new_capacity;
      }

      int size_;
      int capacity_;
      T* data_;
    };

    #endif  // ARRAY_H_

Linked List
============

.. code-block:: c++
    :linenos:

    #ifndef LINKED_LIST_H_
    #define LINKED_LIST_H_

    #include <cassert>
    #include <string>
    #include <iostream>

    using namespace std;

    // 栈

    // 只对链表的头结点进行增、删、查的话，时间复杂度为O(1)
    // 所以链表可以很容易用来实现栈，链表头作为栈顶

    // 用链表实现队列时由于队列要在两端都都进行操作，因此，需要给链表增加尾指针
    // 由于在链表的尾部进行删除操作并不容易，所以只能用链表的尾部作为队列的入队端，
    // 由于在链表头部进行插入和删除操作都很容易，因此可以用链表的头部作为队列的出对端。
    // 实现时采用不带头结点的链表，没有必要了，因为此处的操作不涉及中间节点的情况，不需要
    // 用头结点处理操作不统一的问题
    // 不适用头结点时要注意链表为空的情况，此时，头指针和尾指针都指向 nullptr
    template <typename T>
    class LinkedList {  // 这里用 struct 比较好，默认是 public 的
    private:
      struct Node {
        T val;
        Node* next;

        Node(T val, Node* next) : val(val), next(next) {}

        Node(T val) : Node(val, nullptr) {};
      };

      // Node* head_;  // 无头结点版本
      Node* dummy_head_;  // 虚拟头结点
      int size_;

      void Check(int k) { assert(k >= 0 && k <= size_); }  // 注意这里是 <= size_

    public:
      LinkedList() : dummy_head_(new Node(T())), size_(0) {}

      ~LinkedList() {
        Node* cur = dummy_head_;
        while (cur) {
         Node* del_node = cur;
         cur = cur->next;
         delete del_node;
        }
      }

      inline int GetSize() { return size_; }

      inline bool IsEmpty() { return size_ == 0; }

      // 在链表的第k(0-based)个位置添加新的元素(面试题中会用到，平时不常用)，有头结点版本
      void Add(int k, T val) {
        Check(k);

        Node* pre = dummy_head_;
        for (int i = 0; i < k; i++) {  // 这里需要理解下
         pre = pre->next;
        }

        // Node* node = new Node(val);
        // node->next = pre->next;
        // pre->next = node;
 
        // 更优雅的写法
        pre->next = new Node(val, pre->next);
        size_++;
      }

      // 在链表头添加节点，有头结点版本
      void AddFirst(T val) { Add(0, val); }

      // 在链表末尾添加节点
      void AddLast(T val) { Add(size_, val); }

      // 获得链表的第k个元素
      T Get(int k) {
        Check(k);

        Node* cur = dummy_head_->next;
        for (int i = 0; i < k; i++) {
         cur = cur->next;
        }
        return cur->val;
      }

      // 获取第一个元素
      T GetFirst() { return Get(0); }

      // 获取最后一个元素
      T GetLast() { return Get(size_ - 1); }

      // 修改链表的第k(0-based)个元素的值（练习）
      void Set(int k, T val) {
        Check(k);
        Node* cur = dummy_head_->next;
        for (int i = 0; i < k; i++) {
         cur = cur->next;
        }
        cur->val = val;
      }

      // 查找链表中是否存在val
      bool Contains(T val) {
        Node* cur = dummy_head_->next;
        while (cur) {
         if (cur->val == val) return true;
         cur = cur->next;
        }
        return false;
      }

      // 打印链表
      void PrintLinkedList() {
        Node* cur = dummy_head_->next;
        while (cur) {
         cout << cur->val << "->";
         cur = cur->next;
        }
        cout << "NULL" << endl;
      }

      // 从链表中删除第k(0-based)个节点，并返回节点存储的值
      T Remove(int k) {
        Check(k);
        Node* pre = dummy_head_;
        for (int i = 0; i < k; i++) {
         pre = pre->next;
        }
        Node* del_node = pre->next;
        pre->next = pre->next->next;
        T ret = del_node->val;
        delete del_node;
        size_--;  // 不要忘记

        return ret;
      }

      // 删除第一个元素
      T RemoveFirst() {
        return Remove(0);
      }

      // 删除最后一个元素
      T RemoveLast() {
        return Remove(size_ - 1);  // 这里是 size_ - 1 不是 size_
      }

      // LinkedList() : head_(nullptr), size_(0) {}  // 无头结点版本
  
      // 在链表的第k(0-based)个位置添加新的元素(面试题中会用到，平时不常用)
      // 无头结点版本
      // void Add(int k, T val) {
      //   assert(k >= 0 && k < size_);

      //   if (k == 0) {
      //     AddFirst(val);
      //   } else {
      //     Node* pre = head;
      //     for (int i = 0; i < k - 1; i++) {
      //       pre = pre->next;
      //     }

      //     // Node* node = new Node(val);
      //     // node->next = pre->next;
      //     // pre->next = node;

      //     // 更优雅的写法
      //     pre->next = new Node(val, pre->next);
      //     size_++;
      //   }
      // }

      // 在链表头添加节点
      // 无头结点版本
      // void AddFirst(T val) {
      //   // Node* node = new Node(e);
      //   // node->next = head_;
      //   // head = node;

      //   // 更优雅的写法
      //   head_ = new Node(val, head_);
      //   size_++;
      // }

    };

    // TODO: 用递归实现链表的所有操作
    // 非常有意义，一定要做
    // 实际情况下不会使用递归的方式实现链表，但是确实是一个很好的练习题

    #endif  // LINKED_LIST_H_


Stack
========

.. code-block:: c++
    :linenos:

    #ifndef STACK_H_
    #define STACK_H_

    #include <iostream>

    #include "linked_list.h"
    #include "array.h"

    using namespace std;

    template <typename T>
    class Stack {
    public:
      virtual ~Stack() {}

      virtual int GetSize() = 0;

      virtual bool IsEmpty() = 0;

      virtual void Push(T val) = 0;

      virtual T Pop() = 0;

      virtual T Peek() = 0;

      virtual void PrintStack() = 0;
    };

    template <typename T>
    class LinkedListStack : public Stack<T> {
    public:
      LinkedListStack() : list_(new LinkedList<int>()) {}

      ~LinkedListStack() { delete list_; }

      virtual int GetSize() override { return list_->GetSize(); }

      virtual bool IsEmpty() override { return list_->IsEmpty(); }

      virtual void Push(T val) override { list_->AddFirst(val); }

      virtual T Pop() override { return list_->RemoveFirst(); }

      virtual T Peek() override { return list_->GetFirst(); }

      virtual void PrintStack() override { list_->PrintLinkedList(); }

    private:
      LinkedList<int>* list_;
    };

    template <typename T>
    class ArrayStack : public Stack<T> {
    public:
      ArrayStack() : array_(new Array<int>()) {}

      ~ArrayStack() { delete array_; }

      virtual int GetSize() override { return array_->GetSize(); }

      virtual bool IsEmpty() override { return array_->IsEmpty(); }

      virtual void Push(T val) override { array_->AddFirst(val); }

      virtual T Pop() override { return array_->RemoveFirst(); }

      virtual T Peek() override { return array_->GetFirst(); }

      virtual void PrintStack() override { cout << array_->ToString(); }

    private:
      Array<int>* array_;
    };

    #endif  // STACK_H_


Queue
=======

.. code-block:: c++
    :linenos:

    #ifndef QUEUE_H_
    #define QUEUE_H_

    #include <cassert>
    #include <iostream>

    #include "array.h"

    using namespace std;

    template <typename T>
    class Queue {
    public:
      virtual ~Queue() {}

      virtual int GetSize() = 0;

      virtual bool IsEmpty() = 0;

      virtual void Enqueue(T val) = 0;

      virtual T Dequeue() = 0;

      // 查看队首元素
      virtual T GetFront() = 0;

      virtual void PrintQueue() {}
    };

    template <typename T>
    class ArrayQueue : public Queue<T> {
    public:
      ArrayQueue() : array_(new Array<int>()) {}

      ~ArrayQueue() { delete array_; }

      virtual int GetSize() override { return array_->GetSize(); }

      virtual bool IsEmpty() override { return array_->IsEmpty(); }

      // O(1) 均摊
      virtual void Enqueue(T val) override { array_->AddLast(val); }

      // O(n) 当首元素出队时，其之后的所有元素都要向前移动一个位置
      virtual T Dequeue() override { return array_->RemoveFirst(); }

      // 查看队首元素
      virtual T GetFront() override { array_->GetFirst(); }

      virtual void PrintQueue() override { cout << array_->ToString(); };

    private:
      Array<int>* array_;
    };

    template <typename T>
    class LoopQueue : public Queue<T> {
    public:
      LoopQueue(int capacity)
         : data_(new T[capacity + 1]), front_(0),
           tail_(0), size_(0), capacity_(capacity) {}

      LoopQueue() : LoopQueue(10) {}

      int GetCapacity() { return capacity_; }

      // O(1)
      virtual int GetSize() override { return size_; }

      virtual bool IsEmpty() override {return front_ == tail_; }

      // O(1) 均摊
      virtual void Enqueue(T val) override {
        // 有一个空间会被浪费否则无法使用 tail 和 front 区分队列空或者队列满，因为，
        // 若所有的空间都是用的话，front == tail 既可以表示空，也可以表示满
        // 当故意浪费一个空间的时候，可以用 front == tail 表示空，用 front + 1 == tail
        // 表示满，当然还要考虑循环队列取模的问题
        if ((tail_ + 1) % (capacity_ + 1) == front_) {
         Resize(2 * capacity_);
        }

        data_[tail_] = val;
        tail_ = (tail_ + 1) % (capacity_ + 1);
        size_++;
      }

      // O(1) 均摊
      virtual T Dequeue() override {
        if (IsEmpty()) {
         throw invalid_argument("cannot dequeue from the empty queue.");
        }

        T ret = data_[front_];
        front_ = (front_ + 1) % (capacity_ + 1);
        size_--;
        if (size_ == capacity_ / 4 && capacity_ / 2 != 0) {
         Resize(capacity_ / 2);
        }
        return ret;
      }

      // O(1)
      virtual T GetFront() override {
        if (IsEmpty()) {
         throw invalid_argument("cannot dequeue from the empty queue.");
        }

        return data_[front_];
      }

      virtual void PrintQueue() override {
        cout << "current capacity: " << capacity_ << endl;
        for (int i = front_; i != tail_; i = (i + 1) % (capacity_ + 1)) {
         cout << data_[i] << " ";
        }
        cout << endl;
      }

    private:
      T* data_;
      int front_, tail_, size_, capacity_;
      // TODO: 浪费一个空间的情况下也可以不使用 size_
      // TODO: 使用 size_ 的情况下也可以不浪费一个空间
      // 因为使用 fornt_ 和 tail_ 就足以记录整个队列是空还是满

      void Resize(int new_capacity) {
        T* new_data = new T[new_capacity + 1];
        for (int i = 0; i < size_; i++) {
         new_data[i] = data_[(i + front_) % (capacity_ + 1)];
        }
        delete[] data_;
        data_ = new_data;
        front_ = 0;
        tail_ = size_;
        capacity_ = new_capacity;
      }
    };

    // 此处没有复用 linked_list.h 中的内容
    template <typename T>
    class LinkedListQueue : public Queue<T> {
    public:
      struct Node {
        T val;
        Node* next;

        Node(T val, Node* next) : val(val), next(next) {}

        Node(T val) : Node(val, nullptr) {};
      };

      LinkedListQueue() : size_(0) {}

      ~LinkedListQueue() { }

      virtual int GetSize() override { return size_; }

      virtual bool IsEmpty() override { return size_ == 0; }

      virtual void Enqueue(T val) override {
        if (tail_ == nullptr) {  // 此时 head_ 和 tail_ 都为空
         tail_ = new Node(val);
         head_ = tail_;
        } else {
         tail_->next = new Node(val);
         tail_ = tail_->next;
        }
        size_++;
      }

      virtual T Dequeue() override {
        assert(!IsEmpty());

        Node* del_node = head_;
        head_ = head_->next;
        T ret = del_node->val;
        delete del_node;

        // 当链表中只有一个元素时会出现这种情况
        if (head_ == nullptr) {
         tail_ = nullptr;
        }

        size_--;

        return ret;
      }

      virtual T GetFront() override {
        assert(!IsEmpty());
        return head_->val;
      }

      virtual void PrintQueue() override {
        Node* cur = head_;
        cout << "Queue: front ";
        while (cur) {
         cout << cur->val << "->";
         cur = cur->next;
        }
        cout << "NULL" << endl;
      }

    private:
      Node* head_ = nullptr;
      Node* tail_ = nullptr;
      int size_;
    };

    // 优先队列，重要
    template <typename T>
    class PriorityQueue : public Queue<T> {
    public:
      PriorityQueue(int capacity)
         : max_heap_(new MaxHeap<T>(capacity)) {}

      ~PriorityQueue() {
        delete max_heap_;
      }

      virtual int GetSize() override {
        return max_heap_->Size();
      }

      virtual bool IsEmpty() override {
        return max_heap_->IsEmpty();
      }

      virtual T GetFront() override {
        return max_heap_->FindRoot();
      }

      virtual void Enqueue(T val) override {
        max_heap_->Insert(val);
      }

      virtual T Dequeue() override {
        return max_heap_->ExtractRoot();
      }

    private:
      MaxHeap<T>* max_heap_;  // 这里应该是一个指针，不要搞错了
    };

    // TODO: 
    // 使用动态数组或者链表也可以实现优先队列，比较使用不同底层数据结构实现
    // 的优先队列的性能差异

    #endif  // QUEUE_H_

Sort
=====

.. code-block:: c++
    :linenos:

    #ifndef SORT_H_
    #define SORT_H_

    #include <algorithm>
    #include <iostream>
    #include "heap.h"

    using std::swap;

    // 选择排序  O(n^2)
    template<typename T>
    void SelectionSort(T arr[], int n) {
      for (int i = 0; i < n - 1; i++) {
        int min_index = i;
        for (int j = i + 1; j < n; j++) {
         if (arr[j] < arr[min_index]) {
           min_index =j;
         }
        }
        swap(arr[i], arr[min_index]);
      }
    }

    // 插入排序  O(n^2)
    template<typename T>
    void InsertionSortE1(T arr[], int n) {
      for (int i = 1; i < n; i++) {
        for (int j = i; j > 0 && arr[j] < arr[j - 1]; j--) {
         swap(arr[j], arr[j - 1]);
        }
      }
    }

    // 插入排序优化  O(n^2)
    // 对arr[0...n-1]排序
    template<typename T>
    void InsertionSortE2(T arr[], int n) {
      for (int i = 1; i < n; i++) {
        T e = arr[i];
        int j;
        for (j = i; j > 0 && e < arr[j - 1]; j--) {
         arr[j] = arr[j - 1];
        }
        arr[j] = e;
      }
    }

    // 重载InsertionSortE2
    // 对arr[L...R]排序
    template<typename T>
    void InsertionSortE2(T arr[], int L, int R) {
      for (int i = L + 1; i <= R; i++) {
        T e = arr[i];
        int j;
        for (j = i; j > L && e < arr[j - 1]; j--) {
         arr[j] = arr[j - 1];
        }
        arr[j] = e;
      }
    }

    // 冒泡排序
    // 写法一
    template<typename T>
    void BubbleSortE1(T arr[], int n) {
      for (int i = 0; i < n - 1; i++) {
        bool swaped = false;
        for (int j = n - 1; j > i; j--) {
         if (arr[j] < arr[j - 1]) {
           swap(arr[j], arr[j - 1]);
           swaped = true;
         }
        }
        if (!swaped) {
         return;
        }
      }
    }

    // 写法二
    template<typename T>
    void BubbleSortE2(T arr[], int n){
      bool swapped;
      do {
        swapped = false;
        for(int i = 1 ; i < n; i++) {
         if(arr[i-1] > arr[i]) {
           swap(arr[i-1], arr[i]);
           swapped = true;
         }
        }
        // 优化, 每一趟Bubble Sort都将最大的元素放在了最后的位置
        // 所以下一次排序, 最后的元素可以不再考虑
        n--;
      } while(swapped);
    }

    // 优化写法二
    template<typename T>
    void BubbleSortE3(T arr[], int n){
      int new_n;  // 使用new_n进行优化
      do {
        new_n = 0;
        for(int i = 1; i < n; i++) {
         if(arr[i-1] > arr[i]) {
           swap(arr[i-1] , arr[i]);
           // 记录最后一次的交换位置,在此之后的元素在下一轮扫描中均不考虑
           new_n = i;
         }
        }
        n = new_n;
      } while(new_n > 0);
    }

    // TODO 希尔排序


    // 将arr[L...mid]和arr[mid+1...R]两部分进行归并
    template<typename T>
    void __Merge(T arr[], int L, int mid, int R) {
      T aux[R - L + 1];
      for (int i = L; i <= R; i++) {
        aux[i - L] = arr[i];
      }
      int i = L, j = mid + 1;
      for (int k = L; k <= R; k++) {
        if (i > mid) {
         arr[k] = aux[j - L];
         j++;
        } else if (j > R) {
         arr[k] = aux[i - L];
         i++;
        } else if (aux[i - L] < aux[j - L]) {
         arr[k] = aux[i - L];
         i++;
        } else {
         arr[k] = aux[j - L];
         j++;
        }
      }
    }

    // 递归使用归并排序，对arr[L...R]的范围进行排序
    template<typename T>
    void __MergeSort(T arr[], int L, int R) {
      // 优化一：当元素数量较少时采用插入排序
      // if (L >= R) {
      //   return;
      // }
      if (R - L <= 15) {
        InsertionSortE2(arr, L, R);
        return;
      }
      int mid = (L + R) / 2;
      __MergeSort(arr, L, mid);
      __MergeSort(arr, mid + 1, R);
      // 优化二：若递归过程中发现元素已经有序了，不需要再进行merge
      // __Merge(arr, L, mid, R);
      if (arr[mid] > arr[mid + 1]) {
        __Merge(arr, L, mid, R);
      }
    }

    // 归并排序，自顶向下实现
    template<typename T>
    void MergeSort(T arr[], int n) {
      __MergeSort(arr, 0, n - 1);
    }

    // 归并排序，自底向上实现，仍然是nlog(n)的时间复杂度(虽然有两层循环，所以不要轻易以循
    // 环层数来判断算法的复杂度)
    // TODO: 优化策略同上
    // TODO: 自底向上的归并排序没有使用数组通过索引获取元素的特性，因此可用于链表的排序，
    //       从而实现对链表进行nlog(n)时间复杂度的排序(面试题)
    template<typename T>
    void MergeSortBU(T arr[], int n) {
      for (int size = 1; size <= n; size += size) {
        for (int i = 0; i + size < n; i += size + size) {
         // 对arr[i...i+size-1]和arr[i+size...i+2*size-1]进行归并
         __Merge(arr, i, i + size - 1, std::min(i + size + size - 1, n - 1));
        }
      }
    }

    // 对arr[L...R]进行partion操作
    // 返回p，使得arr[L...p-1] < arr[p]，arr[p+1...R] > arr[p]
    template<typename T>
    int __Partition1(T arr[], int L, int R) {
      // 优化一: 随机选择参照元素防止快速排序退化
      swap(arr[L], arr[rand() % (R - L + 1) + L]);
      T v = arr[L];
      // arr[L...p-1] < v，arr[p+1...R] > v
      int j = L;
      for (int i = L + 1; i <= R; i++) {
        if (arr[i] < v) {
         swap(arr[j + 1], arr[i]);
         j++;
        }
      }
      swap(arr[L], arr[j]);
      return j;
    }

    // 另一种partition策略
    // 解决数组中的数字重复率过高时快速排序退化问题
    template<typename T>
    int __Partition2(T arr[], int L, int R) {
      // 随机选择参照元素防止快速排序退化
      swap(arr[L], arr[rand() % (R - L + 1) + L]);
      T v = arr[L];
      // arr[L+1...i) <= v，arr(j...R] >= v
      int i = L + 1, j = R;
      while (true) {
        while (i <= R && arr[i] < v) i++;
        while (j >= L + 1 && arr[j] > v) j--;
        if (i > j) break;
        swap(arr[i], arr[j]);
        i++;
        j--;
      }
      swap(arr[L], arr[j]);
      return j;
    }

    // 对arr[L...R]进行快速排序
    // 注意：
    // (1) 当整个数组接近有序的情况下，快速排序在进行划分时极度不平衡，可能出现一大一小的
    // 两个区间，递归树的高度接近n，这种情况下快速排序将会退化成O(n^2)级别，而归并排序由于
    // 每次划分都是接近均分的，仍然可以保持很好的效率，递归树的高度仍然是log(n)，所以在元
    // 素相对有序的情况下，快速排序并不占优。解决办法是每次选择参照元素的时候采用随机选择的
    // 方式，而不是每次选择固定位置元素。
    // (2) 当数组中的数字重复率很高时，快速排序也会退化成O(n^2)。原因是当数字重复出现时也
    // 可能导致划分出的区间极度不平衡，从而导致递归树高度接近n。
    template<typename T>
    void __QuickSort(T arr[], int L, int R) {
      // 优化二: 小数据量情况下使用InsertionSort
      // if (L >= R) {
      //   return;
      // }
      if (R - L <= 15) {
        InsertionSortE2(arr, L, R);
        return;
      }
      // int p = __Partition1(arr, L, R);
      int p = __Partition2(arr, L, R);  // better
      __QuickSort(arr, L, p - 1);
      __QuickSort(arr, p + 1, R);
    }

    // 快速排序
    template<typename T>
    void QuickSort(T arr[], int n) {
      srand(time(nullptr));
      __QuickSort(arr, 0, n - 1);
    }

    // 更加优秀的快速排序策略(采用三路partition)
    // quick sort 3 ways
    template<typename T>
    void __QuickSort3Ways(T arr[], int L, int R) {
      if (R - L <= 15) {
        InsertionSortE2(arr, L, R);
        return;
      }

      // partition
      swap(arr[L], arr[rand() % (R - L + 1) + L]);
      T v = arr[L];

      int lt = L;
      int gt = R + 1;
      int i = L + 1;
      while (i < gt) {
        if (arr[i] < v) {
         swap(arr[i], arr[lt + 1]);
         lt++;
         i++;
        } else if (arr[i] > v) {
         swap(arr[i], arr[gt - 1]);
         gt--;
        } else {
         i++;
        }
      }

      swap(arr[L], arr[lt]);

      __QuickSort3Ways(arr, L, lt - 1);
      __QuickSort3Ways(arr, gt, R);
    }

    template<typename T>
    void QuickSort3Ways(T arr[], int n) {
      srand(time(nullptr));
      __QuickSort3Ways(arr, 0, n - 1);
    }

    // 堆排序，多用于动态数据的维护
    template<typename T>
    void HeapSortE1(T arr[], int n) {
      MaxHeap<int> max_heap = MaxHeap<int>(n);
      for (int i = 0; i < n; i++) {
        max_heap.Insert(arr[i]);
      }
      for (int i = n - 1; i >= 0; i--) {
        arr[i] = max_heap.ExtractRoot();
      }
    }

    // 优化建堆过程，使用heapify
    template<typename T>
    void HeapSortE2(T arr[], int n) {
      MaxHeap<int> max_heap = MaxHeap<int>(arr, n);
      for (int i = n - 1; i >= 0; i--) {
        arr[i] = max_heap.ExtractRoot();
      }
    }

    // 改写ShifDown，数组从rank 0开始存储
    template<typename T>
    void __ShiftDown(T arr[], int n, int k) {
      while (2 * k + 1 < n) {
        int j = 2 * k + 1;
        if (j + 1 < n && arr[j + 1] > arr[j]) j += 1;
        if (arr[k] >= arr[j]) break;
        swap(arr[k], arr[j]);
        k = j;
      }
    }

    // 优化空间复杂度，原地堆排序，不适用额外的空间，隐含条件数组是从rank 0开始存储的
    // 因此：parent(i)=(i-1)/2  left_chaild(i)=2*i+1  right_child(i)=2*i+2
    // 少了开辟新空间和往新空间中赋值和处理的过程，性能更优
    template<typename T>
    void HeapSortE3(T arr[], int n) {
      // heapify建堆
      // 注意i的起始位置应该是(n-1-1)/2，因为最后一个元素的索引是n-1
      for (int i = (n - 1 - 1) / 2; i >= 0; i--) {
        __ShiftDown(arr, n, i);
      }

      for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);
        __ShiftDown(arr, i, 0);
      }
    }

    // 算法与数据结构体系课新增内容，老师使用的是 java，这里采用 c++ 进行实现

    // 希尔排序
    template <typename T>
    void ShellSortE1(T arr[], int n) {
      int h = n / 2;
      // 最终复杂度为：O(n^2-A) A值查看 bobo 老师 PPT，总之其复杂度小于 O(n^2)
      // 每一轮都让数组更有序，所以实际情况下希尔排序的性能比实际的理论分析更好
      // 远远好于 O(n^2)，甚至接近 O(n*log(n)) (数据量不是很大的情况下)
      while (h >= 1) {
        // 下面这个循环的复杂度为 O(h*(n/h)^2) = O(n^2/h)
        for (int start = 0; start < h; start++) {  // start 是每个子数组其实元素的索引
         // 对 data[start, start+h, start+2h ...] 进行插入排序
         // 标准的插入排序过程
         // O((n/h)^2)
         for (int i = start + h; i < n; i += h) {
           T e = arr[i];
           int j;
           for (int j = i; j - h >= 0 && e < arr[j - h]; j -= h) {
             arr[j] = arr[j - h];
           }
           arr[j] = e;
         }
        }

        h /= 2;
      }
    }

    // 希尔排序
    // 压缩一层循环，但是复杂度上没有任何变化（一种更简洁的写法）
    template <typename T>
    void ShellSortE2(T arr[], int n) {
      int h = n / 2;
      while (h >= 1) {
        // 对 data[h, h + 1, ...] 进行插入排序
        for (int i = h; i < n; i++) {
         T e = arr[i];
         int j;
         for (int j = i; j - h >= 0 && e < arr[j - h]; j -= h) {
             arr[j] = arr[j - h];
         }
         arr[j] = e;
        }

        h /= 2;
      }
    }

    // 希尔排序
    // 优化：步长序列
    template <typename T>
    void ShellSortE3(T arr[], int n) {
      int h = 1;
      while (h < n) {
        h = 3 * h + 1;  // 步长为 3，寻找最大的 h
      }

      while (h >= 1) {
        // 对 data[h, h + 1, ...] 进行插入排序
        for (int i = h; i < n; i++) {
         T e = arr[i];
         int j;
         for (int j = i; j - h >= 0 && e < arr[j - h]; j -= h) {
             arr[j] = arr[j - h];
         }
         arr[j] = e;
        }

        h /= 3;
      }
    }

    // 总结：
    // 希尔排序可是使用不同的步长，排序效果也不同
    // 步长 h 可以看成希尔排序的超参数，这种参数和用户传给算法的参数不同
    // 到目前为止，学界对于哪个超参数可以使希尔排序的性能最优还没有定论
    // 参考文章：hbfs.wordpress.com/2011/03/01/shellsort/
    // 希尔排序实现简单，数据量适中的情况性能也非常优秀，对于嵌入式来说，有可能底层没有递
    // 归、或者递归的开销比较大，这种情况使用希尔排序不失为一个好的选择（希尔排序只用到了循环）

    #endif  // SORT_H_

BST Tree
===========

.. code-block:: c++
    :linenos:

    #ifndef SEARCH_H_
    #define SEARCH_H_

    #include <iostream>
    #include <queue>
    #include <cassert>

    // 二分查找 复杂度：O(log(n))
    // 在有序数组arr中查找target
    // 如果找到target，则返回相应的index索引
    // 如果没有找到target，则返回-1
    template<typename T>
    int BinarySearch(T arr[], int n, T target) {
      // 在arr[L...R]之中查找target
      int L = 0, R = n - 1;
      while (L <= R) {
        // int mid = (L + R ) / 2;  // L+R 有可能会导致溢出，应该转用下面的写法，重要！！
        int mid = L + (R - L) / 2;
        if (target == arr[mid]) {
          return mid;
        } else if (target < arr[mid]) {
          // 在arr[L...mid-1]之中查找target
          R = mid - 1;
        } else {
          // 在arr[mid+1...R]之中查找target
          L = mid + 1;
        }
      }
      return -1;
    }

    // TODO: 用递归实现二分查找

    // TODO: 二分查找变种
    // floor：返回target第一次出现的位置
    // cail：返回target最后一次出现的位置
    // 应对target在数组中重复出现的情况

    template<typename Key, typename Value>
    class BST {
    public:
      BST() : root_(nullptr), count_(0) {}

      ~BST() { Destroy(root_); }

      inline int Size() { return count_; }

      inline bool IsEmpty() { return count_ == 0; }

      void Insert(Key key, Value value) {
        root_ = Insert(root_, key, value);  // 不要忘记 root_ =
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

    private:
      struct Node {
        Key key;
        Value value;
        Node* left;
        Node* right;

        Node(Key key, Value value)
          : key(key), value(value), left(nullptr), right(nullptr) {}

        Node(Node* node)
          : key(node->key), value(node->value), left(node->left), right(node->right) {}
      };

      Node* root_;
      int count_;

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

        return node;  // don't forget
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
        if (key < node->key) {
          node->left = Remove(node->left, key);
          return node;
        } else if (key > node->key) {
          node->right = Remove(node->right, key);
          return node;
        } else {
          if (node->left == nullptr) {
            Node* right_node = node->right;
            delete node;
            count_--;
            return right_node;
          }
          if (node->right == nullptr) {
            Node* left_node = node->left;
            delete node;
            count_--;
            return left_node;
          }
      
          // node->left != nullptr && node->right != nullptr
          // 陷阱，要复制一次最小节点，因为RemoveMin会把这个最小节点删除
          Node* successor = new Node(Minimum(node->right));
          count_++;
          successor->right = RemoveMin(node->rihgt);  // RemoveMin内会发生一次count_--
          successor->left = node->left;
          delete node;
          count_--;
          return successor;
        }
      }

      // TODO: 采用第二种策略，用节点的左子树的最大值代替节点，从而达到删除的目的

      // 重要：
      // TODO:先根遍历的非递归实现（使用栈）
      // TODO: 层序遍历（使用队列）

    };

    #endif  // SEARCH_H_

AVL Tree
=========

.. code-block:: c++
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

RB Tree
========
.. code-block:: c++
    :linenos:

    #ifndef RB_TREE_
    #define RB_TREE_

    namespace RedBlackTree {

      typedef enum { RED, BLACK } Color_t;

      // 规定红黑树的红色节点在左边，向左倾斜

      // 红黑树的五条性质讲解：
      // 体系课 2-4 小节第 3 分钟左右
      template<typename Key, typename Value>
      class RBTree {
      public:
        RBTree() : root_(nullptr), count_(0) {}
  
        ~RBTree() { Destroy(root_); }
  
        inline int Size() { return count_; }
  
        inline bool IsEmpty() { return count_ == 0; }
  
        void Insert(Key key, Value value) {
          root_ = Insert(root_, key, value);  // 不要忘记 root_ =
          root_->color = BLACK;  // 保持最终的根节点为黑色
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
  
      private:
        struct Node {
          Key key;
          Value value;
          Node* left;
          Node* right;
          Color_t color;
  
          // TODO: 为什么节点的默认颜色为红色？
          // 2-3 小节 17 分有讲，但是没听懂，之后代码实现后再回头想一想。
          Node(Key key, Value value)
            : key(key), value(value), left(nullptr), right(nullptr), color(RED) {}
  
          Node(Node* node)
            : key(node->key), value(node->value), left(node->left), right(node->right), color(RED) {}
        };
  
        Node* root_;
        int count_;

        // 颜色翻转
        // 调用 FlipColors 时要保证符合对应的形状，调用前要有一定判断（在调用的地方实现判断逻辑，这里不进行判断）
        void FlipColors(Node* node) {
          node->color = RED;
          node->left->color = BLACK;
          node->right->color = BLACK;
        }

        //    node                         x
        //   /   \        左旋转         /   \
        //  T1    x     --------->    node   T3
        //       / \                 /  \
        //      T2 T3              T1   T2
        Node* LeftRotate(Node* node) {
          Node* x = node->right;
      
          // 左旋转
          node->right = x->left;
          x->left = node;

          // 维护节点颜色
          x->color = node->color;
          node->color = RED;
      
          // 返回旋转后的根节点
          return x;
        }

        //       node                       x
        //      /   \        右旋转        /  \
        //     x    T3     --------->    y  node
        //    / \                            / \
        //  y  T2                         T2  T3
        Node* RightRotate(Node* node) {
          Node* x = node->left;
      
          // 右旋转
          node->left = x->right;
          x->right = node;

          // 维护节点颜色
          x->color = node->color;
          node->color = RED;  // 表示该节点和其父亲节点是绑定在一起的
      
          // 返回旋转后的根节点
          return x;
        }

        bool IsRed(Node* node) {
          if (node == nullptr) return false;  // 对应性质：每个叶子节点（最底层的空节点）是黑色的
          return node->color == RED ? true : false;
        }
  
        // 向以node为根的红黑树中插入节点(key, value)，返回插入新节点后的红黑树的根
        // TODO: 非递归实现
        Node* Insert(Node* node, Key key, Value value) {
          if (node == nullptr) {
            count_++;
            return new Node(key, value);  // 默认插入红色节点
          }
      
          if (node->key == key) {
            node->key = key;
          } else if (node->key > key) {
            node->left = Insert(node->left, key, value);
          } else {
            node->right = Insert(node->right, key, value);
          }

          // 红黑树维护的逻辑链条
          // 三个 if 不是 if-else 的关系  
          if (IsRed(node->right) && !IsRed(node->left)) {
            node = LeftRotate(node);
          }

          if (IsRed(node->left) && IsRed(node->left->left)) {
            node = RightRotate(node);
          }

          if (IsRed(node->left) && IsRed(node->right)) {
            FlipColors(node);
          }
  
          return node;  // don't forget
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
          if (key < node->key) {
            node->left = Remove(node->left, key);
            return node;
          } else if (key > node->key) {
            node->right = Remove(node->right, key);
            return node;
          } else {
            if (node->left == nullptr) {
              Node* right_node = node->right;
              delete node;
              count_--;
              return right_node;
            }
            if (node->right == nullptr) {
              Node* left_node = node->left;
              delete node;
              count_--;
              return left_node;
            }
        
            // node->left != nullptr && node->right != nullptr
            // 陷阱，要复制一次最小节点，因为RemoveMin会把这个最小节点删除
            Node* successor = new Node(Minimum(node->right));
            count_++;
            successor->right = RemoveMin(node->rihgt);  // RemoveMin内会发生一次count_--
            successor->left = node->left;
            delete node;
            count_--;
            return successor;
          }
        }
  
        // TODO: 采用第二种策略，用节点的左子树的最大值代替节点，从而达到删除的目的
  
      };

    }  // RedBlackTree

    #endif  // RB_TREE_

Heap
======

.. code-block:: c++
    :linenos:

    #ifndef MAX_HEAP_H_
    #define MAX_HEAP_H_

    #include <algorithm>
    #include <cassert>
    #include <ctime>

    using std::swap;  // TODO: 优化点，不使用swap，而是选择每次挪动赋值的形式，等确定最终
                     // 位置后再将目标数字填入

    template<typename T>
    class MaxHeap {
    public:
      // 索引0不使用，从而更好的利用完全二叉树的性质
      // 即：parent(i)=i/2  left_chaild(i)=2*i  right_child(i)=2*i+1
      MaxHeap(int capacity) : size_(0), capacity_(capacity) {
        data_ = new T[capacity + 1];
      }

      // TODO: heapify建堆O(n)比每次插入一个数建堆O(nlog(n))快，待证明
      // 直观上理解，heapify建堆过程调用ShifDown的次数只有另一种方法的一半
      MaxHeap(T arr[], int n) : size_(n), capacity_(n) {
        data_ = new T[n + 1];
        for (int i = 0; i < n; i++) {
          data_[i + 1] = arr[i];
        }
        for (int i = size_ / 2; i >= 1; i--) {
          ShiftDown(i);
        }
      }

      virtual ~MaxHeap() { delete[] data_;}

      inline int Size() { return size_; }

      inline bool IsEmpty() { return size_ == 0; }

      void Insert(T item) {
        assert(size_ + 1 <= capacity_);  // TODO: 若空间不够用，重新申请更大的空间
        data_[size_ + 1] = item;
        size_++;
        // shift up
        ShiftUp(size_);
      }

      // 每次只能从堆中取出最大的元素
      T ExtractRoot() {
        assert(size_ > 0);
        T ret = data_[1];

        swap(data_[1], data_[size_]);
        size_--;
        ShiftDown(1);

        return ret;
      }

      T FindRoot() {
        assert(size_ > 0);
        return data_[1];
      }

      void PrintData() {
        std::cout << "[  ";
        for (int i = 1; i < size_; i++) {  // 注意序号从1开始
          std::cout << data_[i] << "  ";
        }
        std::cout << "]" << std::endl;
      }

    protected:
      virtual void ShiftUp(int k) {
        while (k > 1 && data_[k] > data_[k / 2]) {
          swap(data_[k], data_[k / 2]);
          k /= 2;
        }
      }

      virtual void ShiftDown(int k) {
        while (2 * k <= size_) {
          int j = 2 * k;
          if (j + 1 <= size_ && data_[j + 1] > data_[j]) {
           j += 1;
          }
          if (data_[k] >= data_[j]) {
           break;
          }
          swap(data_[k], data_[j]);
          k = j;
        }
      }

      T* data_;
      int size_;
      int capacity_;
    };

    // 一个值得注意的问题：
    // 模板类继承模板类时由于存在偏特化问题，一个模板子类无法在实例化之前就知道他的模板父类
    // 到底是谁，因此名字也无法resolve，所以只能this->了，子类无法看见父类的成员变量
    // 解决办法：
    // 使用this指针或者使用Base<T>::data_方式
    template <typename T>
    class MinHeap: public MaxHeap<T> {
    public:
      MinHeap(int capacity) : MaxHeap<T>(capacity) {}

      MinHeap(T arr[], int n) : MaxHeap<T>(arr, n) {}

    private:
      virtual void ShiftUp(int k) {
        while (k > 1 && this->data_[k] < this->data_[k / 2]) {
          swap(this->data_[k], this->data_[k / 2]);
          k /= 2;
        }
      }

      virtual void ShiftDown(int k) {
        while (2 * k <= this->size_) {
          int j = 2 * k;
          if (j + 1 <= this->size_ && this->data_[j + 1] < this->data_[j]) {
           j += 1;
          }
          if (this->data_[k] <= this->data_[j]) {
           break;
          }
          // 使用this->data_或者MaxHeap<T>::data_均可
          swap(MaxHeap<T>::data_[k], MaxHeap<T>::data_[j]);
          k = j;
        }
      }
    };

    // 索引堆(一种高级数据结构，之后的图论中会用到)
    // 引入索引堆的原因
    // (1) 上述一般堆在操作的过程中需要进行直接对元素进行拷贝操作，对于某些数据类型可能很
    //     耗时;
    // (2) 一般堆的操作打乱了原始数组中索引和元素的映射关系
    template<typename T>
    class IndexMaxHeap {
    public:
      // 索引0不使用，从而更好的利用完全二叉树的性质
      // 即：parent(i)=i/2  left_chaild(i)=2*i  right_child(i)=2*i+1
      IndexMaxHeap(int capacity) : size_(0), capacity_(capacity) {
        data_ = new T[capacity + 1];
        indexes_ = new int[capacity + 1];
        reverse_ = new int[capacity + 1];
        for (int i = 0; i <= capacity; i++) {
          reverse_[i] = 0;  // 表明i这个索引在堆中不存在
        }
      }

      virtual ~IndexMaxHeap() {
        delete[] data_;
        delete[] indexes_;
        delete[] reverse_;
      }

      inline int Size() { return size_; }

      inline bool IsEmpty() { return size_ == 0; }

      // 传入的i对用户而言从0开始索引
      void Insert(int i, T item) {
        assert(size_ + 1 <= capacity_);  // TODO: 若空间不够用，重新申请新的更大的空间
        assert(i + 1 >= 1 && i + 1 <= capacity_);

        i += 1;
        data_[i] = item;
        indexes_[size_ + 1] = i;
        reverse_[i] = size_ + 1;

        size_++;
        // shift up
        ShiftUp(size_);
      }

      // 每次只能从堆中取出最大的元素
      T ExtractRoot() {
        assert(size_ > 0);
        T ret = data_[indexes_[1]];

        swap(indexes_[1], indexes_[size_]);
        reverse_[indexes_[1]] = 1;
        // 删除最大元素，所以reverse_中对应的元素应该设置为0
        reverse_[indexes_[size_]] = 0;

        size_--;
        ShiftDown(1);

        return ret;
      }

      int ExtractRootIndex() {
        assert(size_ > 0);
        T ret = indexes_[1] - 1;

        swap(indexes_[1], indexes_[size_]);
        reverse_[indexes_[1]] = 1;
        // 删除最大元素，所以reverse_中对应的元素应该设置为0
        reverse_[indexes_[size_]] = 0;
        size_--;
        ShiftDown(1);

        return ret;
      }

      T GetItem(int i) {
        // 要保证当前的索引堆包含i索引
        assert(Contain(i));
    
        return data_[i + 1];
       }

      // 复杂度为 O(n + log(n))
      void ChangeE1(int i, T new_item) {
        // 要保证当前的索引堆包含i索引
        assert(Contain(i));

        i += 1;
        data_[i] = new_item;
    
        // 找到indexes_[j]=i，j表示data_[i]在堆中的位置
        // 之后ShiftUp(j), ShiftDown(j)
        for (int j = 1; j <= size_; j++) {
          if (indexes_[j] == i) {
           // 要么ShiftUp(j)和ShiftDown(j)都不发生，要么只发生其中一种
           ShiftUp(j);
           ShiftDown(j);
           return;
          }
        }
      }

      // change函数的优化, 经典思路：反向查找  复杂度: O(log(n))
      void ChangeE2(int i, T new_item) {
        // 要保证当前的索引堆包含i索引
        assert(Contain(i));

        i += 1;
        data_[i] = new_item;
    
        int j = reverse_[i];
        ShiftUp(j);
        ShiftDown(j);
      }

      void PrintData() {
        std::cout << "[  ";
        for (int i = 1; i < size_; i++) {  // 注意序号从1开始
          std::cout << data_[indexes_[i]] << "  ";
        }
        std::cout << "]" << std::endl;
      }

      bool Contain(int i) {
        assert(i + 1 >= 1 && i + 1 <= capacity_);
        return reverse_[i +1] != 0;
      }

    protected:
      T* data_;
      // reverse_数组和indexes_数组拥有的性质：
      // indexes_[i] = j  表示：堆中的第i个位置应该存放数组的第j个数
      // reverse_[j] = i  表示：数组的第j个数应该存放在堆中的第i个位置
      // indexes_[reverse_[i]] = i
      // reverse_[indexes_[i]] = i
      int* indexes_;
      int* reverse_;  // 优化change
      int size_;
      int capacity_;

      virtual void ShiftUp(int k) {
        while (k > 1 && data_[indexes_[k]] > data_[indexes_[k / 2]]) {
          swap(indexes_[k], indexes_[k / 2]);
          // TODO: 不理解
          reverse_[indexes_[k]] = k;
          reverse_[indexes_[k / 2]] = k / 2;

          k /= 2;
        }
      }

      virtual void ShiftDown(int k) {
        while (2 * k <= size_) {
          int j = 2 * k;
          if (j + 1 <= size_ && data_[indexes_[j + 1]] > data_[indexes_[j]]) {
           j += 1;
          }
          if (data_[indexes_[k]] >= data_[indexes_[j]]) {
           break;
          }
          swap(indexes_[k], indexes_[j]);
          reverse_[indexes_[k]] = k;
          reverse_[indexes_[j]] = j;
          k = j;
        }
      }

    };

    template<typename T>
    class IndexMinHeap : public IndexMaxHeap<T> {
    public:
      IndexMinHeap(int capacity) : IndexMaxHeap<T>(capacity) {}

    private:
      virtual void ShiftUp(int k) {
        while (k > 1 && this->data_[this->indexes_[k]] < this->data_[this->indexes_[k / 2]]) {
          swap(this->indexes_[k], this->indexes_[k / 2]);
          // TODO: 不理解
          this->reverse_[this->indexes_[k]] = k;
          this->reverse_[this->indexes_[k / 2]] = k / 2;

          k /= 2;
        }
      }

      virtual void ShiftDown(int k) {
        while (2 * k <= this->size_) {
          int j = 2 * k;
          if (j + 1 <= this->size_ && this->data_[this->indexes_[j + 1]] < this->data_[this->indexes_[j]]) {
           j += 1;
          }
          if (this->data_[this->indexes_[k]] <= this->data_[this->indexes_[j]]) {
           break;
          }
          swap(this->indexes_[k], this->indexes_[j]);
          this->reverse_[this->indexes_[k]] = k;
          this->reverse_[this->indexes_[j]] = j;
          k = j;
        }
      }
    };

    #endif  // MAX_HEAP_H_

Segment Tree
===============

.. code-block:: c++
    :linenos:

    #include <iostream>
    #include <cassert>
    #include <stdexcept>

    using namespace std;

    template <typename T>
    class SegmentTree {
    public:
      SegmentTree(T arr[], int n) : size_(n) {
        data_ = new T[n];
        tree_ = new T[4 * n];  // TODO: 分析设定大小为 4 * n 的原因
        for (int i = 0; i < n; i++) {
          data_[i] = arr[i];
        }

        // 建立线段树
        BuildSegmentTree(0, 0, size_ - 1);
      }

      ~SegmentTree() {
        delete[] data_;
        delete[] tree_;
      }

      T Get(int index) {
        assert(index >= 0 && index < size_);
        return data_[index];
      }

      inline int GetSize() { return size_; }

      inline int LeftChild(int index) { return 2 * index + 1; }  // 线段树中不需要查询 parent

      inline int RightChild(int index) { return 2 * index + 2; }

      // TODO: Print tree

      // 返回 [L,R] 区间存储的值  O(log(n))
      T Query(int query_L, int query_R) {
        assert(query_L >= 0 && query_L < size_ && query_R >= 0 && query_R < size_ && query_L <= query_R);
        return _Query(0, 0, size_ - 1, query_L, query_R);
      }

      // O(log(n))
      void Set(int index, int e) {
       _Set(0, 0, size_ - 1, index, e); 
      }

    private:
      T* data_;
      int size_;
      T* tree_;
      // TODO: function<> 的使用

      // 在 index 位置创建表示区间 [L, R] 的线段树  O(log(n))
      void BuildSegmentTree(int tree_index, int L, int R) {
        if (L == R) {
          tree_[tree_index] = data_[L];
          return;
        }

        int left_child_index = LeftChild(tree_index);
        int right_child_index = RightChild(tree_index);

        int mid = L + (R - L) / 2;  // 防止越界
        BuildSegmentTree(left_child_index, L, mid);
        BuildSegmentTree(right_child_index, mid + 1, R);

        // 和具体业务相关，如最大值、最小值、求和等等
        // TODO: 可以让用户传入具体的操作，更加灵活，可以考虑让用户传入函数指针、functor等
        tree_[tree_index] = tree_[left_child_index] + tree_[right_child_index];
      }

      T _Query(int tree_index, int L, int R, int query_L, int query_R) {
        // 查询区间[query_L, query_R]和当前区间刚好重合
        if (query_L == L && query_R == R) return tree_[tree_index];

        // 计算当前区间的中点，注意越界问题
        int mid = L + (R - L) /2;
    
        // 计算 child index
        int left_tree_index = LeftChild(tree_index);
        int right_tree_index = RightChild(tree_index);

        if (query_R <= mid) {
          // 查询区间[query_L, query_R]在当前区间的左半部分[L, mid]
          return _Query(left_tree_index, L, mid, query_L, query_R);
        } else if (query_L >= mid +1) {
         // 查询区间[query_L, query_R]在当前区间的右半部分[mid + 1, R]
          return _Query(right_tree_index, mid + 1, R, query_L, query_R);
        }

        // 查询区间[query_L, query_R]在当前区间的左右两部分
        T left_result = _Query(left_tree_index, L, mid, query_L, mid);
        T right_result = _Query(right_tree_index, mid + 1, R, mid + 1, query_R);

        // TODO: 可以由用户指定操作，例如求和、求最大值、求最小值等
        return left_result + right_result;
      }

      void _Set(int tree_index, int L, int R, int index, int e) {
        if (L == R) {
          tree_[tree_index] = e;
          return;
        }

        int mid = L + (R - L) / 2;
        int left_child_index = LeftChild(tree_index);
        int right_child_index = RightChild(tree_index);

        if (index <= mid) {
          _Set(left_child_index, L, mid, index, e);
        } else {
          assert(index >= mid + 1);
          _Set(right_child_index, mid + 1, R, index, e);
        }

        tree_[tree_index] = tree_[left_child_index] + tree_[right_child_index];  // TODO: 用户自定义
      }
    };

Trie
======

.. code-block:: c++
    :linenos:

    #ifndef TRIE_H_
    #define TRIR_H_

    #include <map>
    #include <string>

    using namespace std;

    // 此处没有使用泛型编程，因为Trie这种数据结构是专门针对字符串设置的
    // 如果有需要的话，可以改造成泛型，将char改成对应的类型即可，例如中
    // 文字符
    class Trie {
      // 要定义在使用它的代码之前
      struct Node {
        bool is_word_end;
        map<char, Node*> next;
    
        Node () : is_word_end(false), next(map<char, Node*>()) {}
      };

    public:
      Trie() : root_(new Node()), size_(0) {}

      ~Trie() {
        Delete(root_);
      }

      // 释放空间，使用递归
      // 这个释放资源的方式比较特别，但是是正确的
      // 释放资源的次数等于new Node的次数，（!!!）但是不等于单词数量
      // 因为，插入一个单词可能会执行多次new Node，也可能一次也不执行
      void Delete(Node* node) {
        if (node->next.size() == 0) return;  // 递归到底的情况，其实对于本题来说可以省略不写
        for (map<char, Node*>::iterator itor = node->next.begin(); itor != node->next.end(); itor++) {
          Delete(itor->second);
        }
        delete node;
      }

      int GetSize() { return size_; }

      // 向 Trie 中添加新的单词
      // 向树中添加元素有递归和非递归两种写法，此处使用非递归写法，
      // 注意复习之前小节实现的递归写法
      // TODO: 增加递归写法
      void Add(string word) {
        Node* cur = root_;
        for (int i = 0; i < word.size(); i++) {
          char c = word[i];
          if (cur->next.find(c) == cur->next.end()) {
            // ！！！注意：
            // 此处不能使用at，因为当元素不存在时，
            // 若使用at会跑出out_of_range异常
            // 而使用 [] 则会默认添加新的元素
            cur->next[c] = new Node();
          }
          cur = cur->next[c];
        }
        if (!cur->is_word_end) {  // 防止在添加之前单词已经存在
          cur->is_word_end = true;
          size_++;
        }
      }

      // 查询 trie 中是否包含某个单词
      // TODO: 递归写法
      bool Search(string word) {
        Node* cur = root_;
        for (int i = 0; i < word.size(); i++) {
          char c = word[i];
          if (cur->next.find(c) == cur->next.end()) {
            return false;
          }
          cur = cur->next[c];
        }
        return cur->is_word_end;
      }

      bool IsPrefix(string prefix) {
        Node* cur = root_;
        for (int i = 0; i < prefix.size(); i++) {
          char c = prefix[i];
          if (cur->next.find(c) == cur->next.end()) {
            return false;
          }
          cur = cur->next[c];
        }
        return true;
      }

      // TODO: 此处实现的Trie是集合的一种，可以将其改造成映射，在每个单词处存储单词的
      // 释义等

      // TODO: valgrind 测试是否有内存泄漏

      // TODO: 利用 Trie 统计词频（感觉很简单，只需要简单修改一下 Node 的定义，增加一个单词
      // 计数器，并且在每次插入当前已有的单词的时候将该单词的计数加 1 即可）

      // TODO: 增加删除操作

      // 总结：
      // 相关习题：208、211、677
      // Trie 的局限性在于对空间的消耗较大，例如如要存储英文单词（均为小写字母），那么需要
      // 单纯存储英文单词的空间的27倍（思考一下怎么来的，你可以的）
      // 为了解决这个问题，衍生出了压缩字典树(Compressed Trie)，但是压缩字典树降低了空间消耗的同时却增加了维
      // 护成本；另外还有Ternary Search Trie以及后缀树（需要自己去搜索扩展）

      // 更多字符串问题：
      // 子串查询：例如使用 KMP、Boyer-Moore、Rabin-Karp 等算法
      // 文件压缩
      // 模式匹配
      // 编译原理
      // DNA也是一个超长的字符串

    private:
      Node* root_;
      int size_;
    };

    #endif  // TRIE_H_

UnionFind
============

.. code-block:: c++
    :linenos:

    #ifndef UNION_FIND_
    #define UNION_FIND_

    #include <iostream>
    #include <cassert>

    // 并查集的时间复杂度很难分析
    // 结论: 近乎O(1)
    // 可以查相关资料

    // Quick Find 实现思路
    // 查操作快，但是并操作慢
    class UnionFindE1 {
    public: 
      UnionFindE1(int n) : count_(n) {
        id_ = new int[n];
        for (int i = 0; i < n; i++) {
          id_[i] = i;
        }
      }

      ~UnionFindE1() { delete[] id_; }

      // O(1)
      int Find(int p) {
        assert(p >= 0 && p < count_);
        return id_[p];
      }
  
      bool IsConnected(int p, int q) { return Find(p) == Find(q); }

      // O(n)  效果很差
      void UnionElements(int p, int q) {
        int p_id = Find(p);
        int q_id = Find(q);

        if (p_id == q_id) return;
    
        // 使p和q所在的两个组的id号相同
        for (int i = 0; i < count_; i++) {
          if (id_[i] == p_id) {
            id_[i] = q_id;
          }
        }
      }

    private:
      int* id_;
      int count_;
    };

    // 常规实现思路：Quick Union
    // 性能非常好
    class UnionFindE2 { 
    public:
      UnionFindE2(int count) : count_(count) {
        parent_ = new int[count];
        for (int i = 0; i < count_; i++) {
          parent_[i] = i;
        }
      }

      ~UnionFindE2() { delete[] parent_; }

      int Find(int p) {
        assert(p >= 0 && p < count_);
        while (parent_[p] != p) p = parent_[p];
        return p;
      }

      bool IsConnected(int p, int q) {
        return Find(p) == Find(q);
      }

      void UnionElements(int p, int q) {
        int p_root = Find(p);
        int q_root = Find(q);
        // 固定将P的根节点指向q的根节点，可能会导致树的高度过高，潜在的优化点，可以记录每个
        // 集合的元素个数，将元素更少的集合的根指向元素更多的集合的根
        // 其实利用元素数量进行优化的方式并不是最优的，因为存在节点数很多但是层数很少的集合，
        // 此时若采用上述的优化方法，仍然可能导致树的高度增大
        // 最优的做法是将层数少的集合的根指向层数多的集合的根
        if (p_root == q_root) return;
    
        parent_[p_root] = q_root;
      }

    private:
      int* parent_;
      int count_;
    };

    // 优化Union
    // 针对size的优化
    class UnionFindE3 { 
    public:
      UnionFindE3(int count) : count_(count) {
        parent_ = new int[count];
        size_ = new int[count];
        for (int i = 0; i < count_; i++) {
          parent_[i] = i;
          size_[i] = 1;
        }
      }

      ~UnionFindE3() {
        delete[] parent_;
        delete[] size_;
      }

      int Find(int p) {
        assert(p >= 0 && p < count_);
        while (parent_[p] != p) p = parent_[p];
        return p;
      }

      bool IsConnected(int p, int q) {
        return Find(p) == Find(q);
      }

      void UnionElements(int p, int q) {
        int p_root = Find(p);
        int q_root = Find(q);

        if (p_root == q_root) return;  // 忘记这一句后果很严重

        if (size_[p_root] < size_[q_root]) {
          parent_[p_root] = q_root;
          size_[q_root] += size_[p_root];  // 不需要更改size_[p_root]吗？此时数字应该变成无效？bobo老师没管
        } else {
          parent_[q_root] = p_root;
          size_[p_root] += size_[q_root];
        }
      }

    private:
      int* parent_;
      int* size_;  // size_[i]表示以i为根的集合中元素的个数
      int count_;
    };

    // 优化Union
    // 基于rank的优化
    // 实现时最好实现这一种
    class UnionFindE4 { 
    public:
      UnionFindE4(int count) : count_(count) {
        parent_ = new int[count];
        rank_ = new int[count];
        for (int i = 0; i < count_; i++) {
          parent_[i] = i;
          rank_[i] = 1;
        }
      }

      ~UnionFindE4() {
        delete[] parent_;
        delete[] rank_;
      }

      int Find(int p) {
        assert(p >= 0 && p < count_);
        while (parent_[p] != p) p = parent_[p];
        return p;
      }

      bool IsConnected(int p, int q) {
        return Find(p) == Find(q);
      }

      void UnionElements(int p, int q) {
        int p_root = Find(p);
        int q_root = Find(q);

        if (p_root == q_root) return;  // 忘记这一句后果很严重

        if (rank_[p_root] < rank_[q_root]) {
          parent_[p_root] = q_root;  // rank不需要维护
        } else if (rank_[p_root] > rank_[q_root]) {
          parent_[q_root] = p_root;  // rank不需要维护
        } else {
          parent_[p_root] = q_root;
          rank_[q_root] += 1;  // 此时需要维护rank
        }
      }

    private:
      int* parent_;
      int* rank_;  // rank_[i]表示以i为根的集合所表示的树的层数
      int count_;
    };

    // 优化Find
    // 这一版性能最好，甚至优于下一版
    class UnionFindE5 { 
    public:
      UnionFindE5(int count) : count_(count) {
        parent_ = new int[count];
        rank_ = new int[count];
        for (int i = 0; i < count_; i++) {
          parent_[i] = i;
          rank_[i] = 1;
        }
      }

      ~UnionFindE5() {
        delete[] parent_;
        delete[] rank_;
      }

      // 优化：路径压缩
      int Find(int p) {
        assert(p >= 0 && p < count_);
        while (parent_[p] != p) {
          parent_[p] = parent_[parent_[p]];  // 优化，只需要这一步，优秀，但是这种优化不彻底
          p = parent_[p];
        }
        return p;
      }

      bool IsConnected(int p, int q) {
        return Find(p) == Find(q);
      }

      void UnionElements(int p, int q) {
        int p_root = Find(p);
        int q_root = Find(q);

        if (p_root == q_root) return;  // 忘记这一句后果很严重

        if (rank_[p_root] < rank_[q_root]) {
          parent_[p_root] = q_root;  // rank不需要维护
        } else if (rank_[p_root] > rank_[q_root]) {
          parent_[q_root] = p_root;  // rank不需要维护
        } else {
          parent_[p_root] = q_root;
          rank_[q_root] += 1;  // 此时需要维护rank
        }
      }

    private:
      int* parent_;
      int* rank_;  // rank_[i]表示以i为根的集合所表示的树的层数
      int count_;
    };

    // 优化：最优路径压缩
    // 在面试时用这种方式实现是最好的，但是这种实现由于涉及到了递归，其性能不如UnionFindE6，
    // 实际使用过程中建议使用UnionFindE5，不过面试过程中可以把所有的优化点都说出来
    class UnionFindE6 { 
    public:
      UnionFindE6(int count) : count_(count) {
        parent_ = new int[count];
        rank_ = new int[count];
        for (int i = 0; i < count_; i++) {
          parent_[i] = i;
          rank_[i] = 1;
        }
      }

      ~UnionFindE6() {
        delete[] parent_;
        delete[] rank_;
      }

      // 优化：最优路径压缩，树只有两层
      int Find(int p) {
        assert(p >= 0 && p < count_);
        // while (parent_[p] != p) {
        //   parent_[p] = parent_[parent_[p]];  // 优化，只需要这一步，优秀，但是这种优化不彻底
        //   p = parent_[p];
        // }

        if (parent_[p] != p) {
          // 下面这种优化理论上是最优的，但是由于涉及到了递归，其实际性能不如上一种优化，
          // 理论和实际不能相提并论
          parent_[p] = Find(parent_[p]);
                                      
        }
        return parent_[p];
      }

      bool IsConnected(int p, int q) {
        return Find(p) == Find(q);
      }

      void UnionElements(int p, int q) {
        int p_root = Find(p);
        int q_root = Find(q);

        if (p_root == q_root) return;  // 忘记这一句后果很严重

        if (rank_[p_root] < rank_[q_root]) {
          parent_[p_root] = q_root;  // rank不需要维护
        } else if (rank_[p_root] > rank_[q_root]) {
          parent_[q_root] = p_root;  // rank不需要维护
        } else {
          parent_[p_root] = q_root;
          rank_[q_root] += 1;  // 此时需要维护rank
        }
      }

    private:
      int* parent_;
      int* rank_;  // rank_[i]表示以i为根的集合所表示的树的层数
      int count_;
    };

    #endif  // UNION_FIND_

Graph
========

.. code-block:: c++
    :linenos:

    #ifndef GRAPH_H_
    #define GRAPH_H_

    #include <vector>
    #include <cassert>
    #include <iostream>
    #include <stack>
    #include <queue>

    // 稠密图
    class DenseGraph {
    public:
      DenseGraph(int num_vertex, bool directed)
          : num_vertex_(num_vertex), num_edge_(0), directed_(directed) {
        for (int i= - 0; i < num_vertex; i++) {
          g_.push_back(std::vector<bool>(num_vertex, false));
        }
      }

      ~DenseGraph() {}

      inline int V() { return num_vertex_; }

      inline int E() { return num_edge_; }

      void AddEdge(int v, int w) {
        assert(v >= 0 && v < num_vertex_);
        assert(w >= 0 && w < num_vertex_);
        if (HasEdge(v, w)) return;  // 不考虑平行边
        g_[v][w] = true;
        if (!directed_) {
          g_[w][v] = true;
        }
        num_edge_++;
      }

      bool HasEdge(int v, int w) {
        assert(v >= 0 && v < num_vertex_);
        assert(w >= 0 && w < num_vertex_);
        return g_[v][w];
      }

      void Show() {
        for (int i = 0; i < num_vertex_; i++) {
          std::cout << "vertext" << i << ":  ";
          for (int j = 0; j < num_vertex_; j++) {
            std::cout << g_[i][j] << "    ";
          }
          std::cout << std::endl;
        }
        std::cout << std::endl;
      }

      // 设置迭代器的原因：
      // (1) 封装了数据g_，防止用户的误操作修改了g_中的数据，不过在面试中可以简单的将g_设
      // 置成public的即可被外界访问
      // (2) 统一了稀疏图和稠密图的接口，向用户隐藏了底层的数据结构
      class Iterator {
      public:
        // G_是引用类型，必须放置在初始化列表中
        Iterator(DenseGraph& graph, int v)
            : G_(graph), v_(v), index_(-1) {}

        int Begin() {
          index_ = -1;
          return Next();
        }

        int Next() {
          for (index_ += 1; index_ < G_.V(); index_++) {
            if (G_.g_[v_][index_]) {
              return index_;
            }
          }
          return -1;
        }

        bool End() {
          return index_ >= G_.V();
        }

      private:
        DenseGraph& G_;
        int v_;
        int index_;
      };

    private:
      int num_edge_;  // 边数
      int num_vertex_;  // 定点数
      bool directed_;  // 有向/无向
      std::vector<std::vector<bool>> g_;
    };

    // 稀疏图
    class SparseGraph {
    public:
      SparseGraph(int num_vertex, bool directed)
          : num_vertex_(num_vertex), num_edge_(0), directed_(directed) {
        for (int i = 0; i < num_vertex; i++) {
          g_.push_back(std::vector<int>());
        }
      }

      ~SparseGraph() {}

      inline int V() { return num_vertex_; }

      inline int E() { return num_edge_; }

  
      // 稀疏图使用邻接表实现的缺点：处理平行边复杂度为O(n)
      // 暂时不管平行边问题，因为如果要处理的话，需要调用HasEdge操作，而该操作是O(n)量级的
      // 将拖慢AddEdge操作，所以这种实现下允许平行边的出现
      // TODO: 好的做法是将所有的边都加入后再一次性将平行边清除
      void AddEdge(int v, int w) {
        assert(v >= 0 && v < num_vertex_);
        assert(w >= 0 && w < num_vertex_);

        g_[v].push_back(w);
        if (v != w && !directed_) {  // 用v != w自环边
          g_[w].push_back(v);
        }
        num_edge_++;
      }

      void Show() {
        for (int i = 0; i < num_vertex_; i++) {
          std::cout << "vertext" << i << ":  ";
          for (int j = 0; j < g_[i].size(); j++) {
            std::cout << g_[i][j] << "    ";
          }
          std::cout << std::endl;
        }
        std::cout << std::endl;
      }

      class Iterator {
      public:
        // G_是引用类型，必须放置在初始化列表中
        Iterator(SparseGraph& graph, int v)
            : G_(graph), v_(v), index_(0) {}

        int Begin() {
          index_ = 0;
          if (G_.g_[v_].size()) {
            return G_.g_[v_][index_];
          }
          return -1;
        }

        int Next() {
          index_++;
          if (index_ < G_.g_[v_].size()) {
            return G_.g_[v_][index_];
          }
          return -1;
        }

        bool End() {
          return index_ >= G_.g_[v_].size();
        }

      private:
        SparseGraph& G_;
        int v_;
        int index_;
      };

    private:
      int num_edge_;  // 边数
      int num_vertex_;  // 定点数
      bool directed_;  // 有向/无向
      std::vector<std::vector<int>> g_;
    };

    // 深度优先遍历和广度优先遍历
    // 两种遍历方式最终都会生成一棵树

    // 求连通分量
    // 用到了深度优先遍历 DFS
    template <typename Graph>
    class Compoment {
    public:
      Compoment(Graph& graph) : G_(graph) {
        visited_ = new bool[G_.V()];
        id_ = new int[G_.V()];
        c_count_ = 0;
        for (int i = 0; i < G_.V(); i++) {
          visited_[i] = false;
          id_[i] = -1;
        }

        // 深度优先遍历
        // TODO: 复杂度理解得不是很清楚，需要再想想
        // 复杂度：
        // 稀疏图(邻接表): O(V + E)
        // 稠密图(邻接矩阵): O(V^2)
        // TODO: 深度优先遍历检测图中是否存在环
        for (int i = 0; i < G_.V(); i++) {
          if (!visited_[i]) {
            DFS(i);
            c_count_++;
          }
        }
      }

      ~Compoment() {
        delete[] visited_;
        delete[] id_;
      }

      inline int Count() { return c_count_; }

      bool IsConnected(int v, int w) {
        assert(v >= 0 && v < G_.V());
        assert(w >= 0 && w < G_.V());
        return id_[v] == id_[w];
      }

    private:
      Graph& G_;
      bool* visited_;
      int* id_;
      int c_count_;

      void DFS(int v) {
        visited_[v] = true;
        id_[v] = c_count_;
        typename Graph::Iterator itor(G_, v);
        for (int i = itor.Begin(); !itor.End(); i = itor.Next()) {
          if (!visited_[i]) {
            DFS(i);
          }
        }
      }
    };

    template <typename Graph>
    class Path {
    public:
      Path(Graph& graph, int s) : G_(graph), s_(s) {
        assert(s >= 0 && s < G_.V());
        visited_ = new bool[G_.V()];
        from_ = new int[G_.V()];
        for (int i = 0; i < G_.V(); i++) {
          visited_[i] = false;
          from_[i] = -1;
        }

        // 寻路算法
        DFS(s);
      }

      ~Path() {
        delete[] visited_;
        delete[] from_;
      }

      bool HasPath(int w) {
        assert(w >= 0 && w < G_.V());
        return visited_[w];
      }

      void GetPath(int w, std::vector<int>& vec) {
        assert(w >= 0 && w < G_.V());
        std::stack<int> s;
        int p = w;
        while (p != -1) {
          s.push(p);
          p = from_[p];
        }
        vec.clear();
        while (!s.empty()) {
          vec.push_back(s.top());
          s.pop();
        }
      }

      void ShowPath(int w) {
        std::vector<int> vec;
        GetPath(w, vec);
        for (int i = 0; i < vec.size(); i++) {
          std::cout << vec[i];
          if (i == vec.size() - 1) {
            std::cout << std::endl;
          } else {
            std::cout << " -> ";
          }
        }
        std::cout << std::endl;
      }

    private:
      Graph& G_;
      int s_;
      bool* visited_;
      int* from_;

      void DFS(int v) {
        visited_[v] = true;
        typename Graph::Iterator itor(G_, v);
        for (int i = itor.Begin(); !itor.End(); i = itor.Next()) {
          if (!visited_[i]) {
            from_[i] = v;
            DFS(i);
          }
        }
      }
    };

    // 求最短路径
    // 用到了广度优先遍历 BFS
    // 最短路径有可能有多条，取决于对各个节点的遍历顺序
    template <typename Graph>
    class ShortestPath {
    public:
      ShortestPath(Graph& graph, int s) : G_(graph), s_(s) {
        // 初始化
        assert(s >= 0 && s < graph.V());
        visited_ = new bool[graph.V()];
        from_ = new int[G_.V()];
        order_ = new int[graph.V()];
        for (int i = 0; i < graph.V(); i++) {
          visited_[i] = false;
          from_[i] = -1;
          order_[i] = -1;
        }

        std::queue<int> q;

        // 无向图最短路径算法
        // 广度优先遍历 BFS
        // 复杂度(需要再仔细思考分析过程)
        // 稀疏图(邻接表): O(V + E)
        // 稠密图(邻接矩阵): O(V^2)
        q.push(s);
        visited_[s] = true;
        order_[s] = 0;
        while (!q.empty()) {
          int v = q.front();
          q.pop();
          typename Graph::Iterator itor(G_, v);
          for (int i = itor.Begin(); !itor.End(); i = itor.Next()) {
            if (!visited_[i]) {
              q.push(i);
              visited_[i] = true;
              from_[i] = v;
              order_[i] = order_[v] + 1;
            }
          }
        }
      }

      ~ShortestPath() {
        delete[] visited_;
        delete[] from_;
        delete[] order_;
      }

      bool HasPath(int w) {
        assert(w >= 0 && w < G_.V());
        return visited_[w];
      }

      void GetPath(int w, std::vector<int>& vec) {
        assert(w >= 0 && w < G_.V());
        std::stack<int> s;
        int p = w;
        while (p != -1) {
          s.push(p);
          p = from_[p];
        }
        vec.clear();
        while (!s.empty()) {
          vec.push_back(s.top());
          s.pop();
        }
      }

      void ShowPath(int w) {
        assert(w >= 0 && w < G_.V());
        std::vector<int> vec;
        GetPath(w, vec);
        for (int i = 0; i < vec.size(); i++) {
          std::cout << vec[i];
          if (i == vec.size() - 1) {
            std::cout << std::endl;
          } else {
            std::cout << " -> ";
          }
        }
        std::cout << std::endl;
      }

      int Length(int w) {
        assert(w >= 0 && w < G_.V());
        return order_[w];
      }


    private:
      Graph& G_;
      int s_;  // source
      bool* visited_;
      int* from_;
      int* order_;  // 从s到每个节点的最短距离
    };

    // TODO: 图的等多的威力
    // flood fill: 可用于抠图、扫雷
    // 走迷宫、生成迷宫
    // 求欧拉路径、哈密顿路径
    // 地图填色问题：保证相邻国家的颜色不同

    #endif  // GRAPH_H_

Weighted Graph
=================

.. code-block:: c++
    :linenos:

    #ifndef WEIGHTED_GRAPH_H_
    #define WEIGHTED_GRAPH_H_

    #include <vector>
    #include <cassert>
    #include <iostream>
    #include <stack>
    #include <queue>

    #include "union_find.h"
    #include "sort.h"

    template<typename Weight>
    class Edge {
    public:
      Edge(int v1, int v2, Weight weight)
          : v1_(v1), v2_(v2), weight_(weight) {}

      Edge() {}
  
      ~Edge() {}

      inline int V1() { return v1_; }

      inline int V2() { return v2_; }

      inline Weight Wt() { return weight_; }

      inline int Other(int x) { return x == v1_ ? v2_ : v1_; }

      friend std::ostream& operator<<(std::ostream& os, const Edge& e) {
        os << e.v1_ << "-" << e.v2_ << ": " << e.weight_;
        return os;
      }

      bool operator<(Edge<Weight>& e) { return weight_ < e.Wt(); }

      bool operator<=(Edge<Weight>& e) { return weight_ <= e.Wt(); }

      bool operator>(Edge<Weight>& e) { return weight_ > e.Wt(); }

      bool operator>=(Edge<Weight>& e) { return weight_ <= e.Wt(); }
   
      bool operator==(Edge<Weight>& e) { return weight_ == e.Wt(); }

    private:
      int v1_, v2_;  // 从v1_指向v2_
      Weight weight_;
    };

    // 有权稠密图
    template <typename Weight>
    class WeightedDenseGraph {
    public:
      WeightedDenseGraph(int num_vertex, bool directed)
          : num_vertex_(num_vertex), num_edge_(0), directed_(directed) {
        for (int i= - 0; i < num_vertex; i++) {
          g_.push_back(std::vector<Edge<Weight>*>(num_vertex, nullptr));
        }
      }

      ~WeightedDenseGraph() {
        for (int i = 0; i < num_vertex_; i++) {
          for (int j = 0; j < num_vertex_; j++) {
            if (g_[i][j] != nullptr) {
              delete g_[i][j];
            }
          }
        }
      }

      inline int V() { return num_vertex_; }

      inline int E() { return num_edge_; }

      void AddEdge(int v, int w, Weight weight) {
        assert(v >= 0 && v < num_vertex_);
        assert(w >= 0 && w < num_vertex_);
        if (HasEdge(v, w)) {
          delete g_[v][w];
          if (!directed_) {
            delete g_[w][v];
          }
          num_edge_--;
        }
        g_[v][w] = new Edge<Weight>(v, w, weight);
        if (!directed_) {
          g_[w][v] = new Edge<Weight>(w, v, weight);;
        }
        num_edge_++;
      }

      bool HasEdge(int v, int w) {
        assert(v >= 0 && v < num_vertex_);
        assert(w >= 0 && w < num_vertex_);
        return g_[v][w] != nullptr;
      }

      void Show() {
        for (int i = 0; i < num_vertex_; i++) {
          // std::cout << "vertext" << i << ":  ";
          for (int j = 0; j < num_vertex_; j++) {
            if (g_[i][j]) {
              std::cout << g_[i][j]->Wt() << "\t";
            } else {
              std::cout << "NULL\t";
            }
          }
          std::cout << std::endl;
        }
        std::cout << std::endl;
      }

      // 设置迭代器的原因：
      // (1) 封装了数据g_，防止用户的误操作修改了g_中的数据，不过在面试中可以简单的将g_设
      // 置成public的即可被外界访问
      // (2) 统一了稀疏图和稠密图的接口，向用户隐藏了底层的数据结构
      class Iterator {
      public:
        // G_是引用类型，必须放置在初始化列表中
        Iterator(DenseGraph& graph, int v)
            : G_(graph), v_(v), index_(-1) {}

        Edge<Weight*> Begin() {
          index_ = -1;
          return Next();
        }

        Edge<Weight*> Next() {
          for (index_ += 1; index_ < G_.V(); index_++) {
            if (G_.g_[v_][index_]) {
              return G_.g_[v_][index_];
            }
          }
          return nullptr;
        }

        bool End() {
          return index_ >= G_.V();
        }

      private:
        DenseGraph& G_;
        int v_;
        int index_;
      };

    private:
      int num_edge_;  // 边数
      int num_vertex_;  // 定点数
      bool directed_;  // 有向/无向
      std::vector<std::vector<Edge<Weight>*>> g_;
    };

    // 有权稀疏图
    template <typename Weight>
    class WeightedSparseGraph {
    public:
      WeightedSparseGraph(int num_vertex, bool directed)
          : num_vertex_(num_vertex), num_edge_(0), directed_(directed) {
        for (int i = 0; i < num_vertex; i++) {
          g_.push_back(std::vector<Edge<Weight>*>());
        }
      }

      ~WeightedSparseGraph() {
        for (int i = 0; i < num_vertex_; i++) {
          for (int j = 0; j < g_[i].size(); j++) {
            delete g_[i][j];
          }
        }
      }

      inline int V() { return num_vertex_; }

      inline int E() { return num_edge_; }

  
      // 稀疏图使用邻接表实现的缺点：处理平行边复杂度为O(n)
      // 暂时不管平行边问题，因为如果要处理的话，需要调用HasEdge操作，而该操作是O(n)量级的
      // 将拖慢AddEdge操作，所以这种实现下允许平行边的出现
      // TODO: 好的做法是将所有的边都加入后再一次性将平行边清除
      void AddEdge(int v, int w, Weight weight) {
        assert(v >= 0 && v < num_vertex_);
        assert(w >= 0 && w < num_vertex_);

        g_[v].push_back(new Edge<Weight>(v, w, weight));
        if (v != w && !directed_) {  // 用v != w自环边
          g_[w].push_back(new Edge<Weight>(w, v, weight));
        }
        num_edge_++;
      }

      bool HasEdge(int v, int w) {
        assert(v >= 0 && v < num_vertex_);
        assert(w >= 0 && w < num_vertex_);

        for (int i = 0; i < g_[v].size(); i++) {
          if (g_[v][i]->Other(v) == w) {
            return true;
          }
        }
    
        return false;
      }

      void Show() {
        for (int i = 0; i < num_vertex_; i++) {
          std::cout << "vertext" << i << ":    ";
          for (int j = 0; j < g_[i].size(); j++) {
            std::cout << "( to:" << g_[i][j]->V2() << ", wt:" << g_[i][j]->Wt() << ")\t";
          }
          std::cout << std::endl;
        }
        std::cout << std::endl;
      }

      class Iterator {
      public:
        // G_是引用类型，必须放置在初始化列表中
        Iterator(WeightedSparseGraph& graph, int v)
            : G_(graph), v_(v), index_(0) {}

        Edge<Weight>* Begin() {
          index_ = 0;
          if (G_.g_[v_].size()) {
            return G_.g_[v_][index_];
          }
          return nullptr;
        }

        Edge<Weight>* Next() {
          index_++;
          if (index_ < G_.g_[v_].size()) {
            return G_.g_[v_][index_];
          }
          return nullptr;
        }

        bool End() {
          return index_ >= G_.g_[v_].size();
        }

      private:
        WeightedSparseGraph& G_;
        int v_;
        int index_;
      };

    private:
      int num_edge_;  // 边数
      int num_vertex_;  // 定点数
      bool directed_;  // 有向/无向
      std::vector<std::vector<Edge<Weight>*>> g_;
    };

    // 有权图的最小生成树问题

    // 把图中的节点分为两部分，称为一个切分(cut)

    // 如果一条边的两个端点分属于一个切分的两边，那么这条边称为横切边(crossing edge)

    // 切分定理:
    // 给定任意切分，横切边中权值最小的边必然属于最小生成树

    // 解法一
    // Lazy Prim
    // 时间复杂度：O(E*log(E))
    template <typename Graph, typename Weight>
    class LazyPrimMST {  // MST: minimum spanning tree
    public:
      LazyPrimMST(Graph& graph)
          : G_(graph), priority_queue_(MinHeap<Edge<Weight>>(graph.E())),
            mst_weight_(0) {
        marked_ = new bool[G_.V()];
        for (int i = 0; i < G_.V(); i++) {
          marked_[i] = false;
        }
        mst_.clear();

        // lazy prim  复杂度：O(E*log(E))，可以通过优化变成O(E*log(V))
        Visit(0);
        while (!priority_queue_.IsEmpty()) {  // O(E)
          Edge<Weight> e = priority_queue_.ExtractRoot();  // O(log(E))
          if (marked_[e.V1()] == marked_[e.V2()]) {  // e不是横切边
            continue;
          }
          mst_.push_back(e);
          if (!marked_[e.V1()]) {
            // Visit()的复杂度不应该是O(E*log(E))吗？为什么lazy prim总
            // 体复杂度分析下来还是O(E*log(E))，不应该是O(E^2*log(E))吗？
            Visit(e.V1());
          } else {
            Visit(e.V2());
          }
        }

        for (int i = 0; i < mst_.size(); i++) {
          mst_weight_ += mst_[i].Wt();
        }
      }

      ~LazyPrimMST() { delete marked_; }

      inline std::vector<Edge<Weight>> MSTEdges() { return mst_; }
  
      Weight Result() { return mst_weight_; }

    private:
      Graph& G_;
      MinHeap<Edge<Weight>> priority_queue_;
      bool* marked_;
      std::vector<Edge<Weight>> mst_;
      Weight mst_weight_;

      void Visit(int v) {   
        assert(!marked_[v]);
        marked_[v] = true;
        typename Graph::Iterator itor(G_, v);
        // 邻接表存储下，for循环复杂度为O(E)，邻接矩阵存储下，复杂度为O(V^2)，但是邻接矩阵一般用于
        // 稠密图，这种情况下，V^2和E是同一个量级的，所以总体复杂度是O(E)
        for (Edge<Weight>* e = itor.Begin(); !itor.End(); e = itor.Next()) {
          if (!marked_[e->Other(v)]) {  // 找到了一个横切边
            priority_queue_.Insert(*e);  // O(log(E))
          }
        }
      }
    };

    // 解法二
    // prim算法，性能相比lazy prim有很大提升，复杂度从O(E*log(E))下降到O(E*log(V))
    template <typename Graph, typename Weight>
    class PrimMST {
    public:
      PrimMST(Graph& graph)
          : G_(graph), index_priority_queue_(IndexMinHeap<double>(graph.V())) {
        marked_ = new bool[G_.V()];
        for (int i = 0; i < G_.V(); i++) {
          marked_[i] = false;
          edge_to_.push_back(nullptr);
        }
        mst_.clear();

        // prim
        Visit(0);
        while (!index_priority_queue_.IsEmpty()) {
          int v = index_priority_queue_.ExtractRootIndex();
          assert(edge_to_[v]);
          mst_.push_back(*edge_to_[v]);
          Visit(v);
        }

        for (int i = 0; i < mst_.size(); i++) {
          mst_weight_ += mst_[i].Wt();
        }
      }

      ~PrimMST() { delete[] marked_; }

      inline std::vector<Edge<Weight>> MSTEdges() { return mst_; }
  
      Weight Result() { return mst_weight_; }

    private:
      Graph& G_;
      IndexMinHeap<Weight> index_priority_queue_;  // different from [lazy prim]
      std::vector<Edge<Weight>*> edge_to_;  // different from [lazy prim]
      bool* marked_;
      std::vector<Edge<Weight>> mst_;
      Weight mst_weight_;

      void Visit(int v) {
        assert(!marked_[v]);
        marked_[v] = true;

        typename Graph::Iterator itor(G_, v);
        for (Edge<Weight>* e = itor.Begin(); !itor.End(); e = itor.Next()) {
          int w = e->Other(v);
          if (!marked_[w]) {
            if (!edge_to_[w]) {
              index_priority_queue_.Insert(w, e->Wt());
              edge_to_[w] = e;
            } else if (e->Wt() < edge_to_[w]->Wt()) {
              edge_to_[w] = e;
              index_priority_queue_.ChangeE2(w, e->Wt());
            }
          }
        }
      }
    };

    // 解法三
    // kruskal算法
    // 性能没有prim算法好，其时间复杂度为O(E*log(E) + E*log(V))  (排序+判断是否性能环)
    // 但是kruskal算法思路简单、实现容易，对于较小的图可以利用kruskal算法进行快速实现
    template <typename Graph, typename Weight>
    class KruskalMST {
    public:
      KruskalMST(Graph& graph) {
        MinHeap<Edge<Weight>> priority_queue(graph.E());
        for (int i = 0; i < graph.V(); i++) {
          typename Graph::Iterator itor(graph, i);
          for (Edge<Weight>* e = itor.Begin(); !itor.End(); e = itor.Next()) {
            // 无向图中的同一条边存入了两次，这里进行处理，防止priority_queue中也存入两次
            if (e->V1() < e->V2()) {
              priority_queue.Insert(*e);
            }
          }
        }

        UnionFindE6 uf(graph.V());
    
        while (!priority_queue.IsEmpty() && mst_.size() < graph.V() - 1) {
          Edge<Weight> e = priority_queue.ExtractRoot();
          if (uf.IsConnected(e.V1(), e.V2())) continue;
          mst_.push_back(e);
          uf.UnionElements(e.V1(), e.V2());
        }

        for (int i = 0; i < mst_.size(); i++) {
          mst_weight_ += mst_[i].Wt();
        }
      }

      ~KruskalMST() {}

      inline std::vector<Edge<Weight>> MSTEdges() { return mst_; }
  
      Weight Result() { return mst_weight_; }

    private:
      std::vector<Edge<Weight>> mst_;
      Weight mst_weight_;
    };

    // 有权图的最短路径问题
    // 单源最短路径算法之 --- dijkstra算法
    // 时间复杂度：O(E*log(V))
    // 即(遍历每条边O(E))*(在IndexMinHeap中进行插入和修改操作O(log(V))
    // 前提：图中不能有负权边
    template <typename Graph, typename Weight>
    class Dijkstra {
    public:
      Dijkstra(Graph& graph, int s) : G_(graph), s_(s) {
        dist_to_ = new Weight[G_.V()];
        marked_ = new bool[G_.V()];
        for (int i = 0; i < G_.V(); i++) {
          dist_to_[i] = Weight();  // int or double
          marked_[i] = false;
          from_.push_back(nullptr);
        }

        IndexMinHeap<Weight> index_min_heap(G_.V());

        // dijkstra
        dist_to_[s] = Weight();
        marked_[s] = true;
        index_min_heap.Insert(s, dist_to_[s]);
        while (!index_min_heap.IsEmpty()) {
          int v = index_min_heap.ExtractRootIndex();
      
          // dist_to_[v]就是s到v的最短距离
          marked_[v] = true;
      
          // 松弛操作 Relaxation (核心操作)
          typename Graph::Iterator itor(graph, v);
          for (Edge<Weight>* e = itor.Begin(); !itor.End(); e = itor.Next()) {
            int w = e->Other(v);
            if (!marked_[w]) {
              // from_[w] == nullptr是为了判断节点是否被访问过，传统的做法是将dist_to_
              // 数组种的元素全部初始化成正无穷，由于正无穷不好表示，所以这里利用from_数组
              // 判断节点是否被访问过，从而绕过了正无穷的问题。如果初始值设置为正无穷的话，
              // 是不需要判断from_[w] == nullptr的
              if (from_[w] == nullptr || dist_to_[v] + e->Wt() < dist_to_[w]) {
                dist_to_[w] = dist_to_[v] + e->Wt();
                from_[w] = e;
                if (index_min_heap.Contain(w)) {
                 index_min_heap.ChangeE2(w, dist_to_[w]);
                } else {
                 index_min_heap.Insert(w, dist_to_[w]);
                }
              }
            }
          }
        }

      }

      ~Dijkstra() {
        delete[] dist_to_;
        delete[] marked_;
      }

      Weight ShortestPathTo(int w) {
        return dist_to_[w];
      }

      bool HasPahtTo(int w) {
        return marked_[w];
      }

      Weight ShortestPath(int w, std::vector<Edge<Weight>>& vec) {
        // std::cout << "w: " << w << std::endl;
    
        std::stack<Edge<Weight>*> s;
        Edge<Weight>* e = from_[w];
        // bobo老师给的代码是 while (e->V1() != e->V2())存在bug，我修改成了while (e)
        while (e) {
          // std::cout << e->V1() << "  " << e->V2() << std::endl;
          s.push(e);
          e = from_[e->V1()];
        }

        while (!s.empty()) {
          e = s.top();
          vec.push_back(*e);
          s.pop();
        }
      }

      void ShowPath(int w) {
        assert(w >= 0 && w < G_.V());

        std::vector<Edge<Weight>> vec;
        ShortestPath(w, vec);
        for (int i = 0; i < vec.size(); i++) {
          std::cout << vec[i].V1() << "->";
          if (i == vec.size() - 1) {
            std::cout << vec[i].V2() << std::endl;
          }
        }
      }

    private:
      Graph& G_;
      int s_;  // 源点
      Weight* dist_to_;  // 源点s到每个点的最短距离
      bool* marked_;
      std::vector<Edge<Weight>*> from_;
    };

    // 单源最短路径算法之 --- Bellmam-Ford算法(可以解决图中存在负权边的情况)
    // 图中可以存在负权边，但是不能存在负权环(如果一个图中有负权环，那么一定不存在最短路径，
    // 因为经过一次负权环，路径都可以较小，直至趋于负无穷，两个节点也可以形成负权环)
    // 但是，没有负权环并不是bellman-ford算法的必要前提（和二分查找时数字序列必须有序不同），
    // 当存在负权环时，bellman-ford算法可以自行判断(虽然在这种情况下找不到最短路径)
    // 复杂度：O(E*V)，比dijkstra的复杂度高
    // 对于存在负权边的图来说，如果要对该图求最短路径，那么该图一定是一个有向图，否则，只要
    // 任意一条边来说，只要权值为负，那么该条边一定会形成一个负权环，就无法求出最短路径了。
    // 对于没有负权边的图来说，直接使用dijkstra算法即可
    // Bellman Ford可以利用queue可以进行优化，即queue-based bellman-ford，需要自己查资料
    // 进行优化，效果很显著
    template <typename Graph, typename Weight>
    class BellmanFord {
    public:
      BellmanFord(Graph& graph, int s) : G_(graph), s_(s) {
        dist_to_ = new Weight(G_.V());
        for (int i= 0; i < G_.V(); i++) {
          from_.push_back(nullptr);
        }

        // bellman ford
        dist_to_[s] = Weight();
        from_[s] = new Edge<Weight>();  // 这么玩儿怎么感觉怪怪的

        // Relaxation
        for (int pass = 1; pass < G_.V(); pass++) {
          for (int i = 0; i < G_.V(); i++) {
            typename Graph::Iterator itor(graph, i);
            for (Edge<Weight>* e = itor.Begin(); !itor.End(); e = itor.Next()) {
              if (from_[e->V1()] && (!from_[e->V2()] || dist_to_[e->V1()] + e->Wt() < dist_to_[e->V2()])) {
                dist_to_[e->V2()] = dist_to_[e->V1()] + e->Wt();
                from_[e->V2()] = e;
              }
            }
          }
        }

        has_negative_cycle_ = DetectNegativeCycle();
      }

      ~BellmanFord() {
        delete[] dist_to_;
        delete from_[s_];  // 这里也怪怪的
      }

      inline bool NegativeCycle() { return has_negative_cycle_; }

      Weight ShortestPathTo(int w) {
        assert(w >= 0 && w < G_.V());
        assert(!has_negative_cycle_);
        return dist_to_[w];
      }

      bool HasPathTo(int w) {
        assert(w >= 0 && w < G_.V());
        return from_[w] != nullptr;
      }

      Weight ShortestPath(int w, std::vector<Edge<Weight>>& vec) {
        assert(w >= 0 && w < G_.V());
        assert(!has_negative_cycle_);
    
        std::stack<Edge<Weight>*> s;
        Edge<Weight>* e = from_[w];
        while (e->V1() != s_) {  // 这里会不会有问题？需要仔细想想
          s.push(e);
          e = from_[e->V1()];
        }
        s.push(e);

        while (!s.empty()) {
          e = s.top();
          vec.push_back(*e);
          s.pop();
        }
      }

      void ShowPath(int w) {
        assert(w >= 0 && w < G_.V());
        assert(!has_negative_cycle_);

        std::vector<Edge<Weight>> vec;
        ShortestPath(w, vec);
        for (int i = 0; i < vec.size(); i++) {
          std::cout << vec[i].V1() << "->";
          if (i == vec.size() - 1) {
            std::cout << vec[i].V2() << std::endl;
          }
        }
      }

    private:
      Graph& G_;
      int s_;
      Weight* dist_to_;
      std::vector<Edge<Weight>*> from_;
      bool has_negative_cycle_;

      bool DetectNegativeCycle() {
        for (int i = 0; i < G_.V(); i++) {
          typename Graph::Iterator itor(G_, i);
          for (Edge<Weight>* e = itor.Begin(); !itor.End(); e = itor.Next()) {
            if (from_[e->V1()] && (!from_[e->V2()] || dist_to_[e->V1()] + e->Wt() < dist_to_[e->V2()])) {
              return true;
            }
          }
        }

        return false;
      }
    };

    // 对于有向无环图(DAG)来说，可以使用拓扑排序进行处理，复杂度O(V+E)
    // 需要自己查资料

    // 所有对最短路径算法
    // 可以求出所有节点到其他节点的最短路径
    // 实现方式可以是将dijkstra算法或者bellman-ford算法运行V次
    // 或者使用Floyed算法，时间复杂度O(V^3)，性能相对更好，使用的动态规划的策略
    // 前提条件：无负权环

    // 最长路径算法
    // 不能有正权环
    // 无权图的最长路径问题是指数级的难度
    // 对于有权图，不能使用Dijkstra求最长路径问题
    // 但是可以使用bellman-Ford算法，将所有的路径都取负即可，但是不能有正权环
    // 拓扑排序处理DAG以及求所有节点最短的路径的Floyed算法也可以经过改造用在求最长路径问题

    #endif  // WEIGHTED_GRAPH_H_