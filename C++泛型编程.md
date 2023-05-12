# C++泛型编程

## 1. 模板

###  1.1 函数模板

**语法：**

```c++
template<typename T>
函数声明或定义
```

**解释：**

template  ---- 声明创建模板

typename -----表明其后面的符号是一种数据类型

T ---- 通用的数据类型,名称可以替换，通常为大写字符



**示例：**

```
//交换函数
template<typename T>
void Swap (T &a, T &b){
	T temp = a;
	a = b;
	b = temp;
}

//调用模板的两种方法
void text()
{
	int a = 10;
	int b = 20;
	
	//方法一：自动类型推导
	Swap(a, b);
	
	//方法二：显示指定类型
	Swap<int>(a, b);
}
```



**普通函数和函数模板的区别：**

普通函数可以发生隐式类型转化

函数模板 用自动类型推导，不可以发生隐式类型转化

函数模板 用显示指定类型， 可以发生隐式类型转化



**普通函数与函数模板的调用规则**

调用规则如下：

1.如果函数模板和普通模板都可以实现，优先调用普通函数	 

2.可以通过空模板参数列表来强制调用函数模板： 例子 Swap<>(a, b); 其中<> 即为空模板。

3.函数模板也可以发生重载

```c++
template<typename T>
void myPrint(T a, T b)

template<typename T>
void myPrint(T a, T b, T c)
```

4.如果函数模板可以产生更好的匹配，优先调用函数模板

```
void myPrint(int a, int b)

template<typename T>
void myPrint(T a, T b)

void test()
{
	char a = 'a';
	char b = 'b';
	myPrint(a, b);   //由于调用普通函数myPrint()会产生char->int 的隐式类型转化，而调用模板函数不会，则优先调用隐式函数。
}
```



**模板的局限性：**

```
template<typename T>
bool compared(T a, T b)
{
	if (a == b) return true;
	else return false;
}

Person{
	string Name;
	int Age;
	
	Person(string name, int age){
		this.Name = name;
		this.Age = age;
	}
}

//解决方法:利用具体化Person的版本实现代码，具体化优先调用
template<> bool compared(Person &p1, Person &p2)
{
	if(p1.Name == p2.Name) return true;
	else return false;
}

void main(){
	Person p1("Tom", 10);
	Person p2("Tom", 11);
	
	bool result = compared(p1, p2); //报错，没有明确的解决方法
}


```



### 1.2 类模板

**语法：**

```
template<class T>
定义类	
```

**示例：**

```
template<class NameType, class AgeType>
class Person
{
public:
	Person(NameType name, AgeType age){
		this.Name = name;
		this.age = age;
	}
	NameType Name;
	AgeType Age;
}

void main()
{
	//调用类模板
	Person<string, int> p1("孙悟空"， 999)；
}
```



**类模板和函数模板的区别**

1.类模板没有自动类型推导的使用方式

2.类模板再模板参数列表中可以有默认参数

```
template<class NameType, class AgeType = int>
class Person
{
public:
	Person(NameType name, AgeType age){
		this.Name = name;
		this.age = age;
	}
	NameType Name;
	AgeType Age;
}
void main(){
	Person<string>("孙悟空"， 999)；
}
```



**类模板成员函数创建时机：类模板中的成员函数不是一开始就创建的，在调用才进行创建**

```
class Person1{
public:
	void showPerson1(){cout << "Person1,show" << endl;}
}

class Person2{
public:
	void showPerson2(){cout << "Person1,show" << endl;}
}

template<class T>
class Person
{
public:
	T obj;
	
	//类模板中的函数并不是一开始就创建的，而是在模板使用时才生成，如果没有test()调用，直接编译不会出错
	void func1(){
		cout << obj.showPerson1();
	}
	
	void func2(){
		cout << obj.showPerson2();
	}
}

void test()
{
	Person<Person1> p1;
	p1.func1()  //编译会出错，p1 没有showPerson2函数，说明函数调用才会创建成员函数
}
 
```



**类模板对象作为参数传入函数：**

1.指定传入类型

```
template<class NameType, class AgeType = int>
class Person
{
public:
	Person(NameType name, AgeType age){
		this.Name = name;
		this.age = age;
	}
	NameType Name;
	AgeType Age;
}

//指定具体传入类型
void test(Person<string, int> p)
{
	cout << p.Name << endl;	
}
```

2.参数模板化

```
template<class NameType, class AgeType = int>
class Person
{
public:
	Person(NameType name, AgeType age){
		this.Name = name;
		this.age = age;
	}
	NameType Name;
	AgeType Age;
}

//参数模板化，类似于模板函数
template<class T1, class T2>
void test(Person<T1, T2> p)
{
	cout << p.Name << endl;	
}
```

