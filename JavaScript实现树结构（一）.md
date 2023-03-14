## JavaScript实现树结构（一）

### 一、树结构简介

#### 1.1.简单了解树结构

**什么是树？**

真实的树：

[![image-20200229205530929](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/1.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/1.png)

image-20200229205530929



**树的特点：**

- 树一般都有一个**根**，连接着根的是**树干**；
- 树干会发生分叉，形成许多**树枝**，树枝会继续分化成更小的**树枝**；
- 树枝的最后是**叶子**；

现实生活中很多结构都是树的抽象，模拟的树结构相当于旋转`180°`的树。

[![image-20200229205630945](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/2.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/2.png)

image-20200229205630945



**树结构对比于数组/链表/哈希表有哪些优势呢：**

**数组：**

- 优点：可以通过**下标值访问**，效率高；
- 缺点：查找数据时需要先对数据进行**排序**，生成**有序数组**，才能提高查找效率；并且在插入和删除元素时，需要大量的**位移操作**；

**链表：**

- 优点：数据的插入和删除操作效率都很高；
- 缺点：**查找**效率低，需要从头开始依次查找，直到找到目标数据为止；当需要在链表中间位置插入或删除数据时，插入或删除的效率都不高。

**哈希表：**

- 优点：哈希表的插入/查询/删除效率都非常高；
- 缺点：**空间利用率不高**，底层使用的数组中很多单元没有被利用；并且哈希表中的元素是**无序**的，不能按照固定顺序遍历哈希表中的元素；而且不能快速找出哈希表中**最大值或最小值**这些特殊值。

**树结构：**

优点：树结构综合了上述三种结构的优点，同时也弥补了它们存在的缺点（虽然效率不一定都比它们高），比如树结构中数据都是有序的，查找效率高；空间利用率高；并且可以快速获取最大值和最小值等。

总的来说：**每种数据结构都有自己特定的应用场景**

**树结构：**

- **树（Tree）**:由 n（n ≥ 0）个节点构成的**有限集合**。当 n = 0 时，称为**空树**。

对于任一棵非空树（n > 0），它具备以下性质：

- 数中有一个称为**根（Root）**的特殊节点，用 **r **表示；
- 其余节点可分为 m（m > 0）个互不相交的有限集合 T1，T2，...，Tm，其中每个集合本身又是一棵树，称为原来树的**子树（SubTree）**。

**树的常用术语：**

[![image-20200229221126468](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/3.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/3.png)

image-20200229221126468



- **节点的度（Degree）**：节点的**子树个数**，比如节点B的度为2；
- **树的度**：树的所有节点中**最大的度数**，如上图树的度为2；
- **叶节点（Leaf）**：**度为0的节点**（也称为叶子节点），如上图的H，I等；
- **父节点（Parent）**：度不为0的节点称为父节点，如上图节点B是节点D和E的父节点；
- **子节点（Child）**：若B是D的父节点，那么D就是B的子节点；
- **兄弟节点（Sibling）**：具有同一父节点的各节点彼此是兄弟节点，比如上图的B和C，D和E互为兄弟节点；
- **路径和路径长度**：路径指的是一个节点到另一节点的通道，路径所包含边的个数称为路径长度，比如A->H的路径长度为3；
- **节点的层次（Level）**：规定**根节点在1层**，其他任一节点的层数是其父节点的**层数加1**。如B和C节点的层次为2；
- **树的深度（Depth）**：树种所有节点中的**最大层次**是这棵树的深度，如上图树的深度为4；

#### 1.2.树结构的表示方式

- **最普通的表示方法**：

[![image-20200229230417613](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/4.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/4.png)

image-20200229230417613



如图，树结构的组成方式类似于链表，都是由一个个节点连接构成。不过，根据每个父节点子节点数量的不同，每一个父节点需要的引用数量也不同。比如节点A需要3个引用，分别指向子节点B，C，D；B节点需要2个引用，分别指向子节点E和F；K节点由于没有子节点，所以不需要引用。

这种方法缺点在于我们无法确定某一结点的引用数。

- **儿子-兄弟表示法**：

[![image-20200229232805477](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/5.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/5.png)

image-20200229232805477



这种表示方法可以完整地记录每个节点的数据，比如：

```kotlin
//节点A
Node{
  //存储数据
  this.data = data
  //统一只记录左边的子节点
  this.leftChild = B
  //统一只记录右边的第一个兄弟节点
  this.rightSibling = null
}

//节点B
Node{
  this.data = data
  this.leftChild = E
  this.rightSibling = C
}

//节点F
Node{
  this.data = data
  this.leftChild = null
  this.rightSibling = null
}
```

这种表示法的优点在于每一个节点中引用的数量都是确定的。

- **儿子-兄弟表示法旋转**

以下为儿子-兄弟表示法组成的树结构：

[![image-20200229234549049](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/6.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/6.png)

image-20200229234549049



将其顺时针旋转45°之后：

[![image-20200229235549522](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/7.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/7.png)

image-20200229235549522



这样就成为了一棵**二叉树**，由此我们可以得出结论：**任何树都可以通过二叉树进行模拟**。但是这样父节点不是变了吗？其实，父节点的设置只是为了方便指向子节点，在代码实现中谁是父节点并没有关系，只要能正确找到对应节点即可。

### 二、二叉树

#### 2.1.二叉树简介

**二叉树的概念**：如果树中的每一个节点最多只能由**两个子节点**，这样的树就称为**二叉树**；

二叉树十分重要，不仅仅是因为简单，更是因为几乎所有的树都可以表示成二叉树形式。

**二叉树的组成**：

- 二叉树可以为空，也就是没有节点；
- 若二叉树不为空，则它由根节点和称为其左子树TL和右子树TR的两个不相交的二叉树组成；

**二叉树的五种形态**：

[![image-20200301001718079](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/8.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/8.png)

image-20200301001718079



上图分别表示：空的二叉树、只有一个节点的二叉树、只有左子树TL的二叉树、只有右子树TR的二叉树和有左右两个子树的二叉树。

**二叉树的特性**：

- 一个二叉树的第 i 层的最大节点树为：2(i-1)，i >= 1；
- 深度为k的二叉树的最大节点总数为：2k - 1 ，k >= 1；
- 对任何非空二叉树，若 n0 表示叶子节点的个数，n2表示度为2的非叶子节点个数，那么两者满足关系：n0 = n2 + 1；如下图所示：H，E，I，J，G为叶子节点，总数为5；A，B，C，F为度为2的非叶子节点，总数为4；满足n0 = n2 + 1的规律。

[![image-20200301092140211](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/9.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/9.png)

image-20200301092140211



#### 2.2.特殊的二叉树

**完美二叉树**

完美二叉树（Perfect Binary Tree）也成为满二叉树（Full Binary Tree），在二叉树中，除了最下一层的叶子节点外，每层节点都有2个子节点，这就构成了完美二叉树。

[![image-20200301093237681](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/10.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/10.png)

image-20200301093237681



**完全二叉树**

完全二叉树（Complete Binary Tree）:

- 除了二叉树最后一层外，其他各层的节点数都达到了最大值；
- 并且，最后一层的叶子节点从左向右是连续存在，只缺失右侧若干叶子节点；
- 完美二叉树是特殊的完全二叉树；

[![image-20200301093659373](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/11.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/11.png)

image-20200301093659373



在上图中，由于H缺失了右子节点，所以它不是完全二叉树。

#### 2.3.二叉树的数据存储

常见的二叉树存储方式为**数组**和**链表**：

**使用数组：**

- **完全二叉树**：按从上到下，从左到右的方式存储数据。

[![image-20200301094919588](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/12.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/12.png)

image-20200301094919588



| 节点     | A     | B     | C     | D     | E     | F     | G     | H     |
| -------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| **序号** | **1** | **2** | **3** | **4** | **5** | **6** | **7** | **8** |

使用数组存储时，取数据的时候也十分方便：左子节点的序号等于父节点序号 * 2，右子节点的序号等于父节点序号 * 2 + 1 。

- **非完全二叉树**：非完全二叉树需要转换成完全二叉树才能按照上面的方案存储，这样会浪费很大的存储空间。

[![image-20200301100043636](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/13.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/13.png)

image-20200301100043636



| 节点     | A     | B     | C     | ^     | ^     | F     | ^     | ^     | ^     | ^      | ^      | ^      | M      |
| -------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ------ | ------ | ------ | ------ |
| **序号** | **1** | **2** | **3** | **4** | **5** | **6** | **7** | **8** | **9** | **10** | **11** | **12** | **13** |

**使用链表**

二叉树最常见的存储方式为**链表**：每一个节点封装成一个Node，Node中包含存储的数据、左节点的引用和右节点的引用。

[![image-20200301100616105](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/14.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/14.png)

image-20200301100616105



### 三、二叉搜索树

#### 3.1.认识二叉搜索树

**二叉搜索树**（**BST**，Binary Search Tree），也称为**二叉排序树**和**二叉查找树**。

二叉搜索树是一棵二叉树，可以为空；

如果不为空，则满足以下**性质**：

- 条件1：非空左子树的**所有**键值**小于**其根节点的键值。比如三中节点6的所有非空左子树的键值都小于6；
- 条件2：非空右子树的**所有**键值**大于**其根节点的键值；比如三中节点6的所有非空右子树的键值都大于6；
- 条件3：左、右子树本身也都是二叉搜索树；

[![image-20200301103139916](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/15.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/15.png)

image-20200301103139916



如上图所示，树二和树三符合3个条件属于二叉树，树一不满足条件3所以不是二叉树。

**总结：**二叉搜索树的特点主要是**较小的值**总是保存在**左节点**上，相对**较大的值**总是保存在**右节点**上。这种特点使得二叉搜索树的查询效率非常高，这也就是二叉搜索树中"搜索"的来源。

#### 3.2.二叉搜索树应用举例

下面是一个二叉搜索树：

[![image-20200301111718686](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/16.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/16.png)

image-20200301111718686



若想在其中查找数据10，只需要查找4次，查找效率非常高。

- 第1次：将10与根节点9进行比较，由于10 > 9，所以10下一步与根节点9的右子节点13比较；
- 第2次：由于10 < 13，所以10下一步与父节点13的左子节点11比较；
- 第3次：由于10 < 11，所以10下一步与父节点11的左子节点10比较；
- 第4次：由于10 = 10，最终查找到数据10 。

[![image-20200301111751041](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/17.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/17.png)

image-20200301111751041

同样是15个数据，在排序好的数组中查询数据10，需要查询10次：



[![image-20200301115348138](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%B8%80/18.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树一/18.png)

image-20200301115348138



其实：如果是排序好的数组，可以通过二分查找：第一次找9，第二次找13，第三次找15...。我们发现如果把每次二分的数据拿出来以树的形式表示的话就是**二叉搜索树**。这就是数组二分法查找效率之所以高的原因。

## JavaScript实现树结构（二）

### 一、二叉搜索树的封装

**二叉树搜索树的基本属性**：

如图所示：二叉搜索树有四个最基本的属性：指向节点的**根**（root），节点中的**键**（key）、**左指针**（right）、**右指针**（right）。

[![image-20200301162706755](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/1.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/1.png)

image-20200301162706755



所以，二叉搜索树中除了定义root属性外，还应定义一个节点内部类，里面包含每个节点中的left、right和key三个属性：

```javascript
    //封装二叉搜索树
    function BinarySearchTree(){

      //节点内部类
      function Node(key){
        this.key = key
        this.left = null
        this.right = null
      }

      //属性
      this.root = null
  }
```

**二叉搜索树的常见操作：**

- insert（key）：向树中插入一个新的键；
- search（key）：在树中查找一个键，如果节点存在，则返回true；如果不存在，则返回false；
- inOrderTraverse：通过中序遍历方式遍历所有节点；
- preOrderTraverse：通过先序遍历方式遍历所有节点；
- postOrderTraverse：通过后序遍历方式遍历所有节点；
- min：返回树中最小的值/键；
- max：返回树中最大的值/键；
- remove（key）：从树中移除某个键；

#### 1.插入数据

**实现思路：**

- 首先根据传入的key创建节点对象；
- 然后判断根节点是否存在，不存在时通过：this.root = newNode，直接把新节点作为二叉搜索树的根节点。
- 若存在根节点则重新定义一个内部方法insertNode（）用于查找插入点。

```JavaScript
     //insert方法:对外向用户暴露的方法
      BinarySearchTree.prototype.insert = function(key){
        //1.根据key创建节点
        let newNode = new Node(key)
          
        //2.判断根节点是否存在
        if (this.root == null) {
          this.root = newNode
          //根节点存在时
        }else {
          this.insertNode(this.root, newNode)
        }
      }
```

**内部方法insertNode（）的实现思路**:

根据比较传入的两个节点，一直查找新节点适合插入的位置，直到成功插入新节点为止。

当newNode.key < node.key向左查找:

- 情况1：当node无左子节点时，直接插入：
- 情况2：当node有左子节点时，递归调用insertNode(),直到遇到无左子节点成功插入newNode后，不再符合该情况，也就不再调用insertNode()，递归停止。

[![image-20200301191640632](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/2.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/2.png)

image-20200301191640632



当newNode.key >= node.key向右查找，与向左查找类似：

- 情况1：当node无右子节点时，直接插入：
- 情况2：当node有右子节点时，依然递归调用insertNode(),直到遇到传入insertNode方法的node无右子节点成功插入newNode为止：

[![image-20200301191507181](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/3.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/3.png)

image-20200301191507181



**insertNode()代码实现：**

```javascript
      //内部使用的insertNode方法:用于比较节点从左边插入还是右边插入
      BinarySearchTree.prototype.insertNode = function(node, newNode){
        //当newNode.key < node.key向左查找
/*----------------------分支1:向左查找--------------------------*/      
        if(newNode.key < node.key){
          //情况1：node无左子节点，直接插入
/*----------------------分支1.1--------------------------*/
          if (node.left == null) {
            node.left = newNode
          //情况2：node有左子节点，递归调用insertNode(),直到遇到无左子节点成功插入newNode后，不再符合该情况，也就不再调用insertNode()，递归停止。
/*----------------------分支1.2--------------------------*/
          }else{
            this.insertNode(node.left, newNode)
          }
        //当newNode.key >= node.key向右查找
/*-----------------------分支2:向右查找--------------------------*/        
        }else{
          //情况1：node无右子节点，直接插入
/*-----------------------分支2.1--------------------------*/ 
          if(node.right == null){
            node.right == newNode
          //情况2：node有右子节点，依然递归调用insertNode(),直到遇到无右子节点成功插入newNode为止
/*-----------------------分支2.2--------------------------*/ 
          }else{
            this.insertNode(node.right, newNode)
          }
        }
      }
```

**过程详解：**

为了更好理解以下列二叉搜索树为例：

[![image-20200301193104003](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/4.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/4.png)

image-20200301193104003



想要上述的二叉搜索树（蓝色）中插入数据10：

- 先把key = 10 传入insert方法，由于存在根节点 9，所以直接调用insetNode方法，传入的参数：node = 9，newNode = 10；
- 由于10 > 9，进入分支2，向右查找适合插入的位置；
- 由于根节点 9 的右子节点存在且为 13 ，所以进入分支2.2，递归调用insertNode方法，传入的参数：node = 13，newNode = 10；
- 由于 10 < 13 ，进入分支1，向左查找适合插入的位置；
- 由于父节点 13 的左子节点存在且为11，所以进入分支1.2，递归调用insertNode方法，传入的参数：node = 11，newNode = 10；
- 由于 10 < 11，进入分支1，向左查找适合插入的位置；
- 由于父节点 11 的左子节点不存在，所以进入分支1.1，成功插入节点 10 。由于不符合分支1.2的条件所以不会继续调用insertNode方法，递归停止。

**测试代码：**

```avrasm
    //测试代码
    //1.创建BinarySearchTree
    let bst = new BinarySearchTree()

    //2.插入数据
    bst.insert(11);
    bst.insert(7);
    bst.insert(15);
    bst.insert(5);
    bst.insert(9);
	console.log(bst);
```

应得到下图所示的二叉搜索树：

[![image-20200302002708576](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/5.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/5.png)

image-20200302002708576



**测试结果**

[![image-20200302002409735](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/6.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/6.png)

image-20200302002409735



#### 2.遍历数据

这里所说的树的遍历不仅仅针对二叉搜索树，而是适用于所有的二叉树。由于树结构不是线性结构，所以遍历方式有多种选择，常见的三种二叉树遍历方式为：

- 先序遍历；
- 中序遍历；
- 后序遍历；

还有层序遍历，使用较少。

##### 2.1.先序遍历

先序遍历的过程为：

- 首先，遍历根节点；
- 然后，遍历其左子树；
- 最后，遍历其右子树；

[![image-20200301213506159](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/7.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/7.png)

image-20200301213506159



如上图所示，二叉树的节点遍历顺序为：A -> B -> D -> H -> I -> E -> C -> F -> G。

**代码实现：**

```javascript
	  //先序遍历
      //掺入一个handler函数方便之后对得到的key进行处理
      BinarySearchTree.prototype.preOrderTraversal = function(handler){
        this.preOrderTraversalNode(this.root, handler)
      }

      //封装内部方法，对某个节点进行遍历
      BinarySearchTree.prototype.preOrderTraversalNode = function(node,handler){
        if (node != null) {
          //1.处理经过的节点
          handler(node.key)
/*----------------------递归1----------------------------*/
          //2.遍历左子树中的节点
          this.preOrderTraversalNode(node.left, handler)
/*----------------------递归2----------------------------*/
          //3.遍历右子树中的节点
          this.preOrderTraversalNode(node.right, handler)
        }
      }
```

**过程详解：**

以遍历以下二叉搜索树为例：

[![image-20200301221450001](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/8.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/8.png)

image-20200301221450001



首先调用preOrderTraversal方法，在方法里再调用preOrderTraversalNode方法用于遍历二叉搜索树。在preOrderTraversalNode方法中，递归1负责遍历左子节点，递归2负责遍历右子节点。先执行递归1，执行过程如下图所示：

> **记：preOrderTraversalNode() 为 A()**

[![image-20200302000248291](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/9.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/9.png)

image-20200302000248291



可以看到一共递归调用了4次方法A，分别传入11、7、5、3，最后遇到null不满足 node != null 条件结束递归1；注意此时只是执行完最开始的递归1，并没有执行递归2，并且递归1执行到null停止后要一层层地往上返回，按顺序将调用的函数压出函数调用栈。

关于函数调用栈：之前的四次递归共把4个函数压入了函数调用栈，现在递归执行完了一层层地把函数压出栈。

值得注意的是：每一层函数都只是执行完了递归1，当返回到该层函数时，比如A（3）要继续执行递归2遍历二叉搜索树中的右子节点；

在执行递归2的过程中会不断调用方法A，并依次执行递归1和递归2，以此类推直到遇到null不满足 node != null 条件为止，才停止递归并一层层返回，如此循环。同理A（5）层、A（7）层、A（11）层都要经历上述循环，直到将二叉搜索树中的节点全部遍历完为止。

具体过程如下图所示：

[![image-20200302000007414](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/10.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/10.png)

image-20200302000007414



**测试代码：**

```javascript
    //测试代码
    //1.创建BinarySearchTree
    let bst = new BinarySearchTree()

    //2.插入数据
    bst.insert(11);
    bst.insert(7);
    bst.insert(15);
    bst.insert(5);
    bst.insert(3);
    bst.insert(9);
    bst.insert(8);
    bst.insert(10);
    bst.insert(13);
    bst.insert(12);
    bst.insert(14);
    bst.insert(20);
    bst.insert(18);
    bst.insert(25);
    bst.insert(6);
    
    //3.测试遍历
    let resultString = ""
    //掺入处理节点值的处理函数
    bst.preOrderTraversal(function(key){
      resultString += key + "->"
    })
    alert(resultString)
```

应输出这样的顺序：11 -> 7 -> 5 -> 3 -> 6 -> 9 -> 8 -> 10 -> 15 -> 13 ->12 -> 14 -> 20 -> 18 -> 25 。

**测试结果：**

[![image-20200302003244874](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/11.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/11.png)

image-20200302003244874



##### 2.2.中序遍历

实现思路：与先序遍历原理相同，只不过是遍历的顺序不一样了。

- 首先，遍历其左子树；
- 然后，遍历根（父）节点；
- 最后，遍历其右子树；

**代码实现：**

```//2.中序遍历
      //中序遍历
      BinarySearchTree.prototype.midOrderTraversal = function(handler){
        this.midOrderTraversalNode(this.root, handler)
      }

      BinarySearchTree.prototype.midOrderTraversalNode = function(node, handler){
        if (node != null) {
          //1.遍历左子树中的节点
          this.midOrderTraversalNode(node.left, handler)
          
          //2.处理节点
          handler(node.key)

          //3.遍历右子树中的节点
          this.midOrderTraversalNode(node.right, handler)
        }
      }
```

**过程详解：**

遍历的顺序应如下图所示：

[![image-20200302112920295](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/12.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/12.png)

image-20200302112920295



首先调用midOrderTraversal方法，在方法里再调用midOrderTraversalNode方法用于遍历二叉搜索树。先使用递归1遍历左子树中的节点；然后，处理父节点；最后，遍历右子树中的节点。

**测试代码：**

```javascript
  //测试代码
    //1.创建BinarySearchTree
    let bst = new BinarySearchTree()

    //2.插入数据
    bst.insert(11);
    bst.insert(7);
    bst.insert(15);
    bst.insert(5);
    bst.insert(3);
    bst.insert(9);
    bst.insert(8);
    bst.insert(10);
    bst.insert(13);
    bst.insert(12);
    bst.insert(14);
    bst.insert(20);
    bst.insert(18);
    bst.insert(25);
    bst.insert(6);	
    
    //3.测试中序遍历
    let resultString2 =""
    bst.midOrderTraversal(function(key){
      resultString2 += key + "->"
    })
    alert(resultString2)
```

输出节点的顺序应为：3 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10 -> 11 -> 12 -> 13 -> 14 -> 15 -> 18 -> 20 -> 25 。

**测试结果：**

[![image-20200302112326786](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/13.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/13.png)

image-20200302112326786



##### 2.3.后续遍历

实现思路：与先序遍历原理相同，只不过是遍历的顺序不一样了。

- 首先，遍历其左子树；
- 然后，遍历其右子树；
- 最后，遍历根（父）节点；

**代码实现：**

```javascript
      //后序遍历
      BinarySearchTree.prototype.postOrderTraversal = function(handler){
        this.postOrderTraversalNode(this.root, handler)
      }

      BinarySearchTree.prototype.postOrderTraversalNode = function(node, handler){
        if (node != null) {
          //1.遍历左子树中的节点
          this.postOrderTraversalNode(node.left, handler)
          
          //2.遍历右子树中的节点
          this.postOrderTraversalNode(node.right, handler)

          //3.处理节点
          handler(node.key)
        }
      }
```

**过程详解：**

遍历的顺序应如下图所示：

[![image-20200302120246366](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/14.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/14.png)

image-20200302120246366



首先调用postOrderTraversal方法，在方法里再调用postOrderTraversalNode方法用于遍历二叉搜索树。先使用递归1遍历左子树中的节点；然后，遍历右子树中的节点；最后，处理父节点。

**测试代码：**

```javascript
    //测试代码
    //1.创建BinarySearchTree
    let bst = new BinarySearchTree()

    //2.插入数据
    bst.insert(11);
    bst.insert(7);
    bst.insert(15);
    bst.insert(5);
    bst.insert(3);
    bst.insert(9);
    bst.insert(8);
    bst.insert(10);
    bst.insert(13);
    bst.insert(12);
    bst.insert(14);
    bst.insert(20);
    bst.insert(18);
    bst.insert(25);
    bst.insert(6);
    
    //3.测试后序遍历
    let resultString3 =""
    bst.postOrderTraversal(function(key){
      resultString3 += key + "->"
    })
    alert(resultString3)
```

输出节点的顺序应为：3 -> 6 -> 5 -> 8 -> 10 -> 9 -> 7 -> 12 -> 14 -> 13 -> 18 -> 25 -> 20 -> 15 -> 11 。

**测试结果：**

[![image-20200302115446608](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/15.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/15.png)

image-20200302115446608



**总结：**以遍历根（父）节点的顺序来区分三种遍历方式。比如：先序遍历先遍历根节点、中序遍历第二遍历根节点、后续遍历最后遍历根节点。

#### 3.查找数据

##### 3.1.查找最大值&最小值

在二叉搜索树中查找最值非常简单，最小值在二叉搜索树的最左边，最大值在二叉搜索树的最右边。只需要一直向左/右查找就能得到最值，如下图所示：

[![image-20200302125521501](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/16.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/16.png)

image-20200302125521501



**代码实现：**

```javascript
      //寻找最大值
      BinarySearchTree.prototype.max = function () {
        //1.获取根节点
        let node = this.root
        //2.定义key保存节点值
        let key = null
        //3.依次向右不断查找，直到节点为null
        while (node != null) {
          key = node.key
          node = node.right
        }
        return key
      }

      //寻找最小值
      BinarySearchTree.prototype.min = function(){
         //1.获取根节点
         let node = this.root
        //2.定义key保存节点值
        let key = null
        //3.依次向左不断查找，直到节点为null
        while (node != null) {
          key = node.key
          node = node.left
        }
        return key
      }
```

**测试代码：**

```javascript
   //测试代码
    //1.创建BinarySearchTree
    let bst = new BinarySearchTree()

    //2.插入数据
    bst.insert(11);
    bst.insert(7);
    bst.insert(15);
    bst.insert(5);
    bst.insert(3);
    bst.insert(9);
    bst.insert(8);
    bst.insert(10);
    bst.insert(13);
    bst.insert(12);
    bst.insert(14);
    bst.insert(20);
    bst.insert(18);
    bst.insert(25);
    bst.insert(6);
    
    //4.测试最值
    console.log(bst.max());
    console.log(bst.min());
    
```

**测试结果：**

[![image-20200302133028801](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/17.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/17.png)

image-20200302133028801



##### 3.2.查找特定值

查找二叉搜索树当中的特定值效率也非常高。只需要从根节点开始将需要查找节点的key值与之比较，若**node.key < root**则向左查找，若**node.key > root**就向右查找，直到找到或查找到null为止。这里可以使用递归实现，也可以采用循环来实现。

**实现代码：**

```javascript
     //查找特定的key
      BinarySearchTree.prototype.search = function(key){
        //1.获取根节点
        let node = this.root

        //2.循环搜索key
        while(node != null){
          if (key < node.key) {
            //小于根(父)节点就往左边找
            node = node.left
            //大于根(父)节点就往右边找
          }else if(key > node.key){
            node = node.right
          }else{
            return true
          }
        } 
        return false
      }
```

**测试代码：**

```javascript
    //测试代码
    //1.创建BinarySearchTree
    let bst = new BinarySearchTree()

    //2.插入数据
    bst.insert(11);
    bst.insert(7);
    bst.insert(15);
    bst.insert(5);
    bst.insert(3);
    bst.insert(9);
    bst.insert(8);
    bst.insert(10);
    bst.insert(13);
    bst.insert(12);
    bst.insert(14);
    bst.insert(20);
    bst.insert(18);
    bst.insert(25);
    bst.insert(6);
    
    //3.测试搜索方法
    console.log(bst.search(24));//false
    console.log(bst.search(13));//true
    console.log(bst.search(2));//false
```

**测试结果：**

[![image-20200302141031370](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/18.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/18.png)

image-20200302141031370



#### 4.删除数据

**实现思路：**

**第一步：**先找到需要删除的节点，若没找到，则不需要删除；

首先定义变量current用于保存需要删除的节点、变量parent用于保存它的父节点、变量isLeftChild保存current是否为parent的左节点，这样方便之后删除节点时改变相关节点的指向。

**实现代码：**

```javascript
 		//1.1.定义变量
        let current = this.root
        let parent = null
        let isLeftChild = true

        //1.2.开始寻找删除的节点
        while (current.key != key) {
          parent = current
          // 小于则往左查找
          if (key < current.key) {
            isLeftChild = true
            current = current.left
          } else{
            isLeftChild = false
            current = current.rigth
          }
          //找到最后依然没有找到相等的节点
          if (current == null) {
            return false
          }
        }
        //结束while循环后：current.key = key
```

**第二步：**删除找到的指定节点，后分3种情况：

- 删除叶子节点；
- 删除只有一个子节点的节点；
- 删除有两个子节点的节点；

##### 4.1.情况1：没有子节点

没有子节点时也有两种情况：

当该叶子节点为根节点时，如下图所示，此时**current == this.root**，直接通过：**this.root = null**，删除根节点。

[![image-20200302154316749](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/19.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/19.png)

image-20200302154316749



当该叶子节点不为根节点时也有两种情况，如下图所示：

[![image-20200302154019653](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/20.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/20.png)

image-20200302154019653



若current = 8，可以通过：parent.left = null，删除节点8；

若current = 10，可以通过：parent.right = null，删除节点10；

**代码实现：**

```javas
        //情况1：删除的是叶子节点(没有子节点)
        if (current.left == null && current.right ==null) {
          if (current == this.root) {
            this.root = null
          }else if(isLeftChild){
            parent.left = null
          }else {
            parent.right =null
          }
        }
```

##### 4.2.情况2：有一个子节点

有六种情况分别是：

当current存在左子节点时（current.right == null）：

- 情况1：current为根节点（current == this.root），如节点11，此时通过：this.root = current.left，删除根节点11；
- 情况2：current为父节点parent的左子节点（isLeftChild == true），如节点5，此时通过：parent.left = current.left，删除节点5；
- 情况3：current为父节点parent的右子节点（isLeftChild == false），如节点9，此时通过：parent.right = current.left，删除节点9；

[![image-20200302172806401](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/21.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/21.png)

image-20200302172806401



当current存在右子节点时（current.left = null）：

- 情况4：current为根节点（current == this.root），如节点11，此时通过：this.root = current.right，删除根节点11。
- 情况5：current为父节点parent的左子节点（isLeftChild == true），如节点5，此时通过：parent.left = current.right，删除节点5；
- 情况6：current为父节点parent的右子节点（isLeftChild == false），如节点9，此时通过：parent.right = current.right，删除节点9；

[![image-20200302172527722](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/22.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/22.png)

image-20200302172527722



**实现代码：**

```javascript
        //情况2：删除的节点有一个子节点
        //当current存在左子节点时
        else if(current.right == null){
            if (current == this.root) {
              this.root = current.left
            } else if(isLeftChild) {
                parent.left = current.left
            } else{
                parent.right = current.left
            }
        //当current存在右子节点时
      } else if(current.left == null){
            if (current == this.root) {
              this.root = current.rigth
            } else if(isLeftChild) {
                parent.left = current.right
            } else{
                parent.right = current.right
            } 
      }
```

##### 4.3.情况3：有两个子节点

这种情况**十分复杂**，首先依据以下二叉搜索树，讨论这样的问题：

[![image-20200302181849832](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/23.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/23.png)

image-20200302181849832



**删除节点9**

在保证删除节点9后原二叉树仍为二叉搜索树的前提下，有两种方式：

- 方式1：从节点9的左子树中选择一合适的节点替代节点9，可知节点8符合要求；
- 方式2：从节点9的右子树中选择一合适的节点替代节点9，可知节点10符合要求；

[![image-20200302190601622](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/24.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/24.png)

image-20200302190601622



**删除节点7**

在保证删除节点7后原二叉树仍为二叉搜索树的前提下，也有两种方式：

- 方式1：从节点7的左子树中选择一合适的节点替代节点7，可知节点5符合要求；
- 方式2：从节点7的右子树中选择一合适的节点替代节点7，可知节点8符合要求；

[![image-20200302183058326](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/25.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/25.png)

image-20200302183058326



**删除节点15**

在保证删除节点15后原树二叉树仍为二叉搜索树的前提下，同样有两种方式：

- 方式1：从节点15的左子树中选择一合适的节点替代节点15，可知节点14符合要求；
- 方式2：从节点15的右子树中选择一合适的节点替代节点15，可知节点18符合要求；

[![image-20200302184038470](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/26.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/26.png)

image-20200302184038470



相信你已经发现其中的规律了！

**规律总结：**如果要删除的节点有两个子节点，甚至子节点还有子节点，这种情况下需要从要删除节点**下面的子节点中找到一个合适的节点**，来替换当前的节点。

若用current表示需要删除的节点，则合适的节点指的是：

- current左子树中比current**小一点点的节点**，即current**左子树**中的**最大值**；
- current右子树中比current**大一点点的节点**，即current**右子树**中的**最小值**；

**前驱&后继**

在二叉搜索树中，这两个特殊的节点有特殊的名字：

- 比current小一点点的节点，称为current节点的**前驱**。比如下图中的节点5就是节点7的前驱；
- 比current大一点点的节点，称为current节点的**后继**。比如下图中的节点8就是节点7的后继；

[![image-20200302210841453](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/27.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/27.png)

image-20200302210841453



**代码实现：**

- 查找需要被删除的节点current的后继时，需要在current的**右子树**中查找**最小值**，即在current的**右子树**中一直**向左遍历**查找；
- 查找前驱时，则需要在current的**左子树**中查找**最大值**，即在current的**左子树**中一直**向右**遍历查找。

下面只讨论查找current后继的情况，查找前驱的原理相同，这里暂不讨论。

##### 4.4.完整实现

```kotlin
      //删除节点
      BinarySearchTree.prototype.remove = function(key){
/*------------------------------1.寻找要删除的节点---------------------------------*/
        //1.1.定义变量current保存删除的节点，parent保存它的父节点。isLeftChild保存current是否为parent的左节点
        let current = this.root
        let parent = null
        let isLeftChild = true

        //1.2.开始寻找删除的节点
        while (current.key != key) {
          parent = current
          // 小于则往左查找
          if (key < current.key) {
            isLeftChild = true
            current = current.left
          } else{
            isLeftChild = false
            current = current.right
          }
          //找到最后依然没有找到相等的节点
          if (current == null) {
            return false
          }
        }
        //结束while循环后：current.key = key

/*------------------------------2.根据对应情况删除节点------------------------------*/
        //情况1：删除的是叶子节点(没有子节点)
        if (current.left == null && current.right ==null) {
          if (current == this.root) {
            this.root = null
          }else if(isLeftChild){
            parent.left = null
          }else {
            parent.right =null
          }
        }
        //情况2：删除的节点有一个子节点
        //当current存在左子节点时
        else if(current.right == null){
            if (current == this.root) {
              this.root = current.left
            } else if(isLeftChild) {
                parent.left = current.left
            } else{
                parent.right = current.left
            }
        //当current存在右子节点时
      } else if(current.left == null){
            if (current == this.root) {
              this.root = current.right
            } else if(isLeftChild) {
                parent.left = current.right
            } else{
                parent.right = current.right
            } 
      }
        //情况3：删除的节点有两个子节点
        else{
          //1.获取后继节点
          let successor = this.getSuccessor(current)

          //2.判断是否根节点
          if (current == this.root) {
            this.root = successor
          }else if (isLeftChild){
            parent.left = successor
          }else{
            parent.right = successor
          }

          //3.将后继的左子节点改为被删除节点的左子节点
          successor.left = current.left
        }
      }

      //封装查找后继的方法
      BinarySearchTree.prototype.getSuccessor = function(delNode){
        //1.定义变量,保存找到的后继
        let successor = delNode
        let current = delNode.right
        let successorParent = delNode

        //2.循环查找current的右子树节点
        while(current != null){
          successorParent = successor
          successor = current
          current = current.left
        }

        //3.判断寻找到的后继节点是否直接就是删除节点的right节点
        if(successor != delNode.right){
          successorParent.left = successor.right
          successor.right = delNode.right 
        }
        return successor
      }
```

**测试代码：**

```csharp
   //测试代码
    //1.创建BinarySearchTree
    let bst = new BinarySearchTree()

    //2.插入数据
    bst.insert(11);
    bst.insert(7);
    bst.insert(15);
    bst.insert(5);
    bst.insert(3);
    bst.insert(9);
    bst.insert(8);
    bst.insert(10);
    bst.insert(13);
    bst.insert(12);
    bst.insert(14);
    bst.insert(20);
    bst.insert(18);
    bst.insert(25);
    bst.insert(6);
    bst.insert(19);
    
   //3.测试删除代码
    //删除没有子节点的节点
    bst.remove(3)
    bst.remove(8)
    bst.remove(10)

    //删除有一个子节点的节点
    bst.remove(5)
    bst.remove(19)

    //删除有两个子节点的节点
    bst.remove(9)
    bst.remove(7)
    bst.remove(15)

    //遍历二叉搜索树并输出
    let resultString = ""
    bst.midOrderTraversal(function(key){
      resultString += key + "->"
    })
    alert(resultString)
```

**测试结果：**

[![image-20200302225943296](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/28.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/28.png)

image-20200302225943296



可见三种情况的节点都被成功删除了。

#### 5.二叉搜索树完整封装

```kotlin
    //封装二叉搜索树
    function BinarySearchTree(){

      //节点内部类
      function Node(key){
        this.key = key
        this.left = null
        this.right = null
      }

      //属性
      this.root = null

      //方法
      //一.插入数据：insert方法:对外向用户暴露的方法
      BinarySearchTree.prototype.insert = function(key){
        //1.根据key创建节点
        let newNode = new Node(key)
          
        //2.判断根节点是否存在
        if (this.root == null) {
          this.root = newNode
          //根节点存在时
        }else {
          this.insertNode(this.root, newNode)
        }
      }

      //内部使用的insertNode方法:用于比较节点从左边插入还是右边插入
      BinarySearchTree.prototype.insertNode = function(node, newNode){
        //当newNode.key < node.key向左查找
        if(newNode.key < node.key){
          //情况1：node无左子节点，直接插入
          if (node.left == null) {
            node.left = newNode
          //情况2：node有左子节点，递归调用insertNode(),直到遇到无左子节点成功插入newNode后，不再符合该情况，也就不再调用insertNode()，递归停止。
          }else{
            this.insertNode(node.left, newNode)
          }
        //当newNode.key >= node.key向右查找
        }else{
          //情况1：node无右子节点，直接插入
          if(node.right == null){
            node.right = newNode
          //情况2：node有右子节点，依然递归调用insertNode(),直到遇到无右子节点成功插入newNode为止
          }else{
            this.insertNode(node.right, newNode)
          }
        }
      }

      //二.树的遍历
      //1.先序遍历
      //掺入一个handler函数对得到的key进行处理
      BinarySearchTree.prototype.preOrderTraversal = function(handler){
        this.preOrderTraversalNode(this.root, handler)
      }

      //封装内部方法，对某个节点进行遍历
      BinarySearchTree.prototype.preOrderTraversalNode = function(node,handler){
        if (node != null) {
          //1.处理经过的节点
          handler(node.key)

          //2.遍历经过节点的左子节点
          this.preOrderTraversalNode(node.left, handler)

          //3.遍历经过节点的右子节点
          this.preOrderTraversalNode(node.right, handler)
        }
      }

      //2.中序遍历
      BinarySearchTree.prototype.midOrderTraversal = function(handler){
        this.midOrderTraversalNode(this.root, handler)
      }

      BinarySearchTree.prototype.midOrderTraversalNode = function(node, handler){
        if (node != null) {
          //1.遍历左子树中的节点
          this.midOrderTraversalNode(node.left, handler)
          
          //2.处理节点
          handler(node.key)

          //3.遍历右子树中的节点
          this.midOrderTraversalNode(node.right, handler)
        }
      }

      //3.后序遍历
      BinarySearchTree.prototype.postOrderTraversal = function(handler){
        this.postOrderTraversalNode(this.root, handler)
      }

      BinarySearchTree.prototype.postOrderTraversalNode = function(node, handler){
        if (node != null) {
          //1.遍历左子树中的节点
          this.postOrderTraversalNode(node.left, handler)
          
          //2.遍历右子树中的节点
          this.postOrderTraversalNode(node.right, handler)

          //3.处理节点
          handler(node.key)
        }
      }

      //三.寻找最值
      //寻找最大值
      BinarySearchTree.prototype.max = function () {
        //1.获取根节点
        let node = this.root
        //2.定义key保存节点值
        let key = null
        //3.依次向右不断查找，直到节点为null
        while (node != null) {
          key = node.key
          node = node.right
        }
        return key
      }

      //寻找最小值
      BinarySearchTree.prototype.min = function(){
         //1.获取根节点
         let node = this.root
        //2.定义key保存节点值
        let key = null
        //3.依次向左不断查找，直到节点为null
        while (node != null) {
          key = node.key
          node = node.left
        }
        return key
      }

      //查找特定的key
      BinarySearchTree.prototype.search = function(key){
        //1.获取根节点
        let node = this.root

        //2.循环搜索key
        while(node != null){
          if (key < node.key) {
            //小于根(父)节点就往左边找
            node = node.left
            //大于根(父)节点就往右边找
          }else if(key > node.key){
            node = node.right
          }else{
            return true
          }
        } 
        return false
      }

      //四.删除节点
      BinarySearchTree.prototype.remove = function(key){
/*------------------------------1.寻找要删除的节点---------------------------------*/
        //1.1.定义变量current保存删除的节点，parent保存它的父节点。isLeftChild保存current是否为parent的左节点
        let current = this.root
        let parent = null
        let isLeftChild = true

        //1.2.开始寻找删除的节点
        while (current.key != key) {
          parent = current
          // 小于则往左查找
          if (key < current.key) {
            isLeftChild = true
            current = current.left
          } else{
            isLeftChild = false
            current = current.right
          }
          //找到最后依然没有找到相等的节点
          if (current == null) {
            return false
          }
        }
        //结束while循环后：current.key = key

/*------------------------------2.根据对应情况删除节点------------------------------*/
        //情况1：删除的是叶子节点(没有子节点)
        if (current.left == null && current.right ==null) {
          if (current == this.root) {
            this.root = null
          }else if(isLeftChild){
            parent.left = null
          }else {
            parent.right =null
          }
        }
        //情况2：删除的节点有一个子节点
        //当current存在左子节点时
        else if(current.right == null){
            if (current == this.root) {
              this.root = current.left
            } else if(isLeftChild) {
                parent.left = current.left
            } else{
                parent.right = current.left
            }
        //当current存在右子节点时
      } else if(current.left == null){
            if (current == this.root) {
              this.root = current.right
            } else if(isLeftChild) {
                parent.left = current.right
            } else{
                parent.right = current.right
            } 
      }
        //情况3：删除的节点有两个子节点
        else{
          //1.获取后继节点
          let successor = this.getSuccessor(current)

          //2.判断是否根节点
          if (current == this.root) {
            this.root = successor
          }else if (isLeftChild){
            parent.left = successor
          }else{
            parent.right = successor
          }

          //3.将后继的左子节点改为被删除节点的左子节点
          successor.left = current.left
        }
      }

      //封装查找后继的方法
      BinarySearchTree.prototype.getSuccessor = function(delNode){
        //1.定义变量,保存找到的后继
        let successor = delNode
        let current = delNode.right
        let successorParent = delNode

        //2.循环查找current的右子树节点
        while(current != null){
          successorParent = successor
          successor = current
          current = current.left
        }

        //3.判断寻找到的后继节点是否直接就是删除节点的right节点
        if(successor != delNode.right){
          successorParent.left = successor.right
          successor.right = delNode.right 
        }
        return successor
      }
    }
```

### 二、平衡树

**二叉搜索树的缺陷：**

当插入的数据是有序的数据，就会造成二叉搜索树的深度过大。比如原二叉搜索树右 11 7 15 组成，如下图所示：

[![image-20200302231503639](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/29.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/29.png)

image-20200302231503639



当插入一组有序数据：6 5 4 3 2就会变成深度过大的搜索二叉树，会严重影响二叉搜索树的性能。

[![image-20200302231745864](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%A0%91%E4%BA%8C/30.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/树二/30.png)

image-20200302231745864



**非平衡树**

- 比较好的二叉搜索树，它的数据应该是**左右均匀分布**的；
- 但是插入**连续数据**后，二叉搜索树中的数据分布就变得**不均匀**了，我们称这种树为**非平衡树**；
- 对于一棵**平衡二叉树**来说，插入/查找等操作的效率是**O（logN）**；
- 而对于一棵**非平衡二叉树**来说，相当于编写了一个链表，查找效率变成了**O（N）**;

**树的平衡性**

为了能以**较快的时间O（logN）**来操作一棵树，我们需要**保证树总是平衡**的：

- 起码大部分是平衡的，此时的时间复杂度也是接近O（logN）的；
- 这就要求树中**每个节点左边的子孙节点**的个数，应该尽可能地等于**右边的子孙节点**的个数；

**常见的平衡树**

- **AVL树**：是最早的一种平衡树，它通过在每个节点多存储一个额外的数据来保持树的平衡。由于AVL树是平衡树，所以它的时间复杂度也是O（logN）。但是它的整体效率不如红黑树，开发中比较少用。
- **红黑树**：同样通过**一些特性**来保持树的平衡，时间复杂度也是O（logN）。进行插入/删除等操作时，性能优于AVL树，所以平衡树的应用基本都是红黑树。

## 图解红黑树

### 一、红黑树的五条规则

红黑树除了符合二叉搜索树的基本规则外，还添加了以下特性：

- **规则1：节点是红色或黑色的；**
- **规则2：根节点是黑色的；**
- **规则3：每个叶子节点都是黑色的空节点（NIL节点）；**
- **规则4：每个红色节点的两个子节点都是黑色的（从每个叶子到根的所有路径上不可能有两个连续的红色节点）；**
- **规则5：从任一节点到其每个叶子节点的所有路径都包含相同数目的黑色节点；**

[![image-20200303121529068](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/1.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/1.png)

image-20200303121529068



**红黑树的相对平衡**

前面5条规则的约束确保了以下红黑树的关键特性：

- 从**根到叶子节点**的**最长路径**，不会超过**最短路径**的**两倍**；
- 结果就是这棵树**基本**是平衡的；
- 虽然没有做到绝对的平衡，但是可以保证在最坏的情况下，该树依然是高效的；

为什么可以做到**最长路径不超过最短路径的两倍**呢？

- **性质4**决定了路径上不能有两个相连的红色节点；
- 所以，最长路径一定是红色节点和黑色节点交替而成的；
- 由于根节点和叶子节点都是黑色的，最短路径可能都是黑色节点，并且最长路径中一定是黑色节点多于红色节点；
- **性质5**决定了所有路径上都有相同数目的黑色节点；
- 这就表明了没有路径能多于其他任何路径两倍长。

### 二、红黑树的三种变换

插入一个新节点时，有可能树不再平衡，可以通过三种方式的变换使树保持平衡：

- **变色**；
- **左旋转**；
- **右旋转**；

#### 2.1.变色

为了重新符合红黑树的规则，需要把**红色**节点变为**黑色**，或者把**黑色**节点变为**红色**；

插入的**新节点**通常都是**红色节点**：

- 当插入的节点为**红色**的时候，大多数情况**不违反**红黑树的任何规则；
- 而**插入黑色节点，**必然会导致一条路径上多了一个**黑色节点**，这是很难调整的；
- 红色节点虽然可能导致**红红相连**的情况，但是这种情况可以通过**颜色调换和旋转**来调整；

#### 2.2.左旋转

以节点X为根**逆时针**旋转二叉搜索树，使得父节点原来的位置被自己的右子节点替代，左子节点的位置被父节点替代；

[![image-20200303132706061](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/2.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/2.png)

image-20200303132706061



**详解：**

如上图所示，左旋转之后：

- 节点X取代了节点a原来的位置；
- 节点Y取代了节点X原来的位置；
- 节点X的**左子树** a 仍然是节点X的**左子树**（这里X的左子树只有一个节点，有多个节点时同样适用，以下同理）；
- 节点Y的**右子树** c 仍然是节点Y的**右子树**；
- 节点Y的**左子树** b 向**左平移**成为了节点X的**右子树**；

除此之外，二叉搜索树左旋转之后仍为二叉搜索树：

[![image-20200303132617108](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/3.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/3.png)

image-20200303132617108



#### 2.3.右旋转

以节点X为根**顺时针**旋转二叉搜索树，使得父节点原来的位置被自己的左子节点替代，右子节点的位置被父节点替代；

[![image-20200303132529476](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/4.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/4.png)

image-20200303132529476



**详解：**

如上图所示，右旋转之后：

- 节点X取代了节点a原来的位置；
- 节点Y取代了节点X原来的位置；
- 节点X的**右子树** a 仍然是节点X的**右子树**（这里X的右子树虽然只有一个节点，但是多个节点时同样适用，以下同理）；
- 节点Y的**左子树** b 仍然是节点Y的**左子树**；
- 节点Y的**右子树 \**c 向\**右平移**成为了节点X的**左子树**；

除此之外，二叉搜索树右旋转之后仍为二叉搜索树：

[![image-20200303132501369](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/5.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/5.png)

image-20200303132501369



### 三、红黑树的插入操作

首先需要明确，在保证满足红黑树5条规则的情况下，新插入的节点必然是**红色节点**。

为了方便说明，规定以下四个节点：新插入节点为**N**（Node），N的父节点为**P**（Parent），P的兄弟节点为**U**（Uncle），U的父节点为**G**（Grandpa），如下图所示：

[![image-20200303120344016](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/6.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/6.png)

image-20200303120344016



#### 3.1.情况1

当插入的新节点N位于树的根上时，没有父节点。

这种情况下，只需要将红色节点变为黑色节点即可满足规则2 。

[![image-20200303132357511](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/7.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/7.png)

image-20200303132357511



#### 3.2.情况2

新界点N的父节点P为黑色节点，此时不需要任何变化。

此时既满足规则4也满足规则5。尽管新节点是红色的，但是新节点N有两个黑色节点NIL，所以通向它的路径上黑色节点的个数依然相等，因此满足规则5 。

[![image-20200303132304098](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/8.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/8.png)

image-20200303132304098



#### 3.3.情况3

节点P为红色，节点U也为红色，此时节点G必为黑色，即**父红叔红祖黑**。

在这种情况下需要：

- 先将父节点P变为黑色；
- 再将叔叔节点U变为黑色；
- 最后将祖父节点G变为红色；

即变为**父黑叔黑祖红**，如下图所示：

[![image-20200303132148128](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/9.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/9.png)

image-20200303132148128



可能出现的问题：

- N的祖父节点G的父节点也可能是红色，这就违反了规则4，此时可以通过递归调整节点颜色；
- 当递归调整到根节点时就需要旋转了，如下图节点A和节点B所示，具体情况后面会介绍；

[![image-20200303132050765](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/10.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/10.png)

image-20200303132050765



#### 3.4.情况4

节点P是红色节点，节点U是黑色节点，并且节点N为节点P的**左子节点**，此时节点G一定是黑色节点，即**父红叔黑祖黑**。

在这种情况下需要：

- 先变色：将父节点P变为黑色，将祖父节点G变为红色；
- 后旋转：以祖父节点G为根进行右旋转；

[![image-20200303131956298](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/11.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/11.png)

image-20200303131956298



#### 3.5.情况5

节点P是红色节点，节点U是黑色节点，并且节点N为节点P的**右子节点**，此时节点G一定是黑色节点，即**父红叔黑祖黑**。

在这种情况下需要：

- 先以节点P为根进行左旋转，旋转后如图b所示；
- 随后将**红色**节点**P**和**黑色**节点**B**看成一个整体的**红色**节点**N1**，将新插入的**红色**节点**N**看成**红色**节点**P1** 如图c所示。此时整体就转换为了情况4。

[![image-20200303140225631](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/12.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/12.png)

image-20200303140225631



接着可以按照情况4进行处理：

- 先变色：将N1节点的父节点P1变为黑色，将祖父节点G变为红色；
- 后旋转：以祖父节点G为根进行右旋转，旋转后如图 e 所示；
- 最后将节点N1和P1变换回来，完成节点N的插入，如图 f 所示；

[![image-20200303131736316](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/13.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/13.png)

image-20200303131736316



#### 3.6.案例

在二叉树中依次插入节点：10，9，8，7，6，5，4，3，2，1 。

如果直接采用普通的二叉搜索树，节点全部插入后是这样的：

[![image-20200303161149709](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/14.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/14.png)

image-20200303161149709



是一个严重的**不平衡树**，相当于一个链表，不能体现出二叉搜索树的高效率。而按照红黑树的五条规则插入节点就能最大程度保证搜索二叉树是一棵**平衡树**。以下为过程详解：**为了方便解释省略了部分红黑树的叶子节点（NIL）**

##### 插入10

符合**情况1**：

- 插入节点10；
- 将节点10的颜色变为黑色；

[![image-20200303161257010](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/15.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/15.png)

image-20200303161257010



##### 插入9

符合**情况2**：

- 不需要任何变化；

[![image-20200303161919072](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/16.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/16.png)

image-20200303161919072



##### 插入8

> 快速判断属于情况3还是情况4的方法：
>
> 从新插入的节点N出发，按图示箭头经过的四个节点，若为**红红黑红**3个红色节点则为情况3，若为**红红黑黑**两个红色节点则为情况4；

[![image-20200303162613413](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/17.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/17.png)

image-20200303162613413



符合**情况4**：

- 父节点9变成黑，祖父节点10变为红；
- 以祖父节点为根进行右旋转；

[![image-20200303163803388](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/18.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/18.png)

image-20200303163803388



##### 插入7

符合**情况3**：

- 父节点8和叔节点10变为黑，祖父节点9变为红；
- 此时会出现问题：不符合规则2，即根节点不为黑，此时可以把以9为根节点的二叉搜索树当作一个整体作为一个新插入的节点N，而此时又符合情况1，只需要把9变回黑色即可。

[![image-20200303165135028](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/19.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/19.png)

image-20200303165135028



##### 插入6

符合**情况4**：

- 父节点7变为黑，祖父节点8变为红；
- 以祖父节点8为根进行右旋转；

[![image-20200306170016958](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/20.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/20.png)

image-20200306170016958



##### 插入5

符合**情况3**：

- 父节点6和叔节点8变为黑，祖父节点7变为红；

[![image-20200303170150314](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/21.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/21.png)

image-20200303170150314



##### 插入4

符合**情况4**：

- 父节点5变为黑，祖父节点6变为红；
- 以祖父节点6为根进行右旋转；

[![image-20200303171028009](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/22.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/22.png)

image-20200303171028009



##### 插入3

**第一次变换**：符合**情况3**：

- 父节点4和叔节点6变为黑，祖父节点5变为红；

变换之后发现5和7为相连的两个红色节点，于是把以5为根的整个子树看成一个新插入的节点N1，再进行第二次变换。

[![image-20200303171755035](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/23.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/23.png)

image-20200303171755035



**第二次变换**：符合**情况4**：

- 父节点7变为黑，祖父节点9变为红；
- 以祖父节点9为根进行右旋转；

[![image-20200303172800302](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/24.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/24.png)

image-20200303172800302



最后复原N1得到变换后的红黑树：

[![image-20200303173158141](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/25.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/25.png)

image-20200303173158141



##### 插入2

符合**情况4**：

- 父节点3变为黑，祖父节点4变为红；
- 以祖父节点4为根进行右旋转；

[![image-20200303174011409](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/26.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/26.png)

image-20200303174011409



##### **插入1**

**第一次变换**：符合**情况3**：

- 父节点2和叔节点4变为黑，祖父节点3变为红；

变换之后发现3和5为相连的两个红色节点，于是把以3为根的整个子树看成一个新插入的节点N1，再进行第二次变换。

[![image-20200303180603741](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/27.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/27.png)

image-20200303180603741



**第二次变换**：符合**情况3**：

- 父节点5和叔节点9变为黑，祖父节点7变为红；即由图 b -> 图 c 。

变换之后发现根节点7为红色不符合规则2，所以把以7为根节点的红黑树看成一个新插入的节点N2，再进行第三次变换。

**第三次变换**：符合**情况1**：

- 直接将根节点7变为黑色即可。

[![image-20200303181133397](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/28.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/28.png)

image-20200303181133397



由此，完成了1~10节点的插入，虽然没有遇到情况5，不过情况5经过左旋转的操作便可转换为情况4，原理一样。如下图所示，将这棵红黑树的叶子节点NIL补全之后，经检验满足红黑树的五条规则，并且基本属于**平衡树**，效率较高。

[![image-20200303182326442](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E7%BA%A2%E9%BB%91%E6%A0%91/29.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/红黑树/29.png)

image-20200303182326442



### 四、红黑树的删除操作

红黑树的删除操作结合了复杂的**二叉树的删除操作**和复杂的**红黑树的插入规则**，整体来说难度非常大，篇幅较长，这里暂不进行探讨。

## JavaScript实现图结构

### 一、图论

#### 1.1.图的简介

**什么是图？**

- **图结构**是一种与**树结构**有些相似的数据结构；
- **图论**是数学的一个分支，并且，在数学中，树是图的一种；
- 图论以图为研究对象，研究**顶点**和**边**组成的**图形**的数学理论和方法；
- 主要的研究目的为：**事物之间的联系**，**顶点**代表**事物**，**边**代表两个事物间的**关系**；

**图的特点：**

- **一组顶点**：通常用 **V** （Vertex）表示顶点的集合；

- 一组边

  ：通常用

   

  E

   

  （Edge）表示边的集合；

  - 边是顶点和顶点之间的连线；
  - 边可以是有向的，也可以是无向的。比如A----B表示无向，A ---> B 表示有向；

**图的常用术语：**

- **顶点：**表示图中的一个**节点**；
- **边：**表示**顶点和顶点**给之间的**连线**；
- **相邻顶点：**由一条边连接在一起的顶点称为**相邻顶点**；
- **度：**一个顶点的**度**是**相邻顶点的数量**；
- **路径：**
  - **简单路径：**简单路径要求不包含重复的顶点；
  - **回路：**第一个顶点和最后一个顶点**相同**的路径称为回路；
- **无向图：**图中的所有边都是**没有**方向的；
- **有向图：**图中的所有边都是**有**方向的；
- **无权图：**无权图中的边没有任何权重意义；
- **带权图：**带权图中的边有一定的权重含义；

#### 1.2.图的表示

##### 邻接矩阵

表示图的常用方式为：**邻接矩阵**。

- 可以使用二维数组来表示邻接矩阵；
- 邻接矩阵让**每个节点和一个整数相关联**，该**整数作为数组的下标值**；
- 使用一个**二维数组**来表示顶点之间的**连接**；

[![image-20200303213913574](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/1.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/1.png)

image-20200303213913574



如上图所示：

- 二维数组中的**0**表示没有连线，**1**表示有连线；
- 如：A[ 0 ] [ 3 ] = 1，表示 A 和 C 之间有连接；
- 邻接矩阵的对角线上的值都为0，表示A - A ，B - B，等自回路都没有连接（自己与自己之间没有连接）；
- 若为无向图，则邻接矩阵应为对角线上元素全为0的对称矩阵；

**邻接矩阵的问题：**

- 如果图是一个**稀疏图**，那么邻接矩阵中将存在**大量的 0**，造成存储空间的浪费；

##### 邻接表

另外一种表示图的常用方式为：**邻接表**。

- 邻接表由图中**每个顶点**以及**和顶点相邻的顶点列表**组成；
- 这个列表可用多种方式存储，比如：**数组/链表/字典（哈希表）**等都可以；

[![image-20200303215312091](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/2.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/2.png)

image-20200303215312091



如上图所示：

- 图中可清楚看到**A与B、C、D相邻**，假如要表示这些与A顶点相邻的顶点（边），可以通过将它们作为A的值（value）存入到对应的**数组/链表/字典**中。
- 之后，通过键（key）A可以十分方便地取出对应的数据；

**邻接表的问题：**

- 邻接表可以简单地得出**出度**，即某一顶点指向其他顶点的个数；
- 但是，邻接表计算**入度**（指向某一顶点的其他顶点的个数称为该顶点的入度）十分困难。此时需要构造**逆邻接表**才能有效计算入度；

### 二、封装图结构

在实现过程中采用**邻接表**的方式来表示边，使用**字典类**来存储邻接表。

#### 2.1.添加字典类和队列类

首先需要引入之前实现的，之后会用到的字典类和队列类：

```kotlin
    //封装字典类
function Dictionary(){
  //字典属性
  this.items = {}

  //字典操作方法
  //一.在字典中添加键值对
  Dictionary.prototype.set = function(key, value){
    this.items[key] = value
  }

  //二.判断字典中是否有某个key
  Dictionary.prototype.has = function(key){
    return this.items.hasOwnProperty(key)
  }

  //三.从字典中移除元素
  Dictionary.prototype.remove = function(key){
    //1.判断字典中是否有这个key
    if(!this.has(key)) return false

    //2.从字典中删除key
    delete this.items[key]
    return true
  }

  //四.根据key获取value
  Dictionary.prototype.get = function(key){
    return this.has(key) ? this.items[key] : undefined
  }

  //五.获取所有keys
  Dictionary.prototype.keys = function(){
    return Object.keys(this.items)
  }

  //六.size方法
  Dictionary.prototype.keys = function(){
    return this.keys().length
  }

  //七.clear方法
  Dictionary.prototype.clear = function(){
    this.items = {}
  }
}

   // 基于数组封装队列类
    function Queue() {
    // 属性
      this.items = []
    // 方法
    // 1.将元素加入到队列中
    Queue.prototype.enqueue = element => {
      this.items.push(element)
    }

    // 2.从队列中删除前端元素
    Queue.prototype.dequeue = () => {
      return this.items.shift()
    }

    // 3.查看前端的元素
    Queue.prototype.front = () => {
      return this.items[0]
    }

    // 4.查看队列是否为空
    Queue.prototype.isEmpty = () => {
      return this.items.length == 0;
    }

    // 5.查看队列中元素的个数
    Queue.prototype.size = () => {
      return this.items.length
    }

    // 6.toString方法
    Queue.prototype.toString = () => {
      let resultString = ''
        for (let i of this.items){
          resultString += i + ' '
        }
        return resultString
      }
    }
```

#### 2.2.创建图类

先创建图类Graph，并添加基本属性，再实现图类的常用方法：

```csharp
    //封装图类
    function Graph (){
      //属性：顶点(数组)/边(字典)
      this.vertexes = []  //顶点
      this.edges = new Dictionary() //边
      }
```

#### 2.3.添加顶点与边

如图所示：

[![image-20200303235132868](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/3.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/3.png)

image-20200303235132868



创建一个数组对象vertexes存储图的顶点；创建一个字典对象edges存储图的边，其中key为顶点，value为存储key顶点相邻顶点的数组。

**代码实现：**

```kotlin
      //添加方法
      //一.添加顶点
      Graph.prototype.addVertex = function(v){
        this.vertexes.push(v)
        this.edges.set(v, []) //将边添加到字典中，新增的顶点作为键，对应的值为一个存储边的空数组
      }
      //二.添加边
      Graph.prototype.addEdge = function(v1, v2){//传入两个顶点为它们添加边
        this.edges.get(v1).push(v2)//取出字典对象edges中存储边的数组，并添加关联顶点
        this.edges.get(v2).push(v1)//表示的是无向表，故要添加互相指向的两条边
      }
```

#### 2.4.转换为字符串输出

为图类Graph添加toString方法，实现以邻接表的形式输出图中各顶点。

**代码实现：**

```javascript
      //三.实现toString方法:转换为邻接表形式
      Graph.prototype.toString = function (){
        //1.定义字符串，保存最终结果
        let resultString = ""

        //2.遍历所有的顶点以及顶点对应的边
        for (let i = 0; i < this.vertexes.length; i++) {//遍历所有顶点
          resultString += this.vertexes[i] + '-->'
          let vEdges = this.edges.get(this.vertexes[i])
          for (let j = 0; j < vEdges.length; j++) {//遍历字典中每个顶点对应的数组
            resultString += vEdges[j] + '  ';
          }
          resultString += '\n'
        }
        return resultString
      }
```

**测试代码：**

```javascript
   //测试代码
    //1.创建图结构
    let graph = new Graph()

    //2.添加顶点
    let myVertexes = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']
    for (let i = 0; i < myVertexes.length; i++) {
      graph.addVertex(myVertexes[i])
    }

    //3.添加边
    graph.addEdge('A', 'B')
    graph.addEdge('A', 'C')
    graph.addEdge('A', 'D')
    graph.addEdge('C', 'D')
    graph.addEdge('C', 'G')
    graph.addEdge('D', 'G')
    graph.addEdge('D', 'H')
    graph.addEdge('B', 'E')
    graph.addEdge('B', 'F')
    graph.addEdge('E', 'I')

    //4.输出结果
    console.log(graph.toString());
```

**测试结果：**

[![image-20200303233737451](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/4.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/4.png)

image-20200303233737451



#### 2.5.图的遍历

**图的遍历思想：**

- 图的遍历思想与树的遍历思想一样，意味着需要将图中**所有的顶点**都访问一遍，并且不能有**重复的访问**（上面的toString方法会重复访问）；

**遍历图的两种算法：**

- 广度优先搜索（Breadth - First Search，简称**BFS**）;
- 深度优先搜索（Depth - First Search，简称**DFS**）;
- 两种遍历算法都需要指定**第一个被访问的顶点**；

为了记录顶点是否被访问过，使用**三种颜色**来表示它们的状态

- **白色**：表示该顶点还没有被访问过；
- **灰色**：表示该顶点被访问过，但其相邻顶点并未完全被访问过；
- **黑色**：表示该顶点被访问过，且其所有相邻顶点都被访问过；

首先封装initializeColor方法将图中的所有顶点初始化为白色，代码实现如下：

```javascript
      //四.初始化状态颜色
      Graph.prototype.initializeColor = function(){
        let colors = []
        for (let i = 0; i < this.vertexes.length; i++) {
           colors[this.vertexes[i]] = 'white';
        }
        return colors
      }
```

##### 广度优先搜索

广度优先搜索算法的思路：

- 广度优先搜索算法会从指定的第一个顶点开始遍历图，先访问其所有的相邻顶点，就像一次访问图的一层；
- 也可以说是**先宽后深**地遍历图中的各个顶点；

[![image-20200303233840691](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/5.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/5.png)

image-20200303233840691



**实现思路：**

基于**队列**可以简单地实现广度优先搜索算法：

- 首先创建一个队列Q（尾部进，首部出）；
- 调用封装的initializeColor方法将所有顶点初始化为白色；
- 指定第一个顶点A，将A标注为**灰色**（被访问过的节点），并将A放入队列Q中；
- 循环遍历队列中的元素，只要队列Q非空，就执行以下操作：
  - 先将灰色的A从Q的首部取出；
  - 取出A后，将A的所有未被访问过（白色）的相邻顶点依次从队列Q的尾部加入队列，并变为灰色。以此保证，灰色的相邻顶点不重复加入队列；
  - A的全部相邻节点加入Q后，A变为黑色，在下一次循环中被移除Q外；

**代码实现：**

```javascript
      //五.实现广度搜索(BFS)
      //传入指定的第一个顶点和处理结果的函数
      Graph.prototype.bfs = function(initV, handler){
        //1.初始化颜色
        let colors = this.initializeColor()

        //2.创建队列
        let que = new Queue()

        //3.将顶点加入到队列中
        que.enqueue(initV)

        //4.循环从队列中取出元素，队列为空才停止
        while(!que.isEmpty()){
          //4.1.从队列首部取出一个顶点
          let v = que.dequeue()

          //4.2.从字典对象edges中获取和该顶点相邻的其他顶点组成的数组
          let vNeighbours = this.edges.get(v)

          //4.3.将v的颜色变为灰色
          colors[v] = 'gray'

          //4.4.遍历v所有相邻的顶点vNeighbours,并且加入队列中
          for (let i = 0; i < vNeighbours.length; i++) {
            const a = vNeighbours[i];
            //判断相邻顶点是否被探测过，被探测过则不加入队列中；并且加入队列后变为灰色，表示被探测过
            if (colors[a] == 'white') {
              colors[a] = 'gray'
              que.enqueue(a)
            }
          }

          //4.5.处理顶点v
          handler(v)

          //4.6.顶点v所有白色的相邻顶点都加入队列后，将顶点v设置为黑色。此时黑色顶点v位于队列最前面，进入下一次while循环时会被取出
          colors[v] = 'black'
        }
      }
```

**过程详解：**

下为指定的第一个顶点为A时的遍历过程：

- 如 a 图所示，将在字典edges中取出的与A相邻的且未被访问过的白色顶点B、C、D放入队列que中并变为灰色，随后将A变为黑色并移出队列；
- 接着，如图 b 所示，将在字典edges中取出的与B相邻的且未被访问过的白色顶点E、F放入队列que中并变为灰色，随后将B变为黑色并移出队列；

[![image-20200306144336380](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/6.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/6.png)

image-20200306144336380



- 如 c 图所示，将在字典edges中取出的与C相邻的且未被访问过的白色顶点G（A，D也相邻不过已变为灰色，所以不加入队列）放入队列que中并变为灰色，随后将C变为黑色并移出队列；
- 接着，如图 d 所示，将在字典edges中取出的与D相邻的且未被访问过的白色顶点H放入队列que中并变为灰色，随后将D变为黑色并移出队列。

[![image-20200306144427242](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/7.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/7.png)

image-20200306144427242



如此循环直到队列中元素为0，即所有顶点都变黑并移出队列后才停止，此时图中顶点已被全部遍历。

**测试代码：**

```javascript
    //测试代码
    //1.创建图结构
    let graph = new Graph()

    //2.添加顶点
    let myVertexes = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']
    for (let i = 0; i < myVertexes.length; i++) {
      graph.addVertex(myVertexes[i])
    }

    //3.添加边
    graph.addEdge('A', 'B')
    graph.addEdge('A', 'C')
    graph.addEdge('A', 'D')
    graph.addEdge('C', 'D')
    graph.addEdge('C', 'G')
    graph.addEdge('D', 'G')
    graph.addEdge('D', 'H')
    graph.addEdge('B', 'E')
    graph.addEdge('B', 'F')
    graph.addEdge('E', 'I')
    
    //4.测试bfs遍历方法
    let result = ""
    graph.bfs(graph.vertexes[0], function(v){
      result += v + "-"
    })
    console.log(result);
```

**测试结果：**

[![image-20200304120023711](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/8.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/8.png)

image-20200304120023711



可见，安装了广度优先搜索的顺序**不重复**地遍历了**所有**顶点。

##### 深度优先搜索

广度优先算法的思路：

- 深度优先搜索算法将会从指定的第一个顶点开始遍历图，沿着一条路径遍历直到该路径的最后一个顶点都被访问过为止；
- 接着沿原来路径回退并探索下一条路径，即**先深后宽**地遍历图中的各个顶点；

[![image-20200304120355088](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/9.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/9.png)

image-20200304120355088



**实现思路：**

- 可以使用**栈**结构来实现深度优先搜索算法；
- 深度优先搜索算法的遍历顺序与二叉搜索树中的先序遍历较为相似，同样可以使用**递归**来实现（递归的本质就是**函数栈**的调用）。

基于递归实现深度优先搜索算法：定义dfs方法用于调用递归方法dfsVisit，定义dfsVisit方法用于递归访问图中的各个顶点。

在dfs方法中：

- 首先，调用initializeColor方法将所有顶点初始化为白色；
- 然后，调用dfsVisit方法遍历图的顶点；

在dfsVisit方法中：

- 首先，将传入的指定节点v标注为**灰色**；
- 接着，处理顶点V；
- 然后，访问V的相邻顶点；
- 最后，将顶点v标注为黑色；

**代码实现：**

```javascript
      //六.实现深度搜索(DFS)
      Graph.prototype.dfs = function(initV, handler){
        //1.初始化顶点颜色
        let colors = this.initializeColor()

        //2.从某个顶点开始依次递归访问
        this.dfsVisit(initV, colors, handler)
      }

      //为了方便递归调用，封装访问顶点的函数，传入三个参数分别表示：指定的第一个顶点、颜色、处理函数
      Graph.prototype.dfsVisit = function(v, colors, handler){
        //1.将颜色设置为灰色
        colors[v] = 'gray'

        //2.处理v顶点
        handler(v)

        //3.访问V的相邻顶点
        let vNeighbours = this.edges.get(v)
        for (let i = 0; i < vNeighbours.length; i++) {
          let a = vNeighbours[i];
          //判断相邻顶点是否为白色，若为白色，递归调用函数继续访问
          if (colors[a] == 'white') {
            this.dfsVisit(a, colors, handler)
          }
          
        }

        //4.将v设置为黑色
        colors[v] = 'black'
      }
```

**过程详解：**

这里主要解释一下代码中的第3步操作：访问指定顶点的相邻顶点。

- 以指定顶点A为例，先从储存顶点及其对应相邻顶点的字典对象edges中取出由顶点A的相邻顶点组成的数组：

[![image-20200304155916036](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/10.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/10.png)

image-20200304155916036



- **第一步**：A顶点变为灰色，随后进入第一个for循环，遍历A白色的相邻顶点：B、C、D；在该for循环的第1次循环中（执行B），B顶点满足：colors == "white"，触发递归，重新调用该方法；
- **第二步**：B顶点变为灰色，随后进入第二个for循环，遍历B白色的相邻顶点：E、F；在该for循环的第1次循环中（执行E），E顶点满足：colors == "white"，触发递归，重新调用该方法；
- **第三步**：E顶点变为灰色，随后进入第三个for循环，遍历E白色的相邻顶点：I；在该for循环的第1次循环中（执行I），I顶点满足：colors == "white"，触发递归，重新调用该方法；
- **第四步**：I顶点变为灰色，随后进入第四个for循环，由于顶点I的相邻顶点E不满足：colors == "white"，停止递归调用。过程如下图所示：

[![image-20200304160536187](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/11.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/11.png)

image-20200304160536187



- **第五步**：递归结束后一路向上返回，首先回到第三个for循环中继续执行其中的第2、3...次循环，每次循环的执行过程与上面的同理，直到递归再次结束后，再返回到第二个for循环中继续执行其中的第2、3...次循环....以此类推直到将图的所有顶点访问完为止。

下图为遍历图中各顶点的完整过程：

- **发现**表示访问了该顶点，状态变为**灰色**；
- **探索**表示既访问了该顶点，也访问了该顶点的全部相邻顶点，状态变为**黑色**；
- 由于在顶点变为灰色后就调用了处理函数handler，所以handler方法的输出顺序为发现顶点的顺序即：A、B、E、I、F、C、D、G、H 。

[![image-20200304154745646](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/12.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/12.png)

image-20200304154745646



**测试代码：**

```javascript
    //测试代码
    //1.创建图结构
    let graph = new Graph()

    //2.添加顶点
    let myVertexes = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']
    for (let i = 0; i < myVertexes.length; i++) {
      graph.addVertex(myVertexes[i])
    }

    //3.添加边
    graph.addEdge('A', 'B')
    graph.addEdge('A', 'C')
    graph.addEdge('A', 'D')
    graph.addEdge('C', 'D')
    graph.addEdge('C', 'G')
    graph.addEdge('D', 'G')
    graph.addEdge('D', 'H')
    graph.addEdge('B', 'E')
    graph.addEdge('B', 'F')
    graph.addEdge('E', 'I')
    
    //4.测试dfs遍历顶点
    let result = ""
    graph.dfs(graph.vertexes[0], function(v){
      result += v + "-"
    })
    console.log(result);
    
```

**测试结果：**

[![image-20200304125313739](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E5%9B%BE/13.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/图/13.png)

image-20200304125313739



#### 2.6.完整实现

```kotlin
    //封装图结构
    function Graph (){
      //属性：顶点(数组)/边(字典)
      this.vertexes = []  //顶点
      this.edges = new Dictionary() //边

      //方法
      //添加方法
      //一.添加顶点
      Graph.prototype.addVertex = function(v){
        this.vertexes.push(v)
        this.edges.set(v, []) //将边添加到字典中，新增的顶点作为键，对应的值为一个存储边的空数组
      }
      //二.添加边
      Graph.prototype.addEdge = function(v1, v2){//传入两个顶点为它们添加边
        this.edges.get(v1).push(v2)//取出字典对象edges中存储边的数组，并添加关联顶点
        this.edges.get(v2).push(v1)//表示的是无向表，故要添加互相指向的两条边
      }

      //三.实现toString方法:转换为邻接表形式
      Graph.prototype.toString = function (){
        //1.定义字符串，保存最终结果
        let resultString = ""

        //2.遍历所有的顶点以及顶点对应的边
        for (let i = 0; i < this.vertexes.length; i++) {//遍历所有顶点
          resultString += this.vertexes[i] + '-->'
          let vEdges = this.edges.get(this.vertexes[i])
          for (let j = 0; j < vEdges.length; j++) {//遍历字典中每个顶点对应的数组
            resultString += vEdges[j] + '  ';
          }
          resultString += '\n'
        }
        return resultString
      }

      //四.初始化状态颜色
      Graph.prototype.initializeColor = function(){
        let colors = []
        for (let i = 0; i < this.vertexes.length; i++) {
           colors[this.vertexes[i]] = 'white';
        }
        return colors
      }

      //五.实现广度搜索(BFS)
      //传入指定的第一个顶点和处理结果的函数
      Graph.prototype.bfs = function(initV, handler){
        //1.初始化颜色
        let colors = this.initializeColor()

        //2.创建队列
        let que = new Queue()

        //3.将顶点加入到队列中
        que.enqueue(initV)

        //4.循环从队列中取出元素
        while(!que.isEmpty()){
          //4.1.从队列中取出一个顶点
          let v = que.dequeue()

          //4.2.获取和顶点相相邻的其他顶点
          let vNeighbours = this.edges.get(v)

          //4.3.将v的颜色变为灰色
          colors[v] = 'gray'

          //4.4.遍历v所有相邻的顶点vNeighbours,并且加入队列中
          for (let i = 0; i < vNeighbours.length; i++) {
            const a = vNeighbours[i];
            //判断相邻顶点是否被探测过，被探测过则不加入队列中；并且加入队列后变为灰色，表示被探测过
            if (colors[a] == 'white') {
              colors[a] = 'gray'
              que.enqueue(a)
            }
          }

          //4.5.处理顶点v
          handler(v)

          //4.6.顶点v所有白色的相邻顶点都加入队列后，将顶点v设置为黑色。此时黑色顶点v位于队列最前面，进入下一次while循环时会被取出
          colors[v] = 'black'
        }
      }

      //六.实现深度搜索(DFS)
      Graph.prototype.dfs = function(initV, handler){
        //1.初始化顶点颜色
        let colors = this.initializeColor()

        //2.从某个顶点开始依次递归访问
        this.dfsVisit(initV, colors, handler)
      }

      //为了方便递归调用，封装访问顶点的函数，传入三个参数分别表示：指定的第一个顶点、颜色、处理函数
      Graph.prototype.dfsVisit = function(v, colors, handler){
        //1.将颜色设置为灰色
        colors[v] = 'gray'

        //2.处理v顶点
        handler(v)

        //3.访问v相连的其他顶点
        let vNeighbours = this.edges.get(v)
        for (let i = 0; i < vNeighbours.length; i++) {
          let a = vNeighbours[i];
          //判断相邻顶点是否为白色，若为白色，递归调用函数继续访问
          if (colors[a] == 'white') {
            this.dfsVisit(a, colors, handler)
          }
          
        }

        //4.将v设置为黑色
        colors[v] = 'black'
      }
    }
```

> 参考资料:[JavaScript数据结构与算法](https://www.bilibili.com/video/BV1x7411L7Q7?from=search&seid=3912456004602554239)

## JavaScript实现排序算法

### 一、大O表示法

**大O表示法：**

- 在计算机中采用**粗略的度量**来描述计算机算法的**效率**，这种方法被称为**“大O”表示法**
- 在**数据项个数**发生改变时，**算法的效率**也会跟着改变。所以说算法A比算法B快两倍，这样的比较是**没有意义**的。
- 因此我们通常使用**算法的速度**随着**数据量的变化**会如何变化的方式来表示算法的效率，大O表示法就是方式之一。

**常见的大O表示形式**

| 符号         | 名称           |
| ------------ | -------------- |
| O（1）       | 常数           |
| O（log(n)）  | 对数           |
| O（n）       | 线性           |
| O（nlog(n)） | 线性和对数乘积 |
| O（n²）      | 平方           |
| O（2n）      | 指数           |

**不同大O形式的时间复杂度：**

[![image-20200304164951223](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/1.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/1.png)

image-20200304164951223



可以看到效率从大到小分别是：O（1）> O（logn）> O（n）> O（nlog(n)）> O（n²）> O（2n）

**推导大O表示法的三条规则：**

- **规则一**：用常量1取代运行时间中所有的加法常量。如7 + 8 = 15，用1表示运算结果15，大O表示法表示为O（1）；
- **规则二**：运算中只保留最高阶项。如N^3 + 3n +1，大O表示法表示为：O（N3）;
- **规则三**：若最高阶项的常数不为1，可将其省略。如4N2，大O表示法表示为：O（N2）;

### 二、排序算法

这里主要介绍几种简单排序和高级排序：

- **简单排序：**冒泡排序、选择排序、插入排序；
- **高级排序：**希尔排序、快速排序；

此处创建一个列表类ArrayList并添加一些属性和方法，用于存放这些排序方法：

```javascript
    //创建列表类
    function ArrayList() {
      //属性
      this.array = []

      //方法
      //封装将数据插入到数组中方法
      ArrayList.prototype.insert = function(item){
        this.array.push(item)
      }

      //toString方法
      ArrayList.prototype.toString = function(){
        return this.array.join('-')
      }

      //交换两个位置的数据
      ArrayList.prototype.swap = function(m, n){
        let temp  = this.array[m]
        this.array[m] = this.array[n]
        this.array[n] = temp
      }
```

#### 1.冒泡排序

**冒泡排序的思路：**

- 对未排序的各元素**从头到尾**依次比较**相邻的两个元素**大小关系；
- 如果**左边的人员高**，则将两人**交换位置**。比如1比2矮，不交换位置；
- 向**右移动一位**，继续比较2和3，最后比较 length - 1 和 length - 2这两个数据；
- 当到达**最右端**时，**最高的人**一定被放在了**最右边**；
- 按照这个思路，从最左端重新开始时，只需要走到**倒数第二个位置**即可；

[![image-20200304191223265](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/2.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/2.png)

image-20200304191223265



**实现思路：**

两层循环：

- 外层循环控制冒泡趟数：
  - 第一次：j = length - 1，比较到倒数第一个位置 ；
  - 第二次：j = length - 2，比较到倒数第二个位置 ；
- 内层循环控制每趟比较的次数：
  - 第一次比较： i = 0，比较 0 和 1 位置的两个数据；
  - 最后一次比较：i = length - 2,比较length - 2和 length - 1两个数据；

详细过程如下图所示：

[![image-20200304210611689](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/3.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/3.png)

image-20200304210611689



动态过程：

[![img](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/4.gif)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/4.gif)

**代码实现：**

```//冒泡排序
      //冒泡排序
      ArrayList.prototype.bubblesor = function(){
        //1.获取数组的长度
        let length = this.array.length

        //外层循环控制冒泡趟数
        for(let j = length - 1; j >= 0; j--){
          //内层循环控制每趟比较的次数
          for(let i = 0; i < j; i++){
          if (this.array[i] > this.array[i+1]) {
            //交换两个数据
            let temp  = this.array[i]
        	this.array[i] = this.array[i+1]
        	this.array[i+1] = temp
          }
        }
        }
      }
```

**测试代码：**

```cpp
    //测试类
    let list = new ArrayList()

    //插入元素
    list.insert(66)
    list.insert(88)
    list.insert(12)
    list.insert(87)
    list.insert(100)
    list.insert(5)
    list.insert(566)
    list.insert(23)
    
    //验证冒泡排序
    list.bubblesor()
    console.log(list);
```

**测试结果：**
[![image-20200304210433388](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/5.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/5.png)

image-20200304210433388



**冒泡排序的效率：**

- 上面所讲的对于7个数据项，比较次数为：6 + 5 + 4 + 3 + 2 + 1;
- 对于N个数据项，**比较次数**为：(N - 1) + (N - 2) + (N - 3) + ... + 1 = N * (N - 1) / 2；如果两次比较交换一次，那么**交换次数**为：N * (N - 1) / 4；
- 使用大O表示法表示比较次数和交换次数分别为：O（ N * (N - 1) / 2）和O（ N * (N - 1) / 4），根据大O表示法的三条规则都化简为：**O（N^2）**;

#### 2.选择排序

**选择排序改进了冒泡排序：**

- 将**交换次数**由**O（N^2）**减小到**O（N）**；
- 但是**比较次数**依然是**O（N^2）**；

**选择排序的思路：**

- 选定**第一个索引的位置**比如1，然后依次和后面的元素**依次进行比较**；
- 如果后面的元素，**小于**索引1位置的元素，则**交换位置**到索引1处；
- 经过一轮的比较之后，可以确定一开始指定的索引1位置的元素是**最小的**；
- 随后使用同样的方法除索引1意外**逐个比较剩下的元素**即可；
- 可以看出选择排序，**第一轮**会选出**最小值**，**第二轮**会选出**第二小的值**，直到完成排序。

[![image-20200304213253241](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/6.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/6.png)

image-20200304213253241



**实现思路：**

两层循环：

- 外层循环控制指定的索引：
  - 第一次：j = 0，指定第一个元素 ；
  - 最后一次：j = length - 1，指定最后一个元素 ；
- 内层循环负责将指定索引（i）的元素与剩下（i - 1）的元素进行比较；

动态过程：

[![img](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/7.gif)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/7.gif)

**代码实现：**

```javascript
      //选择排序
      ArrayList.prototype.selectionSort = function(){
        //1.获取数组的长度
        let length = this.array.length

        //2.外层循环：从0开始获取元素
        for(let j = 0; j < length - 1; j++){
          let min = j
          //内层循环：从i+1位置开始，和后面的元素进行比较
        for(let i = min + 1; i < length; i++){
          if (this.array[min] > this.array[i]) {
            min = i
          }
        }
        this.swap(min, j)
        }
      }
```

**测试代码：**

```cpp
    //测试类
    let list = new ArrayList()

    //插入元素
    list.insert(66)
    list.insert(88)
    list.insert(12)
    list.insert(87)
    list.insert(100)
    list.insert(5)
    list.insert(566)
    list.insert(23)
    
    //验证选择排序
    list.selectionSort()
    console.log(list);
```

**测试结果：**

[![image-20200304222224801](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/8.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/8.png)

image-20200304222224801



**选择排序的效率：**

- 选择排序的**比较次数**为：N * (N - 1) / 2，用大O表示法表示为：**O（N^2）**;
- 选择排序的**交换次数**为：(N - 1) / 2，用大O表示法表示为：**O（N）**;
- 所以选择排序的效率高于冒泡排序；

#### 3.插入排序

插入排序是简单排序中效率**最高**的一种排序。

**插入排序的思路：**

- 插入排序思想的核心是**局部有序**。如图所示，X左边的人称为**局部有序**；
- 首先指定一数据X（从第一个数据开始），并将数据X的左边变成局部有序状态；
- 随后将X右移一位，再次达到局部有序之后，继续右移一位，重复前面的操作直至X移至最后一个元素。

[![image-20200304231400959](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/9.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/9.png)

image-20200304231400959



插入排序的详细过程：

[![image-20200304231643777](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/10.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/10.png)

image-20200304231643777



动态过程：

[![img](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/11.gif)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/11.gif)

**代码实现：**

```javascript
      //插入排序
      ArrayList.prototype.insertionSort = function(){
        //1.获取数组的长度
        let length = this.array.length

        //2.外层循环:从第二个数据开始，向左边的已经局部有序数据进行插入
        for(let i = 1; i < length; i++){
          //3.内层循环：获取i位置的元素，使用while循环(重点)与左边的局部有序数据依次进行比较
          let temp = this.array[i]
          let j = i
          while(this.array[j - 1] > temp && j > 0){
            this.array[j] = this.array[j - 1]//大的数据右移
            j--
          }
          //4.while循环结束后，index = j左边的数据变为局部有序且array[j]最大。此时将array[j]重置为排序前的数据array[i]，方便下一次for循环
          this.array[j] = temp
        }
      }
```

**测试代码：**

```cpp
   //测试类
    let list = new ArrayList()

    //插入元素
    list.insert(66)
    list.insert(88)
    list.insert(12)
    list.insert(87)
    list.insert(100)
    list.insert(5)
    list.insert(566)
    list.insert(23)
    // console.log(list);

    //验证插入排序
    list.insertionSort()
    console.log(list);
```

**测试结果：**

[![image-20200304235529516](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/12.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/12.png)

image-20200304235529516



**插入排序的效率：**

- **比较次数：**第一趟时，需要的最大次数为1；第二次最大为2；以此类推，最后一趟最大为N-1；所以，插入排序的总比较次数为N * (N - 1) / 2；但是，实际上每趟发现插入点之前，平均只有全体数据项的一半需要进行比较，所以比较次数为：**N \* (N - 1) / 4**；
- **交换次数：**指定第一个数据为X时交换0次，指定第二个数据为X最多需要交换1次，以此类推，指定第N个数据为X时最多需要交换N - 1次，所以一共需要交换N * (N - 1) / 2次，平局次数为**N \* (N - 1) / 2**；
- 虽然用大O表示法表示插入排序的效率也是**O（N^2）**，但是插入排序整体操作次数更少，因此，在简单排序中，插入排序**效率最高**；

#### 4.希尔排序

**希尔排序**是**插入排序**的一种高效的**改进版**，效率比插入排序要**高**。

**希尔排序的历史背景：**

- 希尔排序按其设计者希尔（Donald Shell）的名字命名，该算法由**1959年公布**；
- 希尔算法首次突破了计算机界一直认为的**算法的时间复杂度都是O（N^2）**的大关，为了纪念该算法里程碑式

的意义，用**Shell**来命名该算法；

**插入排序的问题：**

- 假设一个**很小的数据项**在**很靠近右端的位置**上，这里本应该是**较大的数据项的位置**；
- 将这个**小数据项移动到左边**的正确位置，所有的**中间数据项都必须向右移动一位**，这样效率非常低；
- 如果通过**某种方式**，不需要**一个个移动所有中间的数据项**，就能把较小的数据项移到左边，那么这个算法的执行速度就会有很大的改进。

**希尔排序的实现思路：**

- 希尔排序主要通过对数据进行**分组**实现快速排序；
- 根据设定的增量（gap）将数据分为gap个组（**组数等于gap**），再在每个分组中进行局部排序；

> 假如有数组有10个数据，第1个数据为黑色，增量为5。那么第二个为黑色的数据index=5，第3个数据为黑色的数据index = 10（不存在）。所以黑色的数据每组只有2个，10 / 2 = 5一共可分5组，即**组数等于增量gap**。

- 排序之后，减小增量，继续分组，再次进行局部排序，直到增量gap=1为止。随后只需进行微调就可完成数组的排序；

具体过程如下：

- 排序之前的，储存10个数据的原始数组为：

[![image-20200305102330304](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/13.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/13.png)

image-20200305102330304



- 设初始增量gap = length / 2 = 5，即数组被分为了5组，如图所示分别为：[8, 3]、[9, 5]、[1, 4]、[7, 6]、[2, 0]：

[![image-20200305104914438](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/14.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/14.png)

image-20200305104914438



- 随后分别在每组中对数据进行局部排序，5组的顺序如图所示，变为：[3, 8]、[5, 9]、[1, 4]、[6, 7]、[0, 2]：

[![image-20200305103136251](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/15.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/15.png)

image-20200305103136251



- 然后缩小增量gap = 5 / 2 = 2，即数组被分为了2组，如图所示分别为：[3，1，0，9，7]、[5，6，8，4，2]：

[![image-20200305104933858](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/16.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/16.png)

image-20200305104933858



- 随后分别在每组中对数据进行局部排序，两组的顺序如图所示，变为：[0，1，3，7，9]、[2，4，5，6，8]：

[![image-20200305103815262](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/17.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/17.png)

image-20200305103815262



- 然后然后缩小增量gap = 2 / 1 = 1，即数组被分为了1组，如图所示为：[0，2，1，4，3，5，7，6，9，8]：

[![image-20200305104847458](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/18.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/18.png)

image-20200305104847458



- 最后只需要对该组数据进行插入排序即可完成整个数组的排序：

[![image-20200305104707789](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/19.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/19.png)

image-20200305104707789



动态过程：

[![img](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/20.gif)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/20.gif)

图中d表示增量gap。

**增量的选择：**

- **原稿**中希尔建议的初始间距为**N / 2**，比如对于N = 100的数组，增量序列为：50，25，12，6，3，1，可以发现不能整除时向下取整。
- **Hibbard增量序列：**增量序列算法为：2^k - 1，即1，3，5，7... ...等；这种情况的最坏复杂度为**O（N3/2）\**,平均复杂度为\**O（N5/4）**但未被证明；
- **Sedgewcik增量序列：**

[![image-20200305110724309](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/21.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/21.png)

image-20200305110724309



以下代码实现中采用希尔排序原稿中建议的增量即**N / 2** 。

**代码实现：**

```javascript
      //希尔排序
      ArrayList.prototype.shellSort = function(){
        //1.获取数组的长度
        let length = this.array.length

        //2.初始化增量
        let gap = Math.floor(length / 2)

        //3.第一层循环：while循环(使gap不断减小)
        while(gap >= 1 ){
          //4.第二层循环：以gap为增量，进行分组，对分组进行插入排序
          //重点为：将index = gap作为选中的第一个数据
          for(let i = gap; i < length; i++){
            let temp = this.array[i]
            let j = i
            //5.第三层循环:寻找正确的插入位置
            while(this.array[j - gap] > temp && j > gap - 1){
              this.array[j] = this.array[j - gap]
              j -= gap
            }
          //6.将j位置的元素设置为temp
          this.array[j] = temp
          }

          gap = Math.floor(gap / 2)
        }
      }
```

这里解释一下上述代码中的三层循环：

- **第一层循环：**while循环，控制gap递减到1；
- **第二层循环：**分别取出根据g增量gap分成的gap组数据：将index = gap的数据作为选中的第一个数据，如下图所示，gap=5，则index = gap的数据为3，index = gap - 1的数据为8，两个数据为一组。随后gap不断加1右移，直到gap < length，此时实现了将数组分为5组。

[![image-20200305104914438](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/21.5.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/21.5.png)

image-20200305104914438



- **第三层循环：**对每一组数据进行插入排序；

**测试代码：**

```cpp
   //测试类
    let list = new ArrayList()

    //插入元素
    list.insert(66)
    list.insert(88)
    list.insert(12)
    list.insert(87)
    list.insert(100)
    list.insert(5)
    list.insert(566)
    list.insert(23)
    // console.log(list);

    //验证希尔排序
    list.shellSort()
    console.log(list);
```

**测试结果：**

[![image-20200305114934209](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/22.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/22.png)

image-20200305114934209



**希尔排序的效率：**

- 希尔排序的效率和增量有直接关系，即使使用原稿中的增量效率都高于简单排序。

#### 5.快速排序

快速排序的介绍：

- **快速排序**可以说是**目前所有排序算法**中，**最快**的一种排序算法。当然，没有任何一种算法是在任意情况下都是最优的。但是，大多数情况下快速排序是比较好的选择。
- **快速排序**其实是**冒泡排序**的升级版；

快速排序的核心思想是**分而治之**，先选出一个数据（比如65），将比其小的数据都放在它的左边，将比它大的数据都放在它的右边。这个数据称为**枢纽**

和冒泡排序的不同：

- 我们选择的65可以一次性将它放在最正确的位置，之后就不需要做任何移动；
- 而冒泡排序即使已经找到最大值，也需要继续移动最大值，直到将它移动到最右边；

[![image-20200305154504624](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/23.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/23.png)

image-20200305154504624



**快速排序的枢纽：**

- **第一种方案：**直接选择第一个元素作为枢纽。但是，当第一个元素就是最小值的情况下，效率不高；
- **第二种方案：**使用随机数。随机数本身十分消耗性能，不推荐；
- **优秀的解决方法：**取index为头、中、位的三个数据排序后的中位数；如下图所示，按下标值取出的三个数据为：92，31，0，经排序后变为：0，31，92，取其中的中位数31作为**枢纽**（当（length-1）/2不整除时可向下或向上取整）：

[![image-20200305182934710](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/24.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/24.png)

image-20200305182934710



**实现枢纽选择：**

```javascript
//交换两个位置的数据
let swap = function(arr, m, n){
    let temp  = arr[m]
    arr[m] = arr[n]
    arr[n] = temp
}

//快速排序
//1.选择枢纽
let median = function(arr){
  //1.取出中间的位置
  let center = Math.floor(arr.length / 2)
  let right = arr.length - 1 
  let left = 0

  //2.判断大小并进行交换
  if (arr[left] > arr[center]) {
    swap(arr, left, center)
  }
  if (arr[center] > arr[right]){
    swap(arr, center, right)
  }
  if (arr[left] > arr[right]) {
    swap(arr, left, right)
  }
  //3.返回枢纽
  return center
}
```

数组经过获取枢纽函数操作之后，选出的3个下标值对应的数据位置变为：

[![image-20200320091750654](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/25.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/25.png)

image-20200320091750654



**动态过程：**

[![img](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/26.gif)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/26.gif)

**快速排序代码实现：**

```swift
//2.快速排序
let QuickSort = function(arr){
  if (arr.length == 0) {
    return []
  }
  let center = median(arr)
  let c = arr.splice(center, 1)
  let l = []
  let r = []

  for (let i = 0; i < arr.length; i++) {
      if (arr[i] < c) {
        l.push(arr[i])
      }else{
        r.push(arr[i])
      }        
  }
  return QuickSort(l).concat(c, QuickSort(r))
}
```

算法的巧妙之处在于通过:

```less
QuickSort(l).concat(c, QuickSort(r))
```

递归调用`QuickSort`函数实现了枢纽`Center`左边数据`l`和右边数据`r`的排序；

**测试代码：**

```yaml
let arr = [0, 13, 81, 43, 31, 27, 56, 92]
console.log(QuickSort(arr));
```

**测试结果**

[![image-20200320092254048](https://gitee.com/ahuntsun/BlogImgs/raw/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/28.png)](https://gitee.com/ahuntsun/BlogImgs/raw/master/数据结构与算法/排序算法/28.png)

image-20200320092254048



**快速排序的效率：**

- 快速排序最坏情况下的效率：每次选择的枢纽都是最左边或最右边的数据，此时效率等同于冒泡排序，时间复杂度为**O（n2）**。可根据不同的枢纽选择避免这一情况；
- 快速排序的平均效率：为**O（N\*logN）**，虽然其他算法效率也可达到O（N*logN），但是其中快速排序是**最好的**。