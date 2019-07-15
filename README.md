# Hello-World

#include<stdio.h>
#include<stdlib.h>
#include<malloc.h>
#define RCMAX 512			//行列最大数

typedef int datatype;

typedef struct node {
	int row, col;
	struct node *down, *right;
	union v_next
	{
		datatype v;
		struct node *next;
	}v_next;
}MNode, *MLink;

MLink CreatMLink()			//返回十字链表的头指针
{
	MLink H;
	MNode *p, *q, *hd[RCMAX + 1];
	int i, j, k, s,m,n,t;
	datatype v;
	scanf("%d,%d,%d",&m, &n, &t);
	s = m;
	if (n>s)	s = n;
	H = (MLink)malloc(sizeof(MNode));			//申请总头结点
	H->row = m;	H->col = n;	hd[0] = H;
	for (i = 1; i <= s; i++) {
		p = (MLink)malloc(sizeof(MNode));			//申请第i行头结点
		p->row = 0;	p->col = 0;
		p->right = p;	p->down = p;
		hd[i] = p;
		hd[i - 1]->v_next.next = p;
	}
	hd[s]->v_next.next = H;			//将头结点形成循环表
	for (k = 1; k <= t; k++) {
		scanf("%d,%d,%d", &i, &j, &v);			//输入一个三元组，设值为int
		p = (MLink)malloc(sizeof(MNode));
		p->row = i;	p->col = j;	p->v_next.v = v;
		//以下是将*p插入到第j列链表中去，且按列号有序
		q = hd[i];
		while (q->right != hd[i] && q->right->col<j) 
			q = q->right;
		p->right = q->right;
		q->right = p;
		//以下是将*p插入到第i行s链表中去，且按列号有序
		q = hd[j];
		while (q->down != hd[j] && q->down->row<i) 
			q = q->down;
		p->down = q->down;
		q->down = p;
	}
	return H;
}

MLink Find_JH(MLink Ha, int col)
{
	MLink p = Ha->v_next.next;
	int j = 1;
	while (j<col) {
		p = p->v_next.next;
		j++;
	}
	return p;
}

MLink TransposeMat(MLink Ha) {
	MLink T;
	MNode *p, *q,*pa,*qa,*qb, *hd[RCMAX + 1];
	datatype num;
	int i,m,n,t;
	T = (MLink)malloc(sizeof(MNode));
	T->col = Ha->row;			//转置矩阵的行列与原矩阵列行对应
	T->row = Ha->col;
	hd[0] = T;			//指向T矩阵
	t = Ha->row;
	if (Ha->row < Ha->col)			
		t = Ha->col;
	for (i = 1; i <= t; i++)
	{
		p = (MLink)malloc(sizeof(MNode));
		p->row = 0; p->col = 0;
		p->right = p; p->down = p;
		hd[i] = p;
		hd[i - 1]->v_next.next = p;			
	}
	qa = Ha->v_next.next;			//分别指向原矩阵和转置矩阵第一行头结点
	qb = T->v_next.next;
	hd[t]->v_next.next = T;			
	while (qa != Ha) {			//判定是否处理完矩阵
		pa = qa->right;			
		while (pa != qa) {		//判定当前行是否处理完
			p = (MLink)malloc(sizeof(MNode));
			p->col = pa->row; p->row = pa->col; p->v_next.v = pa->v_next.v;
			m = pa->col; n = pa->row; t = pa->v_next.v;
			//按行列建立转置矩阵
			q = hd[m];			//跳转至第m行
			while (q->right != hd[m] && (q->right->col)<n) 
				q = q->right;
			p->right = q->right;
			q->right = p;
			q = hd[n];			//跳转至第n列
			while (q->down != hd[n] && (q->down->row)<m) 
				q = q->down;
			p->down = q->down;
			q->down = p;
			//处理下一个非零结点
			pa = pa->right;			
		}
		qa = qa->v_next.next;			//处理下一行
	}
	return T;
}

