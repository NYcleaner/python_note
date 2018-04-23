# python_note
相关概念：

MRO：Method Resolution Order，即方法解析顺序，是python中用于处理二义性问题的算法

 

二义性：

python支持多继承，多继承的语言往往会遇到以下两类二义性的问题：

有两个基类A和B，A和B都定义了方法f()，C继承A和B，那么调用C的f()方法时会出现不确定。
有一个基类A，定义了方法f()，B类和C类继承了A类（的f()方法），D类继承了B和C类，那么出现一个问题，D不知道应该继承B的f()方法还是C的f()方法。
C++也是支持多继承的语言之一

对于问题1，C++中通过同名覆盖的方式来解决，子类方法和属性会优先调用，如果要在子类中访问被屏蔽的基类成员，应使用基类名来限定（BaseClassName::Func()）。
对于问题2，C++中通过虚继承来解决，以virtual关键字修饰共同的直接基类，从而保证不会产生多个基类副本产生歧义。
python中通过C3算法很好的避免了以上两类二义性的情况。

 

深度优先算法（DFS，Depth-First-Search）

把根节点压入栈中。
每次从栈中弹出一个元素，搜索所有在它下一级的元素，把这些元素压入栈中。并把这个元素记为它下一级元素的前驱。
找到所要找的元素时结束程序。
如果遍历整个树还没有找到，结束程序。
 

 

广度优先算法（BFS，Breadth-First-Search）

把根节点放到队列的末尾。
每次从队列的头部取出一个元素，查看这个元素所有的下一级元素，把它们放到队列的末尾。并把这个元素记为它下一级元素的前驱。
找到所要找的元素时结束程序。
如果遍历整个树还没有找到，结束程序。
 

拓扑排序：

对一个有向无环图(Directed Acyclic Graph简称DAG)G进行拓扑排序，是将G中所有顶点排成一个线性序列，使得图中任意一对顶点u和v，若边(u,v)∈E(G)，则u在线性序列中出现在v之前。通常，这样的线性序列称为满足拓扑排序(TopologicalOrder)的序列，简称拓扑序列。

拓扑排序的实现步骤：

循环执行以下两步，直到不存在入度为0的顶点为止

选择一个入度为0的顶点并输出之；
从网中删除此顶点及所有出边。
 

python中调用父类方法的两种方式：

class A(object):

   def __init__(self):

       self.name = "A: name"

       print "A:__init__"

   def fun(self):

       print "A:fun"

 

class B(A):

   def __init__(self):

       print "B:__init__"

       A.__init__(self)                               # 使用类名直接调用

       super(B, self).__init__()                # 使用super关键字调用

   def fun(self):

       print "B:fun"

       A.fun(self)

       super(B, self).fun()

       print self.name

对于单继承来说，上面这两种方式并无本质上的区别，但是当出现多继承的时候，super得到的基类就不一定是我们“认为”的基类了，我们看下面这个例子：

class A(object):

   def __init__(self):

       print "enter A"

       print "leave A"

 

class B(object):

   def __init__(self):

       print "enter B"

       print "leave B"

 

class C(A):

   def __init__(self):

       print "enter C"

       super(C, self).__init__()

       print "leave C"

 

class D(A):

   def __init__(self):

       print "enter D"

       super(D, self).__init__()

       print "leave D"

 

class E(B, C):

   def __init__(self):

       print "enter E"

       B.__init__(self)

       C.__init__(self)

       print "leave E"

 

class F(E, D):

   def __init__(self):

       print "enter F"

       E.__init__(self)

       D.__init__(self)

       print "leave F"

 

f = F()

输出结果：

enter F

enter E

enter B

leave B

enter C

enter D

enter A

leave A

leave D

leave C

leave E

enter D

enter A

leave A

leave D

leave F

 

类的继承关系如下所示：

   object

  |       \

  |        A

  |      / |

  B       C  D

   \   /   |

     E     |

       \   |

         F

我们的本意是希望调用构造函数的时候，对于基类的构造方法也进行调用，但是实际结果发现，A和D的构造函数被调用了2次，而且奇怪的是，当调用super(C, self).__init__()的时候，竟然进入D的构造函数，这也是为什么D的构造函数被调用了两次（一次是F调用的，一次是C调用的）！从继承关系上看，C的基类应该是A才对。这就要引出下面要解释的，python中的C3方法。不过针对上面这个例子，修改的思路很简单，要么全部使用类名来调用基类方法，要么全部使用super()来调用，不要混用！

 

C3算法的演变历史：

经典类（python 2.2之前）：

在python 2.2之前，python中使用经典类（classicclass），经典类是一种没有继承的类，所有类型都是type类型，如果经典类作为父类，子类调用父类构造函数会报错。当时用作MRO的算法是DFS（深度优先），下面的例子是当时使用DFS算法的示例（向右是基类方向）：

正常的继承方式：
A->B->D

A->C->E

DFS的遍历顺序为：A->B->D->C->E

这种情况下，不会产生问题。

菱形的继承方式
A->B->D

A->C->D

DFS的遍历顺序为：A->B->D->C

