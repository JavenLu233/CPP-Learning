# 数据结构（C语言）


# 数据结构（C语言）

使用教材：

书名：数据结构（C语言版）

作者：严蔚敏  吴伟民

出版社：人民邮电出版社（2011）

---


## 抽象数据类型


#### 引用参数

在传参时可以添加&，表示引用参数，函数中对该参数的修改会影响到原地址的数据

如：

```Cpp
void MyTest(int &Num){
    Num = 2;
}

int main(){
    int Num = 1;
    MyTest(Num);
    printf("%d", Num);

    return 0;
}
```

在调用函数时**无需**再输入指针。



#### 状态量定义

```Cpp
#define TRUE 1         
#define FALSE 0        
#define OK 1
#define ERROR 0
#define INFEASIBLE -1   // 不可行
#define OVERFLOW -2     // 溢出
```

抽象数据类型定义函数时，返回值类型是Status，表示该函数执行后返回的状态值，实际上可以设置为int

或者添加

```Cpp
typedef int Status
```



#### 数据类型定义

为了方便灵活修改数据结构存储元素的类型，元素类型用ElemType表示，代码开头宏定义添加所需元素类型，以int举例

```Cpp
#define ElemType int
```

 ``


#### 常量定义

为了让一些具体数字更有意义，同时定义一些边界数值，通常在代码开头进行宏定义

如：

```Cpp
#define MAXSIZE 10000
```


#### 其他注意事项

* malloc函数在<stdlib.h>中
* 插入到第i个元素之前，操作后插入的元素变成新的第i个元素



## 线性表



### 顺序表

```Cpp
#include <stdio.h>
#include <stdlib.h>

#define TRUE 1
#define FALSE 0
#define OK 1
#define ERROR 0
#define INFEASIBLE -1
#define OVERFLOW -2 

#define MAXSIZE 100
typedef int ElemType;
typedef int Status;
// 定义顺序表的结构体 
typedef struct{
	ElemType *elem;	// 存储空间的基地址 
	int length;	// 顺序表的当前长度 

}SqList;

// 初始化顺序表 
Status InitList_Sq(SqList &L){
	//构造一个空的顺序表L 
	L.elem = (ElemType*)malloc(sizeof(ElemType)*MAXSIZE); 
	if(!L.elem) exit(OVERFLOW);
	L.length = 0;
	return OK;
} 

// 查找元素
Status LocateElem_Sq(SqList L, int e){
	for(int i=0; i<L.length; i++){
		if(L.elem[i] == e)	return i+1;
	}

	return 0;
} 

// 插入元素
Status ListInsert_Sq(SqList &L, int i, ElemType e){
	// 在顺序表L中第i个元素的位置之前插入新的元素e
	// i的合法范围是 [1, L.length+1]
	if(i<1 || i>L.length+1)	return ERROR;
	if(L.length == MAXSIZE)	return OVERFLOW;

	for(int j=L.length; j>=i-1; j--) 
		L.elem[j+1] = L.elem[j];  // 从后往前移动才不会造成原始数据丢失

	L.elem[i-1] = e;
	++L.length;

	return OK;
}

// 删除元素
Status ListDelete_Sq(SqList &L, int i, ElemType &e){
	// 在顺序表中删除第i个元素，并用e返回其值 
	// i合法范围为[1,L.length] 
	if(i<1 || i>L.length)	return ERROR;
	e = L.elem[i-1];

	for(int j=i; j<=L.length-1; j++){
		L.elem[j-1] = L.elem[j];
	}

	--L.length;
	return OK;

} 

int main(){
	SqList L;
	InitList_Sq(L);
	L.elem[0] = 1;
	L.elem[1] = 233;
	L.length = 2;  // 自己在初始化数据时不要忘了修改length 

	ListInsert_Sq(L, 2, 99);
	printf("[0]=%d [1]=%d [2]=%d\n",L.elem[0], L.elem[1], L.elem[2]);

	int index, e;
	index = LocateElem_Sq(L, 99);
	printf("index = %d\n", index);

	ListDelete_Sq(L, 1, e);
	printf("e = %d\n", e);
	printf("[0]=%d [1]=%d [2]=%d\n",L.elem[0], L.elem[1], L.elem[2]);


	return 0;
}
```



### 单链表

**一些注意事项**：

* 表头不要忘了初始化申请空间
* 双向链表删除处理时，如果右指针是NULL则不用对他进行操作


结点：node

整个链表必须从**头指针**开始