MLink AddMat(MLink Ha, MLink Hb)
{
	MNode *p, *q, *pa, *pb, *ca, *cb, *qa;
	datatype x;
	if (Ha->row != Hb->row || Ha->col != Hb->col) 
		return NULL;
	ca = Ha->v_next.next;			//ca初始指向A矩阵中第一行表头结点
	cb = Hb->v_next.next;			//cb初始指向B矩阵中第一行表头结点
	do {pa = ca->right;			//pa指向A矩阵当前行第一个结点
		qa = ca;			//qa是pa的前驱
		pb = cb->right;			//pb指向B矩阵当前行中第一个结点
		while (pb->col != 0) {			//当前行没有处理完
			if (pa->col<pb->col && pa->col != 0) {			//pa在pb左边且不是表头结点，pa指针右移
				qa = pa;
				pa = pa->right;
			}
			else if (pa->col>pb->col || pa->col == 0) {			//pa在pb右边或pb是表头结点（A矩阵此行全为零），在A的链表中插入pb指向的结点
				p = (MLink)malloc(sizeof(MNode));
				p->row = pb->row;	p->col = pb->col;
				p->v_next.v = pb->v_next.v;
				p->right = pa;			//新结点插入*pa的前面
				qa->right = p;	pa = p;
				//新结点还要插到列链表的合适位置，先找位置，再插入
				q = Find_JH(Ha, p->col);			//从链表头结点找起
				while (q->down->row != 0 && q->down->row<p->row) 
					q = q->down;
				p->down = q->down;			//插在*q的后面
				q->down = p;
				pb = pb->right;
			}
			else {			//pa所指结点值域改写为和值
				x = pa->v_next.v + pb->v_next.v;
				if (x == 0) {			//和为零，从A中删除pa所指结点
					qa->right = pa->right;			//从行链中删除
					//还要从行链中删除，找*pa的列前驱结点
					q = Find_JH(Ha, pa->col);			//从列链表的头结点找起
					while (q->down->row<pa->row) 
						q = q->down;
					q->down = pa->down;
					free(pa);
					pa = qa;
				}
				else {			//和不为零，改写结点值域
					pa->v_next.v = x;
					qa = pa;
				}
				pa = pa->right;
				pb = pb->right;
			}
		}
		ca = ca->v_next.next;			//ca指向A中下一行的表头结点
		cb = cb->v_next.next;			//cb指向B中下一行的表头结点
	} while (ca->row == 0);			//当还有未处理完的行则继续
	return Ha;
}

MLink MutiplyMat(MLink Ha, MLink Hb) {
	MNode *pa, *pb, *qa, *qb, *ca, *cb, *cc, *d,*D;
	datatype c[RCMAX];			//用于存储
	int i, j, s, t;

	if (Ha->col != Hb->row) {			//若A的列数与B的行数不相等，则不能相乘
		printf("Error:Size mismatch");
		return NULL;
	}
	qa = Ha->v_next.next;
	cb = D->v_next.next;
	for (s = 1; s <= Ha->row; s++) {			//按行扫描Ha矩阵
		for (i = 0; i <= Hb->col; i++)			//初始化存储计算结果的数组
			c[i] = 0;			
		qb = Hb->v_next.next;
		for (t = 1; t <= Hb->col; t++) {			//按列扫描Hb矩阵
			pa = qa->right;
			pb = qb->down;
			while (pa->col != 0 || pb->row != 0) {			//回到头结点则结束
				if (pa->col>pb->row)			//pa在右
					if (pb->row == 0)			//Hb该列全为零，不处理
						pa = pa->right;
					else 
						pb = pb->down;			//有非零元素，移动pb
				else if (pa->col = pb->row) {			//pa、pb在同列
					c[t] = c[t] + (pa->v_next.v)*(pb->v_next.v);			//相应元素相乘并存储
					pa = pa->right;
					pb = pb->down;
				}
				else if (pa->col<pb->row)			//pb在右
					if (pa->col == 0)			//同第一种情况处理
						pb = pb->down;
					else 
						pa = pa->right;
			}
			qb = qb->v_next.next;
		}
		cc = cb;
		d = NULL;
		for (i = 1; i <= Hb->col; i++){			//处理存储计算结果的数组
			if (c[i] != 0) {				//只处理非零元素
				ca = (MLink)malloc(sizeof(MNode));			//创建新结点，数组下标为结点列号，值域相同
				ca->col = i;	ca->row = qa->row;
				ca->v_next.v = c[i];
				ca->right = cb;		cc->right = ca;
				cc = ca;
				d = Find_JH(D, i);
				ca->down = d;
				while (d->down != Find_JH(D, i))			//插入列
					d = d->down;
				d->down = ca;
			}
		}
		qa = qa->v_next.next;			//处理下一行
		cb = cb->v_next.next;
	}
	return D;
}

void print_MLinkMat(MLink A)
{
	int i, j;
	MLink q, p = A;
	printf("    ");
	for (j = 1; j <= A->col; j++) 
		printf("%4d", j);
	printf("\n");
	for (i = 1; i <= A->row; i++) {
		j = 1;
		p = p->v_next.next;			//第i行的头结点
		q = p->right;			//第i行的第一个结点
		printf("i=%d", i);
		while (q != p) {
			for (; j<q->col; j++) 
				printf("    ");
			j++;
			printf("%4d", q->v_next.v);
			q = q->right;
		}
		printf("\n");
	}
}

void main() {
	MNode *A, *B, *C, *D, *T1,*T2;
	printf("请输入矩阵A：\n");
	A = CreatMLink();
	print_MLinkMat(A);
	printf("请输入矩阵B：\n");
	B = CreatMLink();
	print_MLinkMat(B);
	T1 = TransposeMat(A);
	T2 = TransposeMat(B);
	printf("A的转置：\n");
	print_MLinkMat(T1);	
	printf("B的转置：\n");
	print_MLinkMat(T2);
D = MutiplyMat(A, B);
	printf("AB的积：\n");
	print_MLinkMat(D);
	C = AddMat(A, B);
	printf("AB的和：\n");
	print_MLinkMat(C);
}

