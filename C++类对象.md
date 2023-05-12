# 类对象

## 类和对象

### 静态对象

**静态成员就是在成员变量和成员函数前加上关键字static，称为静态成员**

静态成员分为：

 1. **静态成员变量**	
    所有对象共享一份数据
    在编译阶段分配内存（全局区）
    类内声明，类外初始化
    **为什么static成员一定要在类外初始化?**
     这是因为被static声明的类静态数据成员，其实体远在main()函数开始之前就已经在全局数据段中诞生了(《Inside the C++ Object Model》page247)！其生命期和类对象是异步的，这是最主要的原因。静态语意说明即使没有类实体的存在,其静态数据成员的实体也是存的，这个时候对象的生命期还没有开始，如果你要到类中去初始化类静态数据成员,让静态数据成员的初始化依赖于类的实体，那怎么满足前述静态语意呢?难道类永远不被实例化，我们就永远不能访问到被初始化的静态数据成员吗? 

    ```
    class Person
    {
    public:
    	static int m_A;   //类内声明
    private:
    	static int m_B;   //私有的静态成员变量
    }
    int Person::m_A = 0;  //类外初始化
    int Person::m_B = 0;
    void test1()
    {
    	Person p1;
    	cout << p1.m_A << endl;  //输出0；
    	
    	//p1 和 p2共享一份数据
    	Person p2;
    	p2.m_A = 10;
    	cout << p1.m_A << endl;  //输出10
    }
    
    void test2()
    {
    	//静态成员变量有两种访问方式
    	//01.通过对象进行访问
    	Person p3;
    	cout << p3.m_A << endl;
    	
    	//02.通过类名进行访问
    	cout << Person::m_A << endl;
    }
    void test3()
    {
    	cout << Person::m_B << endl;  //报错，无法访问私有的静态成员变量
    }
    ```

 2. **静态成员函数**

    所有对象共享一个函数

    静态成员函数只能访问静态成员变量

    **静态成员函数只能使用静态成员变量**

    ```
    class Person
    {
    public:
    	static void func1()
    	{
    		cout << "func调用" << endl;
    		m_A = 100;   //正确，可以调用
    		m_B = 100; // 错误，不能调用非静态成员变量
    	}
    	static int m_A;  //静态成员
    	int m_B;
    private:
    	static void func2()
    	{
    		cout << "func2调用" << endl;
    	}
    };
    int Person::m_A = 10;
    
    void test()
    {
    	//通过类对象调用
    	Person p1;
    	p1.func1();
    	
    	//通过类直接调用
    	Person::func1();
    	
    	//静态成员函数也有作用域
    	Person::func2();  //不能调用，因为func2()私有
    }
    ```

    

## C++对象模型和this指针

### 成员变量和成员函数分开存储

在C++中，类内的成员变量和成员函数分开存储，只有非静态成员变量才属于类的对象上

### this指针的用途

1. 当形参和成员变量同名时，可用this指针来区分

2. return对象本身

   ```
   class Person
   {
   public:
   	Perosn(int age)
   	{
   		//当形参和成员变量同名时，可用this指针来区分
   		this.age = age; 
   	}
   	
   	Person& PersonAddPerson(Person p)
   	{
   		this->age += p.age;
   		return *this;   //返回对象本身，但是注意函数类型为Person&
   	}
   	int age;
   };
   
   ```



### 空指针访问成员函数

C++中空指针也是可以调用成员函数的，但是也要注意有没有用到this指针
如果用到this指针，需要加以判断保证代码的健壮性。

```
class Person
{
public:
	void ShowClassName()
	{
		cout << "我是Person类" << endl;
	}
	
	void ShowPerson()
	{
		cout << "年龄为" << m_Age << endl;
	}
	
	//为了保持代码的健壮性，需要对this指针进行判空
	void ShowPersonX()
	{
		if(this == NULL) return;   //判空
		cout << "年龄为" << m_Age << endl;  //这里实际上是this->m_Age
	}
	
	int m_Age;
};
void test1()
{
	Person *p = NULL;
	p->ShowClassName();  //不会报错，只是输出一个语句
	p->ShowPerson();  //会报错，因为p是一个空指针，没有m_Age
	p->ShowPersonX();  //不会报错，因为有判空；
}

```