```cpp
#include <stdio.h>
#include <stdlib.h>


#define TRUE 1
#define FALSE 0
#define OK 1
#define ERROR 0
#define INFEASIBLE -1
#define OVERFLOW -2 

#define MAXSIZE 1000
typedef int ElemType;
typedef int Status; 

typedef struct LNode{
	ElemType data;			// 结点的数据域 
	struct LNode *next;		// 结点的指针域 

}LNode, *LinkList;			// LinkList 为指向结构体LNode的指针类型，用来表示头指针 

// 初始化链表 
Status IntList_L(LinkList &L){
	// 构造一个空的单链表L 

	L->next = NULL;		// 将头节点的指针域置空 
	return OK;
}

// 查找元素（按序号）
Status GetElem_L(LinkList L, int i, ElemType &e){
	// 查找单链表L中的第i个元素，并返回给e 

	LNode *p = L->next;		// 初始化，p指向第一个元素 
	int j = 1; 				// j为计数器 

	while(p && j<i){		// 向后扫描，直到第i个元素所在位置，无法找到时 p == NULL，会退出 
		p = p->next;
		++j;
	}

	if(!p || j>i) return ERROR;   // 第i个元素不存在 

	e = p->data;			// 第i个元素赋值给e 
	return OK;
} 

// 查找元素（按值）
LNode *LocateElem_L(LinkList L, ElemType e){
	// 查找带有e的值的元素 

	LNode *p = L->next;

	while(p && p->data != e){
		p = p->next;		// 寻找符合值的元素 
	}

	return p;  // 返回该元素在链表L中的位置 
} 

// 插入元素 
Status ListInsert_L(LinkList &L, int i, ElemType e){
	// 在链表L中，将元素e插入到第i个元素之前
	LNode *p = L;
	int j = 0;

	while( p && j<i-1){   // 找到第i-1个元素 
		p = p->next;
		++j;
	}

	if(!p || j>i-1)	 return ERROR;		// i大于表长+1(表含头指针) 或者 小于1 

	LNode *s = (LNode*)malloc(sizeof(LNode));  // 创建新的结点 
	s->data = e;							// 新结点数据赋值 
	s->next = p->next;						// 新结点指向第i+1个结点 
	p->next = s;							// 第i-1个结点指向新结点 
	return OK;

}

Status ListDelete_L(LinkList &L, int i, ElemType &e){
	// 在带头节点的单链表L中，删除第i个元素，并由e返回其值
	int j = 0;
	LNode *p = L, *q;
	while(p && j<i-1){		// 找到第i-1个结点 
		p = p->next;
		++j;
	}
	 
	if(!p || j>i-1)	return ERROR;	// i大于表长+1 或者 小于1 

	q = p->next;				// 记录待删除的第i个结点 
	p->next = q->next;			// 第i-1个结点的指向第i+1个结点 
	e = q->data;				// 取出第i个结点的数据用以返回 
	free(q);					// 释放第i个结点的空间 
	return OK;
} 


// 前插入结点建立链表(每次都插入在头指针之后) 
void CreateList_F(LinkList &L, int n){
	// 先清空(初始化)原链表,然后逆序输入n个元素的值,建立带头节点的单链表
	L = (LNode*)malloc(sizeof(LNode));
	L->next = NULL;

	printf("请输入%d个元素:", n);
	for(int i=n; i>0; i--){
		LNode *p = (LNode*)malloc(sizeof(LNode));
		scanf("%d", &p->data);	// 输入结点的数据(为了不混用stdio和iostream,默认数据类型为int) 
		p->next = L->next;		// 新结点插入到头指针之后 
		L->next = p; 			// 头指针指向新结点 
	} 

}

// 后插入结点建立链表(每次都插入到最后一个结点)
void CreateList_L(LinkList &L, int n){
	// 先清空(初始化)原链表,然后正序输入n个元素的值,建立带头节点的单链表 
	L = (LNode*)malloc(sizeof(LNode));
	L->next = NULL;
	LNode *r = (LNode*)malloc(sizeof(LNode));  // r用来记录已插入的上一个结点 
	r = L;

	printf("请输入%d个元素:", n);
	for(int i=n; i>0; i--){
		LNode *p = (LNode*)malloc(sizeof(LNode));
		scanf("%d", &p->data);			// 输入结点的数据(为了不混用stdio和iostream,默认数据类型为int)
		p->next = NULL;					// 当前结点是链表的最后一个结点,故其下一个元素指向NULL 
		r->next = p;					// 上一个结点指向当前结点 
		r = p;							// r赋值为当前结点的指针 
	} 

} 

//正序输出整个链表的数据 
void DisplayList(LinkList L){
	L = L->next;     // 跳过头指针

	printf("List:"); 
	while(L){	 // 结点非空时 
		printf("%d ", L->data);
		L = L->next;
	}
	printf("\n");
}


int main(){
	LinkList L;  
	CreateList_L(L, 10); // 1 2 3 4 5 6 7 8 9 0
	DisplayList(L); 

	int temp;
	GetElem_L(L, 2, temp);
	printf("L[2] = %d\n", temp);

	LNode *tmp;
	tmp = LocateElem_L(L, 5);
	printf("L[5] = %d\n", tmp->data);

	ListInsert_L(L, 8, 233);
	DisplayList(L);

	ListDelete_L(L, 4, temp);
	printf("Delete %d\n", temp);

	DisplayList(L);

	return 0;
}
```