3.将整个类模板化

```
template<class NameType, class AgeType = int>
class Person
{
public:
	Person(NameType name, AgeType age){
		this.Name = name;
		this.age = age;
	}
	NameType Name;
	AgeType Age;
}

//将整个类模板化，泛型类对象
template<class T>
void test(T p)
{
	cout << p.Name << endl;	
}
```



**类模板与继承：**

1. 当子类继承的父类是一个类模板时，子类在声明时，要指定父类中T的类型
2. 如果不指定，编译器无法给子类分配内存
3. 如果想灵活指定父类中T类型， 子类也要变成类模板

```
template<class T>
class Base
{
	T m;
};

//错误，必须要知道父类中的T类型，才能继承给子类
class Son1:public Base<int>
{

};

//如果想灵活指定父类中T类型， 子类也要变成类模板
template<class T1, class T2>
class Son2:public Base<T2>
{
	T1 obj;
}
test1(){
	Son2<int,char> S2;
}
```



 **类模板中成员函数的类外实现**

```
//类模板
template<class T1, class T2>
class Person{
public:
	//成员函数声明
	Person(T1 name, T2 age);
	void shortPerson();
public:
	T1 m_Name;
	T2 mAge;
};

//构造函数类外实现
template<T1, T2>
Person<T1, T2>::Person(T1 name, T2 age){
	 this.m_Name = name;
	 this.m_Age = age;
}

template<T1, T2>
void Person<T1, T2>::shortPerson(){
	.....
}
```



**类模板分文件编写**

**问题：类模板中成员函数创建时机是在调用阶段**，导致文件编写时链接不到

**解决：将声明和实现写到一个文件中，并改后缀名为  .hpp  ， .hpp 是约定的名称，并不是强制  **

```
//方法一
#include "person.h" -> #include "person.cpp"

//方法二
将person.h头文件和person.cpp源文件写到一起，新文件起名person.hpp
```



***类模板与友元**



**类模板案列—数组类**

```c++
#pragma once
#include<iostream>
using namespace std;

template<class T>
class MyArray
{
public:
	//构造函数
	MyArray(int capacity) {
		//cout << "调用了构造函数" << endl;
		this->m_Capacity = capacity;
		this->m_Size = 0;
		this->pAddress = new T[this->m_Capacity];
	}

	//深拷贝  加const 是指arr可以访问但不能被修改
	MyArray(const MyArray& arr)
	{
		//cout << "调用了拷贝函数" << endl;
		this->m_Capacity = arr.m_Capacity;
		this->m_Size = arr.m_Size;

		//this->pAdress = arr.pAdress;  浅拷贝， 会导致堆区溢出
		this->pAddress = new T[arr.m_Capacity];
		for (int i = 0; i < this->m_Size; i++)
		{
			this->pAddress[i] = arr.pAddress[i];
		}
	}

	//operator= 防止浅拷贝问题
	MyArray& operator=(const MyArray& arr)
	{
		//cout << "调用了operator=函数" << endl;
		//先判断原来堆区是否有数据
		if (this->pAddress != NULL)
		{
			delete[] this->pAddress;
			this->pAddress = NULL;
			this->m_Capacity = 0;
			this->m_Size = 0;
		}


		//深拷贝
		this->m_Capacity = arr.m_Capacity;
		this->m_Size = arr.m_Size;
		this->pAddress = new T[arr.m_Capacity];
		for (int i = 0; i < this->m_Size; i++)
		{
			this->pAddress[i] = arr.pAddress[i];
		}

		return *this;
	}

	//尾插法
	void Push_back(const T& val)
	{
		if (this->m_Capacity == this->m_Size) {
			return;
		}

		this->pAddress[this->m_Size] = val;
		this->m_Size++;
	}

	//尾删法
	void Pop_back()
	{
		if (this->m_Size == 0)
		{
			return;
		}

		this->m_Size--;  //不会发生内存泄漏
	}

	//下标法查找数据
	T& operator[](int index)
	{
		if (index > this->m_Size)
		{
			return NULL;
		}
		return this->pAddress[index];
	}

	//返回数组容量
	int getSize()
	{
		return this->m_Size;
	}

	//析构函数
	~MyArray() {
		//cout << "调用了析构函数" << endl;
		if (this->pAddress != NULL)
		{
			delete[] this->pAddress;
			this->pAddress = NULL;
		}
	}

private:
	T* pAddress;   //指针指向堆区开辟的真实数组
	int m_Capacity;  //数组容量
	int m_Size;  //数组大小
};
```







