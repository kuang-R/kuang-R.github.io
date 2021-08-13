我一直都对C语言情有独钟，可能是《C语言点滴》这本书的影响吧。glib库作为一整个gnome的底层架构，在实现了类似于c++ std库的功能下，还包括了文件系统，多线程，定时器这些跨平台的方便组件。在跨语言领域上，也有完善的g-object系统bind各种语言。如果需要编写一个完善的库，可以说使用glib是非常合适的。我准备过一遍glib参考手册，然后再读一读源码，写一些使用这个库的使用方法和抽象概念。

前几天glib官方的参考手册移到了gtk的域名下进行管理，整个界面变得现代化了很多。加上之前的gtk官网更新，感觉gtk系列也要慢慢地崛起了。当然，跟qt系的使用率还是没法比。

基础数据结构可以类比c++的std标准库，数据结构只包括glib库的一部分。论使用的便利当然是不如c++的，但加上glib的其它各种组件，用起来还是很不错。

### GArray
类似于c++中的vector，但是功能没有那么强大。new的时候需要指定数组元素的大小，但是不需要指定元素个数。也就是说new出来的GArray都是空数组。而且也不能一个一个地添加元素，只能添加数组。内置有排序，二分查找，引用计数，各种删除操作。

将这个数据结构跟vector进行对比其实是不合适的，vector致力于取代c的数组，而GArray则是对c数组进行补充。从只能append_vals中就可以看出来，用法大约是先暂存在c数组里，然后一次性添加到GArray中，再进行各种操作。当然，一个一个地添加也是可以的。

``` c
#include <glib.h>
gint comp(gconstpointer a, gconstpointer b)
{
	double x = *(double *)a;
	double y = *(double *)b;
	if (x < y)
		return -1;
	else if (x > y)
		return 1;
	else
		return 0;
}
int main()
{
	GArray *arr = g_array_new(FALSE, FALSE, sizeof(double));
	double t[] = {1., 5., 6., 8., 3., 9.};

	g_array_append_vals(arr, t, sizeof(t)/sizeof(double));
	g_array_sort(arr, comp);

	g_print("len: %d\n", arr->len); // len: 6
	double *p = (double *)arr->data;
	/* 1.000000 3.000000 5.000000 6.000000 8.000000 9.000000 */
	for (int i = 0; i < arr->len; i++)
		g_print("%lf ", p[i]);
}
```

### GByteArray
字节数组，是GArray的子集，元素类型固定为Byte，在使用上方便很多。

### GPtrArray
保存指针的数组，根据new的方法决定free的时候是否把指针指向的内容也给free掉。

### GString
我没在参考手册中的GString对应方法中找到new，所以用了g_malloc创建GString，结果g_print没办法输出了。看来这些数据结构如果没用专门的new创建的话，会产生不可知的效果。

惯例的增删查改操作，值得注意的是明确指定使用utf-8编码，也就是说c源文件必须是utf-8的编码吧。

### GStringChunk
貌似是用来管理一大堆字符串？insert之后会复制对应内存到这个数据结构里面，然后返回对应的字符串指针。因为是统一分配销毁，所以内存开销会比用GString小很多，但我想象不出来需要用到它的场景。

### GHashTable
以Hash表的形式实现的map或者是set，创建的时候需要指定Hash函数和判等函数。因为内部存储的是指针，所以在销毁对象时需要free。当然，也可以在new时指定销毁函数。insert时如果key在表中已存在，只更新value，replace则会更新key和value。

``` c
#include <glib.h>
#include <string.h>
guint gs_hash(gconstpointer key)
{
	const GString *k = key;
	return g_str_hash(k->str);
}
gboolean eq(gconstpointer a, gconstpointer b)
{
	return strcmp(a, b) == 0;
}
void gs_free(gpointer data)
{
	g_print("destory key\n");
	g_string_free((GString *)data, TRUE);
}
void print_dict(gpointer key, gpointer value, gpointer user_data)
{
	const GString *s = key;
	g_print("\"%s\": %d\n", s->str, *(int *)value);
}
int main()
{
	/* like set<string, int> */
	GHashTable *dict = g_hash_table_new_full(gs_hash, eq, gs_free, g_free);

	for (int i = 0; i < 1000; i++) {
		GString *st = g_string_new("");
		int *it = g_malloc(sizeof(int));

		g_string_printf(st, "%d", i);
		*it = i;
		g_hash_table_insert(dict, st, it);
	}

	g_hash_table_foreach(dict, print_dict, NULL);
	g_hash_table_destroy(dict);
}
```

这东西写起来有够累人……还有就是，insert会覆盖相同值的key，在这里被覆盖掉的值如果在创建时没有指定DestroyNotify函数，会产生内存泄漏。

### Bytes
不可变的字节序列，主要以引用计数的方式使用。按照官方文档所说，用GBytes作为key可以跟GHashTable和GTree很好地结合使用。有意思的是它有几种不同的new方法。

看到这里，我认为glib肯定有一种方法能方便地简化这些累人的写法，可能是引用计数或者是其它的什么。看来有需要对glib的内存管理和类型系统来一个概括性的探索。

``` c
#include <glib.h>
static int s_i = 100;
int main()
{
	GBytes *gb = g_bytes_new(&s_i, sizeof(int));
	GBytes *st = g_bytes_new_static(&s_i, sizeof(int));

	g_print("new: %d\n", *(int *)g_bytes_get_data(gb, NULL)); // new: 100
	g_print("new_static: %d\n", *(int *)g_bytes_get_data(st, NULL)); // new_static: 100

	s_i = 200;
	g_print("new: %d\n", *(int *)g_bytes_get_data(gb, NULL)); // new: 100
	g_print("new_static: %d\n", *(int *)g_bytes_get_data(st, NULL)); // new_static: 200

	g_bytes_unref(gb);
	gb = NULL;
	g_bytes_unref(st);
	st = NULL;
}
```

### GList
双向链表。找了一番，竟然没有找到new方法，结果到源码单元测试里看才知道根本不用初始化。没有元素就是直接NULL，也就是说这种链表并没有头结点这种东西。

``` c
#include <glib.h>
int main(int argc, char **argv) {
	GList *list = NULL;
	g_print("len: %d\n", g_list_length(list)); // len: 0
	list = g_list_append(list, "test");
	g_print("len: %d\n", g_list_length(list)); // len: 1
	g_list_free(list);
}

```

### GSList
单向链表，用法同上

### GQueue
先进先出队列，没什么特别的。

### GTree
额，第一次见有库提供平衡二叉树结构，详细探索一下。

## 参考
[Manage C data using the GLib collections](https://developer.ibm.com/tutorials/l-glib/)，2005年的一篇glib指导，详尽且全面，真正的大佬。

[GLib – 2.0 - GTK Documentation](https://docs.gtk.org/glib/index.html)