### const修饰成员函数

**常函数：**

	1. 成员函数后加const后我们称这个函数为常函数
	1. 常函数内不可以修改成员属性
	1. 成员属性声明时加关键字mutable后，在常函数中依然可以修改

**常对象：**

	1. 对象前加const称该对象为常对象
	1. 常对象只能调用常函数

```
class Person
{
public:
	//this指针的本质是指针常量(Person * const this)， 指针的指向是不可被修改的
	//const Person * const this 前面的const表示this指向对象的       内容不可被修改，后面的const指this的指向不可被修改，
	void showPerson() const //常函数
	{
		this->m_A = 100;  //报错，this指向对象的内容不可被修改
		this->m_b = 100;  //正确，加了mutable关键字
	}
	
	void func()
	{}
	
	int m_A;
	mutable int m_B;
	
}

void test()
{
	const Person p;  //常对象（对象的内容不可被修改）
	p.m_A = 100;  //报错
	p.m_B = 100;  //正确
	
	p.showPerson();  //正确
	p.func();  //错误，常对象只能调用常函数
}
```



## 友元

**在程序里，有些私有属性也想让类外特殊的一些函数或者类进行访问，就需要用到友元的技术，友元的目的就是让一个函数或者类访问另一个类中私有成员，友元的关键字为friend**

**友元的三种实现：**

1. **全局函数做友元**

2. **类做友元**

3. **成员函数做友元**

   

### 全局函数做友元

```
Class Person
{
	//AccessToAge全局函数是Person类的友元，可以访问Person类中的私       有成员变量
 	friend void AccessToAge(Person *person);
 public:
 	Person(){}
 Private:
 	string m_Name;
 	int m_Age;
}

void AccessToAge(Person *person)
{
	cout << person->m_Age << endl;   //正确，AccessToAge全局函数是Person类的友元，可以访问Person类中的私有成员变量
}
```



### 类作为友元

```
class Perosn()
{
public:
	Person(){
		building = new Building();
	}
	void visit();
private:
	BUilding * building;
};

class Building
{
	//告诉编译器Person是友元类，可以访问Building
	friend class Person;
public:
	Building();
public:
	string m_SittingRoom;  //客厅
private:
	string m_Bedroom;  //卧室
};

void Person::visit()
{
	//都正确
	cout << "访问" << building->m_SittingRoom << endl;
	cout << "访问" << building->m_Bedroom << endl;
}
```

### 友元函数

**让Person类的某个成员函数成为Building类的友元函数，用来访问Building类的私有成员变量**

```
class Perosn()
{
public:
	Person(){
		building = new Building();
	}
	void visit();
private:
	BUilding * building;
};

class Building
{

	//告诉编译器Person是友元类，可以访问Building
	friend void Person::visit();
	
public:
	Building();
public:
	string m_SittingRoom;  //客厅
private:
	string m_Bedroom;  //卧室
};

void Person::visit()
{
	//都正确
	cout << "访问" << building->m_SittingRoom << endl;
	cout << "访问" << building->m_Bedroom << endl;
}
```



## 运算符重载

### 加号运算符重载（+）

两种实现方法：

 	1. 成员函数重载实现
 	2. 全局函数重载实现

```
class Person
{
public:
	Person();
	//成员函数重载实现
	Person operator+(Person& p)
	{
		Person p3;
		p3.m_A = this->m_A + p2.m_A;
		p3.m_b = this->m_B + p2.m_B;
		return p3;
	}
public:
	int m_A;
	int m_B;
};
//全局函数重载实现
Person operator+(Person& p1, Person& p2)
{
	Person p3;
	p3.m_A = p1.m_A + p2.m_A;
	p3.m_b = p1.m_B + p2.m_B;
	return p3;
}



```



### 左移重载运算符（<<）

作用：可以输出自定义数据类型

