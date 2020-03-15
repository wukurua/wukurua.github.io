---
layout:     post
title:      C++标准库STL(2)——vector
subtitle:   
date:       2020-03-12
author:     wukurua
header-img: img/linux/post-bg-C++.jpg
catalog: true
tags:
    - C++
    - STL
---

# 一、向量(vector) #

vector属于顺序容器，用于容纳**不定长**线性序列（即线性群体），提供对序列的快速随机访问（也称直接访问）

数据结构很像一个数组，但与数组不同，**向量的内存用尽时，向量自动分配更大的连续内存区，将原先的元素复制到新的内存区，并释放旧的内存区**。这是向量类的优点。

# 二、使用 #
头文件：

	#include <vector>

创建vector对象(6种)：

	vector <int> vec;								//创建一个空vector
	vector <int> vec1(5);							//创建一个vector,元素个数为5
	vector <int> vec2(5,1);							//创建一个vector，元素个数为5,且值均为1	
	vector <int> vec3(vec2);						//复制构造函数
	vector <int> vec4(vec3.begin(),vec3.end()-2);	//复制[vec3.begin(),vec3.end()-2)区间内另一个数组的元素到vector中
	//vector <int> vec5(5)={1,2,3,4,5};				//C++11标准下才能使用

添加：
	
	vec.push_back(1);						//尾部插入
	vec.insert(vec.begin()+1,98);			//插入元素

删除：

	vec.pop_back();							//删除尾部结点
	vec.erase(vec.begin()+1,vec.end());		//删除第3个元素
	vec.erase(vec.begin()+i,vec.end()+j);	//删除区间[i,j-1];区间从0开始

访问	：

	
	cout<<vec[0]<<endl;						//使用下标访问元素
	vector<int>::iterator it;				//使用迭代器访问元素
	for(it=vec.begin();it!=vec.end();++it)
    	cout<<*it<<endl;

获取向量大小：
	
	cout<<vec.size()<<endl;

改变向量大小：

	vec.resize(4);

获取向量容量大小：

	cout<<vec.capacity()<<endl;

清空(清空内容，不释放内存空间)：

	vec.clear();

如果<>内为自定义类型且使用指针访问：

	vector <CMenu> *menuvec=new vector <CMenu>;
	menuvec->push_back(CMenu(101,"卤鸡翅"));
	menuvec->push_back(CMenu(102,"白饭"));
	menuvec->push_back(CMenu(103,"酸菜鱼"));
	vector <CMenu> *pNode=menuvec;
	for(int i=0;i<pNode->size();i++)
		cout<<menuvec->at(i).getName()<<endl;

	vector <CMenu>::iterator it=menuvec->begin();
	for(;it!=menuvec->end();it++)
		cout<<it->getName()<<endl;
	delete menuvec;

# 三、使用细节 #
## 1.要重载拷贝构造函数进行深拷贝 ##
如果没有重载，报错：

	main.obj : error LNK2001: unresolved external symbol "public: __thiscall CMenu::CMenu(class CMenu const &)" (??0CMenu@@QAE@ABV0@@Z)
	Debug/try12.exe : fatal error LNK1120: 1 unresolved externals

另外，如果没有重载拷贝构造函数实现深拷贝，在如下所示定义了一个新的CMenu对象后，编译器自动生成默认的拷贝构造函数浅拷贝此对象作为push_back函数的形参。然后，编译器调用析构函数释放`CMenu("卤鸡翅")`时分配的内存空间后，在`delete menuvec`时又调用析构函数释放menuvec中对应结点的内存空间时，因为此时为浅拷贝，两个CMenu对象的name指向同一内存空间，就存在**重复释放同一内存空间**的错误，造成栈溢出，从而报错。

	vector <CMenu> *menuvec=new vector <CMenu>;
	menuvec->push_back(CMenu("卤鸡翅"));
	......
	delete menuvec;

## 2.拷贝构造函数的参数前要加const关键字 ##

拷贝构造函数的参数前没有加const关键字，报错：

	e:\vc6++\microsoft visual studio\vc98\include\xmemory(34) : error C2558: class 'CMenu' : no copy constructor available
        e:\vc6++\microsoft visual studio\vc98\include\xmemory(66) : see reference to function template instantiation 'void __cdecl std::_Construct(class CMenu *,const class CMenu &)' being compiled

原因：
此时是在重载系统默认的拷贝构造函数，如果没有const就不符合STD库中的函数参数形式。另外，加const可以防止拷贝构造函数中修改参数的值，更安全。

举例：
>CMenu.cpp

	CMenu::CMenu(const CMenu &me)
	{
		this->count++;
		this->id=me.id;
		this->name=new char[20];
		strcpy(this->name,me.name);
	}

## 3.it++和++it的区别 ##
效果没区别，但效率有区别：
运算符重载时，使用++it要定义一个临时变量来接受it指向的值再对值进行自增再返回临时变量，而使用it++在对自身的值自增后返回this指针，效率更高，所以**推荐使用++it**。

	fushu &fushu::operator ++()
	{
		this->real++;
		this->imaginary++;
		return *this;
	}
	
	fushu fushu::operator ++(int)
	{
		fushu c=(*this);
		this->real++;
		this->imaginary++;
		return c;
	}

## 4.使用sort比较两个类的数据成员时需要重载运算符 ##
例如如果要比较两个类的id大小时，没有重载运算符时，报错：

	e:\vc6++\microsoft visual studio\vc98\include\algorithm(583) : error C2784: 'bool __cdecl std::operator <(const class std::vector<_Ty,_A> &,const class std::vector<_Ty,_A> &)' : could not deduce template argument for 'const class std::vector<_Ty,_A>
	 &' from 'class CMenu'
        e:\vc6++\microsoft visual studio\vc98\include\algorithm(548) : see reference to function template instantiation 'void __cdecl std::_Unguarded_insert(class CMenu *,class CMenu)' being compiled

重载运算符举例：

>CMenu.cpp

	bool CMenu::operator <(CMenu &b)
	{
		return (this->id<b.id);
	}

>main.cpp

	vector <CMenu> *menuvec=new vector <CMenu>;
	menuvec->push_back(CMenu("卤鸡翅"));
	menuvec->push_back(CMenu("白饭"));
	menuvec->push_back(CMenu("酸菜鱼"));
	sort(menuvec->begin(),menuvec->end());

## 5.倒序遍历输出 ##
(1)运算符重载

>CMenu.cpp

	bool CMenu::operator <(CMenu &b)
	{
		return (this->id>b.id);
	}

>main.cpp

	for(int i=0;i<menuvec->size();i++)
		cout<<menuvec->at(i).getId()<<" "<<menuvec->at(i).getName()<<endl;

(2)反向迭代器

	vector <CMenu>::reverse_iterator ir;
	for(ir=menuvec->rbegin();ir!=menuvec->rend();++ir)
		cout<<ir->getId()<<" "<<ir->getName()<<endl;




