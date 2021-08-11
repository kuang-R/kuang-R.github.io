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

### GBytes
跟HashTable有点关系