**应该用全局函数重载<<,  本质 operator<<(cout, p) 简化为 cout<<p**

```
class Person
{
	friend ostream& operator<<(ostream& cout, Person& person);
	//利用成员函数重载左移运算符，  p.operator<<(cout) 简化版本
	p << cout
	//不会利用成员函数重载<< 运算符，因为无法实现cout 在左侧
	//成员函数重载<<:   void operator<<(ostream& cout)
	int m_A;
	int m_B;
}

//应该用全局函数重载<<,  本质 operator<<(cout, p) 简化为 cout<<p
//此为链式编程方法：cout << p << "hello world" << endl;
//cout << p调用后返回一个ostream& cout,接着cout << "hello world"调用；
ostream& operator<<(ostream& cout, Person& person)
{
	cout << "person.m_A=" << person.m_A << "person.m_B" << person.m_B << endl;
	return cout;
}
```

### 递增运算符（++）

**注意：递增运算符++前置与后置的区别，即p++ 与 ++p 之间的区别**

```
class Person
{
public:
	Person():m_A(0); //初始化
	//递增运算符++前置：先++ 后返回值
	Person& operator++(){
		m_A++;
		return *this;
	}
	
	//递增运算符++后置：先返回值 后++
	Person operator++(int){//int是代表一个占位参数，可以区分前置与后置递增
		//先记录this的值
		Person temp = *this;
		//对this进行++
		this->m_A++;
		//返回temp,不能返回*this,因为此时*this内容已经发生改变
		return temp;
	}
	
}
```

### 赋值运算符重载（=）

```
class Person
{
public:
	Person(int a){
		m_A = new int(a);
	}
	//析构函数，释放堆区内存
	~Person(){
		if(m_A != NULL){
			delete m_A;
			m_A = NULL;
		}
	}
	
	Person& operate=(Person& person)
	{
		if(this->m_A != NULL){
			delete this->m_A;
			this->m_A = NULL;
		}
		this->m_A = new int(*person.m_A);
		return *this;
	}
	
public:
	int* m_A;
};
```

### 关系重载符重载(==)

```
class Person
{
public:
	Person();
	bool operator==(Person& person){
		if (this->m_Name == person.m_Name && this->m_Age == person.m_Age) return true;
	}
public:
	string m_Name;
	int m_Age;
}
```

### 函数调用运算符重载

```
class MyPrint
{
public:
	void operator()(string text){  //重载（）符号
		cout << text << endl;
	} 
}；

void test1()
{
	//重载（）操作符也称为仿函数
	MyPrint myFunc;
	myFunc("Hello world");  //直接输出Hello world
}



class MyAdd
{
public:
	int operator()(int v1,int v2)
	{
		return v1+v2;
	}
}
void test2()
{
	int v1 = 10, v2 = 20;
	MyAdd myadd;
	cout << myadd(v1, v2) << endl;
	cout << MyAdd()(v1, v2) << endl;  //其中MyAdd()为匿名对象
}
```



## 继承

### 继承方式：

![image-20230105215951740](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20230105215951740.png)

### 继承中的对象模型

```
//查看类的对象模型
cl /d1 reportSingleClassLayout类名 "文件路径"
cl /d1 reportSingleClassLayoutPerson "Person.cpp"
```

```
class Base
{
public:
	int m_A;
protected:
	int m_B;
private:
	int m_C;
}

class Person public Base
{
public:
	int m_D;
}

void test()
{
	//输出16
	//父类中所有非静态成员属性都会被子类继承下去
	//父类中私有成员属性是被编译器给隐藏了，因此是访问不到，但是确实被继承下去了
	Person persson;
	cout << sizeof(person) << endl;
}
```

### 继承中构造和析构的顺序

**先运行父类的构造函数**

**再运行子类的构造函数**

**先运行子类的析构函数**

**再运行父类的析构函数**

### 继承中父子类同名成员的访问

**父类和子类中的同名成员变量或者成员函数，子类直接访问的是子类中的成员，若要访问父类中的同名成员，则需要加作用域**

