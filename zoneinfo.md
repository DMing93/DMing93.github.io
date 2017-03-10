#编写内核模块——打印内存中区的相关信息
内核把物理页作为内存管理的基本单元。内存管理单元（MMU）通常以页为单位进行处理。但由于硬件的限制，内核并不能对所有的页一视同仁。有些页位于内存中特定的物理地址上，所以不能将其用于一些特定的任务。
由于存在这种限制，所以内核把页划分为不同的区（zone）。内核使用区对具有相似特性的页进行分组。
通过内核模块将区的名称打印出来，可以方便我们理解其中的联系。
##zoneinfo模块
	zoneinfo.c
    #include <linux/init.h>
    #include <linux/module.h>
    #include <linux/mmzone.h>
    MODULE_LICENSE("Dual BSD/GPL");
    
    static int zone_init(void)
    {
    	struct pglist_data *node;
        node = NODE_DATA(0);
        struct zone zone;
        int i;
        for(i = 0;i < MAX_NR_ZONES; i++)
        {
        	zone = node ->node_zones[i];
            printk(KERN_ALERT "%s\n", zone.name);
        }
    	return 0;
    }
    static void zone_exit(void)
    {
    	printk(KERN_ALERT "Module has been uninstalled\n");
    }
    module_init(zone_init);
    module_exit(zone_exit);
    
上面是zoneinfo模块的代码，定义了两个函数，zone\_init在模块被装载到内核时调用，zone\_exit在模块被移除时调用。MODULE\_LICENSE这个宏用来告诉内核，该模块采用自由许可证。

X86体系结构采用UMA架构，所有的处理器共享全部物理内存，所以将内存视为一个结点，而描述这个结点的结构体是pglist_data,结点号为0，所有的分区建立在这个结点之下。

node指针指向描述结点结构体的首地址，根据 #define NODE\_DATA(nid)          (&contig\_page_data)，返回结点结构体地址，赋值给node指针。

定义struct zone 类型的变量 zone,结点的结构体中包含描述分区的数组node\_zones[MAX_NR_ZONES],利用for循环遍历所有区，并将其信息打印出来。

node->node_zones[i]，将结构体中的成员node\_zones[]赋值给zone变量，这个数组中包含描述区的结构体struct\_ zone

最后将描述分区的结构体struct zone 里的name信息打印出来。

在模块被移除时，打印信息“Module has been uninstalled”。

##编译过程
	Makefile文件
    obj-m := zoneinfo.o
编译

	make -C /usr/src/linux-headers-4.4.0-66-generic/ M=/home/dming/ modules
    
插入模块
	
    sudo insmod zoneinfo.ko
    
显示打印信息

	dmesg

![分区信息](http://i.imgur.com/TtrkEuW.png)
移除模块

	sudo rmmod zoneinfo.ko
    
![卸载模块](http://i.imgur.com/8euC502.png)