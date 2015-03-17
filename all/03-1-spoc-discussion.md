# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- 最先匹配：优点：简单；在高地址空间有大块空闲分区；    缺点：容易产生外部碎片；分配大块时较慢。
- 最优匹配：优点：大部分分配的尺寸较小时效果很好，可避免大的空闲分区被拆分，可减小外部碎片的大小，相对简单；
-           缺点：外部碎片；释放分区较慢；容易产生很多无用的小碎片；
- 最差匹配：优点：中等大小的分配较多时，效果较好；避免出现太多的小碎片；
-           缺点：释放分区较慢；容易产生外部碎片；容易破坏大的分区，后续难以分配大的分区。
- 伙伴系统：优点：结合了前面算法的各种优点，折中了各自的缺点，释放相对简单，并且不会产生太大的碎片。
-           缺点：算法相对复杂，合并地址块判断条件较多，切地址块也消耗一定时间。
- 新算法：  结合最优匹配和最先匹配，生成一种有条件的最先匹配：假设要分配的内存块大小为k，如果从前往后找，找到一块不大于2k且不小于k的空闲内存，就分给它；如果找不到符合条件的块，就用最先分配。优点和伙伴系统类似，不过不需要切较大的地址块，释放合并的时候也不用看大小和首地址的限制。

>  

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

```
如何表示空闲块？ 如何表示空闲块列表？ 
[(start0, size0),(start1,size1)...]
在一次malloc后，如果根据某种顺序查找符合malloc要求的空闲块？如何把一个空闲块改变成另外一个空闲块，或消除这个空闲块？如何更新空闲块列表？
在一次free后，如何把已使用块转变成空闲块，并按照某种顺序（起始地址，块大小）插入到空闲块列表中？考虑需要合并相邻空闲块，形成更大的空闲块？
如果考虑地址对齐（比如按照4字节对齐），应该如何设计？
如果考虑空闲/使用块列表组织中有部分元数据，比如表示链接信息，如何给malloc返回有效可用的空闲块地址而不破坏
元数据信息？
伙伴分配器的一个极简实现
http://coolshell.cn/tag/buddy
```
代码：
```
#include <iostream>
#include <vector>

using namespace std;

struct free_block{
	int start;
	int size;
};

vector<free_block> free_area;
vector<free_block>::iterator it;
/*
struct pmm_manager {
    const char *name;                                 // XXX_pmm_manager's name
    void (*init)(void);                               // initialize internal description&management data structure
                                                      // (free block list, number of free block) of XXX_pmm_manager 
    void (*init_memmap)(struct Page *base, size_t n); // setup description&management data structcure according to
                                                      // the initial free physical memory space 
    struct Page *(*alloc_pages)(size_t n);            // allocate >=n pages, depend on the allocation algorithm 
    void (*free_pages)(struct Page *base, size_t n);  // free >=n pages with "base" addr of Page descriptor structures(memlayout.h)
    size_t (*nr_free_pages)(void);                    // return the number of free pages 
    void (*check)(void);                              // check the correctness of XXX_pmm_manager 
};
*/
//worst fit allowcation

void m_sort(vector<free_block> a){
	for(int i = 0; i < a.size() - 1; i ++)
		for(int j = i + 1; j < a.size(); j ++){
			if(a[i].size < a[j].size){
				free_block temp = a[i];
				a[i] = a[j];
				a[j] = temp;
			}

		}
	return;
}
void n_sort(vector<free_block> a){
	for(int i = 0; i < a.size() - 1; i ++)
		for(int j = i + 1; j < a.size(); j ++){
			if(a[i].start > a[j].start){
				free_block temp = a[i];
				a[i] = a[j];
				a[j] = temp;
			}

		}
	return;
}

static free_block *
m_alloc_pages(int size) {

	m_sort(free_area);
	free_block * ret;

	if(free_area[0].size == size){
		ret = new free_block;
		ret -> start = free_area[0].start;
		ret -> size = free_area[0].size;
		it = free_area.begin();
		free_area.erase(it);
	}else if(free_area[0].size > size){
		ret = new free_block;
		ret -> start = free_area[0].start;
		ret -> size = size;
		free_area[0].start = free_area[0].start + size;
		free_area[0].size = free_area[0].size - size;
	}
	else{
		ret = NULL;
	}
    return ret;
}

static void
m_free_pages(int start, int size) {
    //n_sort(free_area);
    free_block * temp = new free_block;
    temp -> start = start;
    temp -> size = size;
    free_area.push_back(*temp);

    n_sort(free_area);
    int flag = -1;
    for(int i = 0; i < free_area.size(); i ++){
    	if(free_area[i].start==temp->start){
    		flag = i;
    		break;
    	}
    }

	if(free_area[flag].start + free_area[flag].size == free_area[flag+1].start){
	    	free_area[flag].size = free_area[flag].size + free_area[flag+1].size;
	    	it = free_area.begin() + flag + 1;
	    	free_area.erase(it);
	    }
    if(free_area[flag-1].start + free_area[flag-1].size == free_area[flag].start){
    	free_area[flag].start = free_area[flag-1].start;
    	free_area[flag].size = free_area[flag-1].size + free_area[flag].size;
    	it = free_area.begin() + flag - 1;
    	free_area.erase(it);
    }

    return;
}

void m_print(){
	for(int i = 0; i < free_area.size(); i ++){
		cout << "" << free_area[i].start << " " << free_area[i].size << "\t";
	}
	cout << "\n";
	return;
}

int main(){
	char op;
	int input1, input2;

	free_area.clear();
	free_block init;
	init.start = 0;
	init.size = 1024;
	free_area.push_back(init);
	
	while(1){
		cin >> op;
		if(op == 'q'){
			break;
		}else if(op == 'a'){
			cin >> input1;
			m_alloc_pages(input1);
			m_print();
		}else if(op == 'f'){
			cin >> input1 >> input2;
			m_free_pages(input1, input2);
			m_print();
		}else{
			cout << "Input Error!\n";
			continue;
		}
	}
}
```
运行结果：
```
a 64
64 960
a 64
128 896
a 128
256 768
a 256
512 512
f 64 64
512 512 64 64
a 64
576 448 64 64
q
```
可见最后一次分配64大小的块时，系统分配了后面那块大的，符合最差匹配的原理。

--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



