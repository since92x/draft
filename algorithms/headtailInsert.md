# 单链表——头插尾插

##

```c
void TailCreatList(List *L) //尾插法建立链表 {
	List *s, *r;
	//s用来指向新生成的节点。r始终指向L的终端节点。
	r = L;
	//r指向了头节点，此时的头节点是终端节点。
	for (int i = 0; i < 10; i++) {
		s = (struct List*) malloc(sizeof(struct List));
		//s指向新申请的节点
		s->data = i;
		//用新节点的数据域来接受i
		r->next = s;
		//用r来接纳新节点
		r = s;
		//r指向终端节点
	}
	r->next = NULL;
	//元素已经全部装入链表L中
	//L的终端节点指针域为NULL，L建立完成
}
void HeadCreatList(List *L) //头插法建立链表 {
	List *s;
	//不用像尾插法一样生成一个终端节点。
	L->next = NULL;
	for (int i = 0; i < 10; i++) {
		s = (struct List*) malloc(sizeof(struct List));
		s->data = i;
		s->next = L->next;
		//将L指向的地址赋值给S;//头插法与尾插法的不同之处主要在此，
		//s所指的新节点的指针域next指向L中的开始节点
		L->next = s;
		//头指针的指针域next指向s节点，使得s成为开始节点。
	}
}
```
