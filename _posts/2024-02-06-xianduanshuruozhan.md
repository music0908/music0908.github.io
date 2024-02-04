---
layout: post
title: 线段树拓展
date: 2024-02-04
Author: music0908
categories: true
tags: [OI, 线段树, 数据结构]
comments: true
--- 

# 线段树拓展

![库库尔坎](https://cdn.luogu.com.cn/upload/image_hosting/lvyze04d.png)

### I.权值线段树与动态开点线段树

区别于普通线段树所记录的区间和，权值线段树维护的是一个序列的**权值数组**。

- 权值数组所记录的是**某一元素在整个序列中出现的次数**。例如，在 1,1,3,5,6,4,8 中，将每个元素记为 a[i] ，权值数组记为 b[i] ，我们要得到数字 1 出现的次数时所得的结果便是 b[1]=2 。显然地，我们会发现
  $$
  b_i=\sum^n_{j=1} [a_j=i]
  $$
  因而权值数组的大小与序列元素的大小（值域）有直接关系，在序列的值域巨大时，权值数组的空间占用是很可怖的。为了解决这个问题，我们使用了**动态开点线段树**。

动态开点线段树的原理与离散化类似，用标号解决**因过于稀疏而产生的空间过大问题**，其本质上是将操作的点在使用时再创造。因此，用以记录左右儿子的 i >> 1 和 i >> 1|1 便失效了。于是我们使用创造并更新左右儿子编号的方式来记录这棵树。*大抵上，这与链式前向星有些近似。*

建树：

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN=1e5+5;
const int M=1e7;
#define ls (t[i].l)
#define rs (t[i].r)
#define mid ((l+r)>>1)
int n;
int tot;
int root;
struct tree{
	int v,l,r;
};
tree t[MAXN<<4];
```

此处的$M$为偏移量，在含有负数的动态开点线段树中，一般会使用一个大小为值域中最小值的绝对值作偏移量，并给每个元素加上偏移量。这一操作可以视作将整个线段树向上移动。

单点修改

```cpp
inline void newnode(int &i){//动态开点
	if(!i){
		i=(++tot);
	}
}
```

```cpp
inline void update(int &i,int l,int r,int k,int cnt){//单点修改
    //i为区间编号，l~r为当前遍历区间，k为加入或删除的元素，cnt为加入或删除的个数
    //删除时，直接令cnt=-|删除的个数|
	newnode(i);//将每一个第一次操作的点创造编号
	t[i].v+=cnt;//权值线段树中只要有经过就会有贡献
	if(l==r){
		if(t[i].v<0){
			t[i].v=0;//防止在未创造的节点上删除元素
		}
		return;
	}
	if(k<=mid){//k在左半区间，包含mid
		update(ls,l,mid,k,cnt);
	}
	else{
		update(rs,mid+1,r,k,cnt);
	}
}
```

区间查询

```cpp
inline int rnk(int &i,int l,int r,int finall,int finalr){//区间求和，1~k的区间和即k的排名
    //i为当前节点编号，l~r表示当前遍历区间，finall~finalr表示查询区间 
	if(!i){//遍历到不存在的节点
		return 0;
	}
	if(finall<=l&&finalr>=r){//所求区间完全包含当前区间 
		return t[i].v; 
	}
	int res=0;//当前遍历区间对查询区间的贡献
	if(finall<=mid){
		res+=rnk(ls,l,mid,finall,finalr);
	}
	if(finalr>mid){
		res+=rnk(rs,mid+1,r,finall,finalr);
	}
	return res;
}
```

对于res的计算，要保证两侧都能被判断到。为了防止遍历区间完全包括查询区间，且查询区间位于遍历区间中部时，贡献只记录一侧的情况出现。对于左侧判定和右侧判定，**都不可加else**。

给定排名求元素

```cpp
inline int kth(int &i,int l,int r,int k){//给排名求元素
   	//i为当前遍历区间编号，l~r表示当前遍历区间，k为给定的排名
	if(l==r){
		return l;
	}
	if(t[ls].v>=k){//如果当前节点所代表的左子树的元素个数大于当前排名，则该排名所代表元素必在左子树中
		return kth(ls,l,mid,k);
	}
	else{
		return kth(rs,mid+1,r,k-t[ls].v);
        //在右子树中的元素的排名是重新被记录的，故而要将当前排名剪掉不会对结果再有影响的左子树的元素个数
	}
} 
```

前驱与后继

前驱： k 的前驱的定义为小于 k ，且最大的数

后继： k 的后继的定义为大于 k ，且最小的数

```cpp
kth(root,1,M<<1,rnk(root,1,M<<1,1,x+M-1))-M;
kth(root,1,M<<1,rnk(root,1,M<<1,1,x+M)+1)-M;
```

鉴于 kth 所求索的是一个数在排列中的总排名，故而在寻找后继时只用输出 k 的

##### 完整模板：

```cpp
//普通平衡树-动态开点权值线段树
#include<bits/stdc++.h>
using namespace std;
int n;
const int MAXN=1e5+5;
#define ls (t[i].l)
#define rs (t[i].r)
#define mid ((l+r)>>1)
const int M=1e7;
struct sg{
	int v,l,r;
};
sg t[MAXN<<4];

int tot;
int root;
inline void newnode(int &i){
	if(!i){
		i=(++tot);
	}
}
inline void update(int &i,int l,int r,int k,int cnt){
	newnode(i);
	t[i].v+=cnt;
	if(l==r){
		if(t[i].v<0){
			t[i].v=0;
		}
		return;
	}
	if(k<=mid){ 
		update(ls,l,mid,k,cnt);
	}
	else{
		update(rs,mid+1,r,k,cnt);
	}
}
inline int rnk(int &i,int l,int r,int finall,int finalr){ 
	if(!i){
		return 0;
	}
	if(finall<=l&&finalr>=r){
		return t[i].v; 
	}
	int ans=0;
	if(finall<=mid){
		ans+=rnk(ls,l,mid,finall,finalr);
	}
	if(finalr>mid){
		ans+=rnk(rs,mid+1,r,finall,finalr);
	}
	return ans;
}
inline int kth(int &i,int l,int r,int k){
	if(l==r){
		return l;
	}
	if(t[ls].v>=k){
		return kth(ls,l,mid,k);
	}
	else{
		return kth(rs,mid+1,r,k-t[ls].v);
	}
} 
int main(){
	scanf("%d",&n);
	while(n--){
		int op,x;
		scanf("%d%d",&op,&x);
		switch(op){
			case 1:{
				update(root,1,M<<1,x+M,1);
				break;
			}
			case 2:{
				update(root,1,M<<1,x+M,-1);
				break;
			}
			case 3:{
				printf("%d\n",rnk(root,1,M<<1,1,x+M-1)+1);
				break;
			}
			case 4:{
				printf("%d\n",kth(root,1,M<<1,x)-M);
				break;
			}
			case 5:{
				printf("%d\n",kth(root,1,M<<1,rnk(root,1,M<<1,1,x+M-1))-M);
				break;
			}
			case 6:{
				printf("%d\n",kth(root,1,M<<1,rnk(root,1,M<<1,1,x+M)+1)-M);
				break;
			}
		}
	}
	return 0;
}
```

###### 完。

> 参考：
>
> 权值线段树详解-xiezheyuan  https://www.cnblogs.com/zheyuanxie/p/wsgt.html
>
> 浅谈权值线段树-bf https://www.luogu.com.cn/blog/bfqaq/qian-tan-quan-zhi-xian-duan-shu
>
> 动态开点线段树-OI Wiki https://oi-wiki.org/ds/seg/#%E5%8A%A8%E6%80%81%E5%BC%80%E7%82%B9%E7%BA%BF%E6%AE%B5%E6%A0%91

### II.线段树合并

线段树合并，顾名思义，即是将两棵线段树合为一棵，其合并方式共分为两种。

- 
