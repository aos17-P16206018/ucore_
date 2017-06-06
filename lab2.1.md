

static void
default_init(void) {

```c
list_init(&free_list);
nr_free = 0;
```
}



static void
default_init_memmap(struct Page *base, size_t n) {

```c
assert(n > 0);
// n是页数
struct Page *p = base;
for (; p != base + n; p ++) {
    assert(PageReserved(p));
    p->flags = p->property = 0;
	SetPageProperty(p);
    set_page_ref(p, 0); // 引用计数设置为0
	// 也就是在free_list的前面插入，不过考虑到这是一个双向链表，所以实际的意思是，插到链表的尾部
	list_add_before(&free_list, &(p->page_link));
}
// 因为base是头表
// property这个玩意只有在free block首地址对应的Page有用，其余都被设置为了0
// 用于指示这个free block一共有多少个free page
base->property = n; // 可用的页面数是n
SetPageProperty(base); // base所在的页面为头表
nr_free += n; // 空闲的页面数目增加了n
list_add(&free_list, &(base->page_link)); // 添加到free_list的头部
```
}

static struct Page *
default_alloc_pages(size_t n) {

```c
assert(n > 0);
if (n > nr_free) {
    return NULL;
}
struct Page *page = NULL;
list_entry_t *le = &free_list;
while ((le = list_next(le)) != &free_list) {
    struct Page *p = le2page(le, page_link);
    if (p->property >= n) {
        page = p;
        SetPageReserved(page);
       ClearPageProperty(page);
        break;
    }
}
if (page != NULL) {
    
    list_del(&(page->page_link));
    if (page->property > n) {
        struct Page *p = page + n;
        p->property = page->property - n;
        list_add(&free_list, &(p->page_link));
      
}
    
      nr_free -= n;
}
return page;
//  if (page != NULL) { // 在空闲列表中找到了足够大的块
  //      list_del(&(page->page_link)); // 将这个page从列表中删除
  //      if (page->property > n) { //这里没有等于号
  //          struct Page *p = page + n;
  //          p->property = page->property - n;
  //          list_add(&free_list, &(p->page_link)); // 将剩余的部分重新加入list
		//}
  //      nr_free -= n; // 可以被分配的页的数目减少了n
  //      ClearPageProperty(page); // 大概是标记这个页已经被分配了吧！
  //  }
  //  return page; // 返回page，
```
}

static void
default_free_pages(struct Page *base, size_t n) {

```c
// 用于释放页

assert(n > 0);
assert(PageReserved(base));

list_entry_t *le = &free_list; // le指向头部
struct Page *p;
while ((le = list_next(le)) != &free_list) {
	p = le2page(le, page_link);
	if (p > base) { // free_list里面的数据都是按照地址从小到大排列的吧！
		break;
	}
}
// le现在指向一个恰好在base页面之后的page
for (p = base; p < base + n; p++) { // 不断地在前面插入
	list_add_before(le, &(p->page_link));
}
base->flags = 0;
set_page_ref(base, 0); // 引用计数变为了0
ClearPageProperty(base); // 
SetPageProperty(base); // 只需要这句就行了吧！
base->property = n;

p = le2page(le, page_link);
if (base + n == p) { // 也就是说，前后可以连接起来
	base->property += p->property;
	p->property = 0;
}

le = list_prev(&(base->page_link));
p = le2page(le, page_link); // 找到free_list中base之前的那个free page
if (le != &free_list && p == base - 1) { // 如果两个也可以连起来
	while (le != &free_list) {
		if (p->property) {
			p->property += base->property;
			base->property = 0;
			break;
		}
		le = list_prev(le);
		p = le2page(le, page_link);
	}
}

nr_free += n;
return;
```
}

这个算法中的page结构将所有的页面都连在一起，遍历时会比较慢，可以在新增一个块的数据结构，将连续的空页面只记录头一个页面指针和页面数，则加快了查找速度。