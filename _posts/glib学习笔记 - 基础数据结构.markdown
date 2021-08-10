我一直都对C语言情有独钟，可能是《C语言点滴》这本书的影响吧。glib库作为一整个gnome的底层架构，在实现了类似于c++ std库的功能下，还包括了文件系统，多线程，定时器这些跨平台的方便组件。在跨语言领域上，也有完善的g-object系统bind各种语言。如果需要编写一个完善的库，可以说使用glib是非常合适的。我准备过一遍glib参考手册，然后再读一读源码，写一些使用这个库的使用方法和抽象概念。

前几天glib官方的参考手册移到了gtk的域名下进行管理，整个界面变得现代化了很多。加上之前的gtk官网更新，感觉gtk系列也要慢慢地崛起了。当然，跟qt系的使用率还是没法比。

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

### AsyncQueue
异步队列，用于多线程操作。这个队列保存的是指针，也就是说要线程自己管理资源的分配和释放。很多方法都有对应的unlocked版本，这是必须在持有队列锁（也就是调用了lock）之后调用的版本，否则可能会产生死锁。

我猜unlocked是为了加快调用速度，避免很多push这种操作对信号量申请过多。

``` c
#include <glib.h>
gpointer producer(gpointer data)
{
	GAsyncQueue *async_queue = data;
	for (int i = 0; TRUE; i++) {
		int *t = g_malloc(sizeof(i));
		*t = i;
		g_async_queue_push(async_queue, t);
	}
}
gpointer consumer(gpointer data)
{
	GAsyncQueue *async_queue = data;
	while (TRUE) {
		int *t = g_async_queue_pop(async_queue);
		g_print("%d ", *t);
		g_free(t);
	}
}
int main()
{
	GMainLoop *main_loop = g_main_loop_new(NULL, FALSE);
	GThread *th1, *th2;
	GAsyncQueue *async_queue = g_async_queue_new();

	th1 = g_thread_new("Producer", producer, async_queue);
	th2 = g_thread_new("Consumer", consumer, async_queue);

	g_main_loop_run(main_loop);
}
```
