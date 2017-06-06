/* Select the victim */

```c
 /*LAB3 EXERCISE 2: YOUR CODE*/ 
 //(1)  unlink the  earliest arrival page in front of pra_list_head qeueue
 //(2)  set the addr of addr of this page to ptr_page
 list_entry_t *le = head->prev; // 这个是找到最后的一个节点
 assert(head != le);
 struct Page *p = le2page(le, pra_page_link);
 list_del(le); // 从链表中删除这个项
 assert(p != NULL);
 *ptr_page = p; // 记录下要被替换的页面
 return 0;
```
//record the page access situlation

```c
/*LAB3 EXERCISE 2: YOUR CODE*/ 
//(1)link the most recent arrival page at the back of the pra_list_head qeueue.
list_add(head, entry); // 将entry插入到head之后
return 0;
```
需要在每个页面对应的pte中加入clock的标志位，在fifo算法的基础上，增加对clock位的判断来选择画出的页面就是extended clock替换算法

特征是最近未曾被访问修改过。

在遍历页面时增加对标志位判断

页面访问异常时换入缺的页，换出最近未被访问修改的页。也可以主动将久未使用的页面换出