对于这种情况，如果公共父类D中也定义了f()，C中重写了方法f()，那么C中的f()方法永远也访问不到，因为按照遍历的顺序始终先发现D中的f()方法，导致子类无法重写基类方法。

 

新式类（python2.2）：

在python2.2开始，为了使类的内置类型更加统一，引入了新式类（new-style class），新式类每个类都继承自一个基类，默认继承自object，子类可以调用基类的构造函数。由于所有类都有一个公共的祖先类object，所以新式类不能使用DFS作为MRO。在当时有两种MRO并存：

如果是经典类，MRO使用DFS

如果是新式类，MRO使用BFS

针对新式类的BFS示例如下（向右是基类方向）：

正常继承方式：
A->B->D

A->C->E

BFS的遍历顺序为：A->B->C->D->E

D是B的唯一基类，但是遍历时却先遍历节点C，这种情况下应该先从唯一基类进行查找，这个原则称为单调性。

菱形的继承方式
A->B->D

A->C->D

BFS的遍历顺序为：A->B->C->D

BFS解决了前面提到的子类无法重写基类方法的问题。

 

经典类和新式类并存（python2.3-python2.7），C3算法产生：

由于DFS和BFS针对经典类和新式类都有缺陷，从python2.3开始，引入了C3算法。针对前面两个例子，C3算法的遍历顺序如下：

正常继承方式：
A->B->D

A->C->E

C3的遍历顺序为：A->B->D->C->E

菱形的继承方式
A->B->D

A->C->D

C3的遍历顺序为：A->B->C->D

看起来是DFS和BFS的综合，但是并非如此，下面的例子说明了C3算法的具体实现：

从前面拓扑排序的定义可知，将有向无环图进行拓扑排序后，按照得到的拓扑序列遍历即可满足单调性，原因是由根到叶即是子类到基类的方向，当基类的入度为0是，它就是子类的唯一基类，此时会优先遍历此基类，符合单调性。而子类无法重写方法的问题也可以得到解决，因为当多个子类继承自同一个基类时，该基类的入度不会先于子类减为0，所以可以保证优先遍历入度减为0的子类。

结合下面这张图的例子来说明C3算法的执行步骤（图中箭头由子类指向父类）：

object c

首先找入度为0的点，只有A，把A取出，把A相关的边去掉，再找下一个入度为0的点，B和C满足条件，从左侧开始取，取出B，这时顺序是AB，然后去掉B相关的边，这时候入度为0的点有E和C，依然取左边的E，这时候顺序为ABE，接着去掉E相关的边，这时只有一个点入度为0，那就是C，取C，顺序为ABEC。去掉C的边得到两个入度为0的点D和F，取出D，顺序为ABECD，然后去掉D相关的边，那么下一个入度为0的就是F，然后是object。所以最后的排序就为ABECDFobject。

了解了C3算法，我们前面那个混用的例子中调用super(C,self).__init__()会去调用D构造函数的原因也就显而易见了。

在python中提供了__mro__内置属性来查看类的MRO，例如：

class D(object):

   pass

 

class E(object):

   pass

 

class F(object):

   pass

 

class C(D, F):

   pass

 

class B(E, D):

   pass

 

class A(B, C):

   pass

 

print A.__mro__

 

#输出：(<class '__main__.A'>, <class '__main__.B'>, <class '__main__.E'>, <class '__main__.C'>, <class '__main__.D'>, <class '__main__.F'>, <type 'object'>)

 

 

参考链接：

二义性：http://baike.baidu.com/link?url=chZ4pXPmkAFI3nLMSi3LdIWV3iP7lOoHTcxkRnI3AgebLSFllRJH_WnGkGwVYvdQ2czM35DBLixOWI1IS8fL0XVZbKhBl-u5IBaCjfPgCaFjtI6SgK-RIlN3CteSYlC1

C++ 二义性问题：http://blog.csdn.net/whz_zb/article/details/6843298

C++中常见的两种二义性问题及其解决方式：http://www.cnblogs.com/heimianshusheng/p/4836282.html

Python中Class类用法实例分析：http://www.jb51.net/article/74743.htm

广度优先算法：http://baike.baidu.com/link?url=Jpp4QfrvgYlZEmfDAtFWyku1V77IKM0XZj8NjFUsCElWApCeA3sWIY61_Z-fS4PEUgirnOVBnPESxhgO3tEjNwYabJ684m-cjj7caIbdAEoEL7FoYexv7YQ0_AZIsUVMkgCQrZywDyAPwTcpV1RhJ_

深度优先算法：http://baike.baidu.com/item/%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E7%AE%97%E6%B3%95

拓扑排序：http://baike.baidu.com/link?url=Ccrwv5pGeErmtnscpAMfDA6gp0c8GlT1jIaMD2HfyBSb9CaSXq96nYB_zXGJqbymLfWSkiJMgGHoRvOnuxjeAt0KwC72Lc79Y1GVFU7I5u0t6E-aquE9rqRvCBeWC1xv

拓扑排序的原理及其实现：http://blog.csdn.net/dm_vincent/article/details/7714519

python多继承详解：http://www.pythontab.com/html/2013/pythonhexinbiancheng_0828/550.html

你真的理解Python中MRO算法吗：http://python.jobbole.com/85685/