<br />

### 循环链表

表中最后一个指针指向头指针,整个链表形成一个环.

循环链表的操作与单链表基本一致，区别在于算法中的循环条件不是 **p非空** 或 **p->next非空**，而是 **p非头指针**。


### 双向链表

指针域有两个，分别指向前驱结点和后继结点。

在对双向链表的操作时，要涉及两个指针的变动。

```cpp
typedef struct DuLNode{
    ELemType data;
    struct DuLNode *prior;
    struct DuLNode *next;
}DuLNode, *DuLinkList;
```


**算法描述：**

```cpp
// 插入元素e到第i个元素之前
Status ListInsert_DuL(DuLinkList &L, int i, ElemType e){
    if(!(p = GetElem_DuL(L, i))    return ERROR;
  
    s = new DulNode;
    s->data = e;
    s->prior = p->prior;
    s->next = p;
    p->prior = s;
    return OK;
}

// 删除第i个元素，并以e返回
Status ListDelete_DuL(DuLinkList &L, int i, ElemType &e){
    if(!(p= GetElem_DuL(L, i))) return ERROR;
    e = p->data;
    p->prior->next = p->next;
    p->next->prior = p->prior;
    free(p);
    return OK;
}
```


洛谷例题 P1160 队列安排

总结：链表只是一种思想，实际实现方式不一定要使用链表

```cpp
// AC

#include <bits/stdc++.h>
using namespace std;

int Node[100000+5][3];

int main(){
	int N;
	cin >> N;

	Node[1][0] = 1;
	for(int i=2; i<=N; i++){
		int k, p;
		cin >> k >> p;
		Node[i][0] = 1;

		if(p){
			int R = Node[k][2];
			Node[i][1] = k;
			Node[i][2] = R;
			if(R)	Node[R][1] = i;
			Node[k][2] = i;
		}

		else{
			int L = Node[k][1];
			Node[i][1] = L;
			Node[i][2] = k;
			if(L)	Node[L][2] = i;
			Node[k][1] = i;
		}
	}

	int M, del;
	cin >> M;
	while(M--){
		cin >> del;
		Node[del][0] = 0;
	}


	int begin;
	for(begin=1; begin<=N; begin++){
		if(Node[begin][1] == 0){
			break;
		}
	}

	while(begin){
		if(Node[begin][0]) cout << begin << ' ';
		begin = Node[begin][2];
	}

	return 0;
}

// 下方的代码RE

//typedef struct LNode{
//	struct LNode *next;
//	struct LNode *prior;
//	int num;
//
//}LNode, *LinkList; 
//
//LNode *FindElem(LinkList L, int k){
//	while(L && L->num != k){
//		L = L->next;
//	}
//	return L;
//}
//
//
//void InsertElem(LinkList &L, int k, int p, int num){
//	LNode *Former = FindElem(L, k), *Later;
//
//	if(Former){
//		if(!p) Former = Former->prior;
//		Later = Former->next;
//
//		LNode *NEW = (LNode*)malloc(sizeof(LNode));
//		NEW->num = num;
//		NEW->prior = Former;
//		NEW->next = Later;
//		Former->next = NEW;
//		Later->prior = NEW;
//	}
//
//}
//
//
//void ShowList(LinkList L){
//	L = L->next;
////	cout << "List:";
//	while(L){
//		cout << L->num << ' ';
//		L = L->next;
//	}
////	cout << '\n';
//}
//
//void DeleteElem(LinkList &L, int k){
//	LNode *P = FindElem(L, k);
//	if(P){
//		LNode *Former = P->prior;
//		LNode *Later = P->next;
//		Former->next = Later;
//		if(Later){
//			Later->prior = Former;
//		}
//	
//		free(P);
//	}
//
//}
//
//void InitList(LinkList &L, int N){
//	LNode *q = (LNode*)malloc(sizeof(LNode));
//	L->next = q;
//	q->prior = L;
//	q->num = 1;
//	q->next = NULL;
//
//	for(int num = 2; num <= N; num++){
//		int k,p;
//		cin >> k >> p;
//		InsertElem(L, k, p, num);
//	}
//}
//
//void DealList(LinkList &L, int M){
//	while(M--){
//		int k;
//		cin >> k;
//		DeleteElem(L, k);
//	}
//
//}
//
//
//int main(){
//	LinkList L = (LinkList)malloc(sizeof(LNode));
//	L->next = NULL;
//	L->prior = NULL;
//
//	int N;
//	cin >> N;
//
//	InitList(L, N);
////	ShowList(L);
//
//	int M;
//	cin >> M;
//	DealList(L, M);
//	ShowList(L);
//
//
//	return 0;
//}

```
