# C++对象模型

```
class X {};
class Y : public virtual X {};
class Z : public virtual X {};
class A : public Y, public Z {};
```

![image-20230106203629770](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20230106203629770.png)



**X大小不为0，为1B，那是被编译器安排进去的一个char，这是为了让这一类的两个对象得以在内存中配置独一无二的地址**

![image-20230106203337361](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20230106203337361.png)

1. **Y和Z虚继承X，故Y和Z中有一个指向虚基类表vbt的虚基类指针vbptr，vbt中存放virtual base class subobject 或者 偏移位置offset**
2. 