```
//例如Base和Person中有同名成员变量m_A和同名成员函数func()，则访问父类的同名成员时，加父类作用域
Person p;
p.Base::m_A;
p.Base::func();
```

### 继承中父子类同名静态成员的访问

```
class Base
{
public:
	static void func(){
		cout << "Base-func()"  << endl;
	}
	
	static void func(int){
		cout << "Base-func()" << int << endl;
	}
public:
	static int m_A;

}
int Base::m_A = 100;

class Person
{
public:
	static void func(){
		cout << "Person-func()"  << endl;
	}
public:
	static int m_A;
}
int Person::m_A = 200;

void test1()
{
	Person p;
	
	//通过类对象访问
	p.m_A;  //访问Person的m_A;
	p.Base::m_A;  //访问Base 的m_A;
	
	p.func();
	p.Base::func();
	
	//通过类名访问
	Person::m_A;
	Person::Base::m_A;
	
	Person::func();
	Person::Base::func();
}

void test2()
{
	//子类出现和父类同名静态成员函数，也会隐藏父类中所有同名成员函数
	//如果想访问父类中被隐藏同名成员，需要加作用域
	Person p;
	p.func(10);  //会报错，因为父类的func(int)被子类同名函数隐藏
	p.Base::func(10);  //正确
}

```

### 多继承语法

**注意：多继承中，父类中有同名成员时，访问时要加上作用域**

```
class 子类 : 继承方式 父类1, 继承方式 父类2, .....
```

### 菱形继承问题

![image-20230106161827518](C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20230106161827518.png)

```
//方法一
class Animal
{
public:
	int m_Age;
};
class Sheep : public Animal
{};
class Tuo : public Animal
{};

class SheepTuo : public Sheep, public Tuo
{};


```

**注意：以上菱形继承会导致SheepTuo对象最后出现两个m_Age,造成内存浪费并且会发生歧义**

**解决方法：虚继承**

```
//虚继承
class Animal
{
public:
	int m_Age;
};
class Sheep : virtual public Animal
{};
class Tuo : virtual public Animal
{};

class SheepTuo : public Sheep, public Tuo
{};

```

<img src="C:\Users\86130\AppData\Roaming\Typora\typora-user-images\image-20230106162759191.png" alt="image-20230106162759191"  />

## 多态

### 多态的基本概念

多态分为两类：

 1. 静态多态:函数重载和运算符重载属于静态多态，复用函数名

 2. 动态多态:派生类和虚函数实现运行时多态

    

静态多态和动态多态区别:
	1. 静态多态的函数地址早绑定–编译阶段确定函数地址
	1. 动态多态的函数地址晚绑定–运行阶段确定函数地址



```
#include<iostream>
#include<string>

using namespace std;

class Animal
{
public:
	//虚函数：实现函数地址晚绑定
	virtual void speak() {
		cout << "动物在说话" << endl;
	}
};
class Cat : public Animal
{
public:
	//重写speak函数
	void speak() {
		cout << "猫在说话" << endl;
	}
};

class Dog : public Animal
{
public:
	//重写speak函数
	void speak() {
		cout << "狗在说话" << endl;
	}
};

int main()
{
	//方式一(指针)
	Animal* animal1 = new Cat();
	animal1->speak();

	//方式二(引用)
	Cat cat;
	Animal& animal2 = cat;
	animal2.speak();

	//方法三 强制类型转化
	Animal* animal3 = new Dog();
	if (Dog *dog = dynamic_cast< Dog* > (animal3))
	{
		dog->speak();
	}
	
	system("pause");
}

```



### 纯虚函数和抽象类

在多态中，通常父类中虚函数的实现是毫无意义的，主要都是调用子类重写的内容
因此可以将虚函数改为**纯虚函数**（**除了纯虚析构函数外，纯虚函数在抽象类中不需要实现**）

```
//纯虚函数语法
virtual 返回值类型 函数名 (参数列表) = 0;
```

当类中有了纯虚函数，这个类也称为**抽象类**



**抽象类特点:**

	1. **无法实例化对象**
	1. **子类必须重写抽象类中的纯虚函数，否则也属于抽象类**



### 虚析构和纯虚析构

**多态使用时，如果子类中有属性开辟到堆区，那么父类指针在释放时无法调用到子类的析构代码**
**解决方式:将父类中的析构函数改为虚析构或者纯虚析构**

虚析构和纯虚析构共性:
**可以解决父类指针释放子类对象，都需要有具体的函数实现**。

虚析构和纯虚析构区别:
**如果是纯虚析构，该类属于抽象类，无法实例化对象。**

纯虚析构相对于纯虚函数的特点：
**纯虚析构：需要声明也需要实现**



**Question：由于animal为Animal*类型的指针，所以执行delete animal时候会调用Animal的析构函数导致Cat中m_Name所指内存无法释放，会导致内存泄露**

**解决方式：虚析构或纯虚析构**

```
//问题代码
class Animal
{
public:
	Animal() {
		cout << "调用了Animal的构造函数" << endl;
	}
	virtual void speak() = 0;

	~Animal()
	{
		cout << "调用了Animal的析构函数" << endl;
	}
};

class Cat : public Animal
{
public:
	Cat() {
		cout << "调用了Cat的构造函数" << endl;
		m_Name = new string();
		*m_Name = "Jack";
	}
	void speak() {
		cout << *m_Name << "在说话" << endl;
	}

	~Cat() {
		if (m_Name != NULL) {
			cout << "调用了Cat的析构函数" << endl;
			delete m_Name;
			m_Name = NULL;
		}
	}
public:
	string* m_Name;
};

void test()
{
	Animal* animal = new Cat();
	animal->speak();
	delete animal;
}

int main()
{
	test();

	system("pause");
}


//运行结果
调用了Animal的构造函数
调用了Cat的构造函数
Jack在说话
调用了Animal的析构函数
请按任意键继续. . .
```

```
//虚析构方式
class Animal
{
public:
	Animal() {
		cout << "调用了Animal的构造函数" << endl;
	}
	virtual void speak() = 0;

	virtual ~Animal()
	{
		cout << "调用了Animal的析构函数" << endl;
	}
};

class Cat : public Animal
{
public:
	Cat() {
		cout << "调用了Cat的构造函数" << endl;
		m_Name = new string();
		*m_Name = "Jack";
	}
	void speak() {
		cout << *m_Name << "在说话" << endl;
	}

	~Cat() {
		if (m_Name != NULL) {
			cout << "调用了Cat的析构函数" << endl;
			delete m_Name;
			m_Name = NULL;
		}
	}
public:
	string* m_Name;
};

void test()
{
	Animal* animal = new Cat();
	animal->speak();
	delete animal;
}

int main()
{
	test();

	system("pause");
}

//运行结果
调用了Animal的构造函数
调用了Cat的构造函数
Jack在说话
调用了Cat的析构函数
调用了Animal的析构函数
请按任意键继续. . .
```

```
//纯虚析构方式
class Animal
{
public:
	Animal() {
		cout << "调用了Animal的构造函数" << endl;
	}
	virtual void speak() = 0;
	
	//纯虚析构：需要声明也需要实现
	virtual ~Animal() = 0;
};
Animal::~Animal(){
	cout << "cout << "调用了Animal的析构函数" << endl;"
}

class Cat : public Animal
{
public:
	Cat() {
		cout << "调用了Cat的构造函数" << endl;
		m_Name = new string();
		*m_Name = "Jack";
	}
	void speak() {
		cout << *m_Name << "在说话" << endl;
	}

	~Cat() {
		if (m_Name != NULL) {
			cout << "调用了Cat的析构函数" << endl;
			delete m_Name;
			m_Name = NULL;
		}
	}
public:
	string* m_Name;
};

void test()
{
	Animal* animal = new Cat();
	animal->speak();
	delete animal;
}

int main()
{
	test();

	system("pause");
}

调用了Animal的构造函数
调用了Cat的构造函数
Jack在说话
调用了Cat的析构函数
调用了Animal的析构函数
请按任意键继续. . .
```



### 