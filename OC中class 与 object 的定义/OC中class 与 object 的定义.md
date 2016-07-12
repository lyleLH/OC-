#类和对象的内部结构(runtime基础)

## 引子

	
###NSObject类

	NSObject.h头文件中
```
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}

```

NSObject类的结构体只有一个成员isa,虽然其使用的是Class isa

这个Class是什么？


进入objc.h中查看objc_class结构体的定义，会发现:

	objc.h中
	
```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```


###Class指针 
Class指针本身就是一个结构体

是一个指向objc_class结构体的指针。

再来看 
### objc_class 结构体
	runtime.h中
	
```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```

objc_class结构体里面存储着一堆的指针。对于OC2.0可以简化为:

```
struct objc_class {

    Class isa  OBJC_ISA_AVAILABILITY;
};
    
```

如果大家用c写过链表，可以看出这其实还是一个链表结构。

通过isa的数据成员可以形成一个链
通过super_class也可以形成一个链。






## id指针和objc_object结构体

上面声明Class为objc_class结构体指针时，还声明了一个id指针

	objc.h中
```
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;

```
id是一个指向objc_object结构体的指针
对比一下

其成员isa是一个指向objec_class结构体的指针。


通过删除一些无关的内容及进行替换，可以得到如下的NSObject结构。

```
@interface NSObject{
    struct objc_class* isa;
}

```

也就是说
- NSObject的数据成员就只有一个指向objc_class结构体的指针。

对比一下

- objc_ object 的数据成员 也是只有一个指向objc_class的一个指针


		数据结构NSObject，objc_class，objc_object都
		仅有一个objc_class * 类型，
		也就是Class类型的变量isa
		可见isa这个变量是多么的重要啊
结论

		在objc的runtime中
		类是用objc_class结构体表示的
		对象是用objc_object结构体表示的。
		对象的isa用来标示这个对象是哪个类的实例

		

## 其他

##objec_class中各成员
###isa
	objec_object（对象）中isa指针指向的类结构称为class（也就是该对象所属的类），
	其中存放着普通成员变量与对象方法 （“-”开头的方法）；
	然而此处isa指针指向的类结构称为metaclass，
	其中存放着static类型的成员变量与static类型的方法 （“+”开头的方法）。
	
	
###super_class
	指向该类的父类的指针，
	如果该类是根类（如NSObject或NSProxy），那么super_class就为NULL。
###metaclass
	所有的metaclass中isa指针都是指向根metaclass，
	而根metaclass则指向自身
	根metaclass是通过继承根类产生的
	与根class结构体成员一致
	不同的是根metaclass的isa指针指向自身
	
<p>
	1、当我们调用某个对象的对象方法时，它会首先在自身isa指针指向的类（class）methodLists中查找该方法，如果找不到则会通过class的super_class指针找到其父类，然后从其methodLists中查找该方法，如果仍然找不到，则继续通过 super_class向上一级父类结构体中查找，直至根class；
</p>

<p>
2、当我们调用某个类方法时，它会首先通过自己的isa指针找到metaclass，并从其methodLists中查找该类方法，如果找不到则会通过metaclass的super_class指针找到父类的metaclass结构体，然后从methodLists中查找该方法，如果仍然找不到，则继续通过super_class向上一级父类结构体中查 找，直至根metaclass；
	</p>
	



 
	
## OC中类与对象的继承层次关系

![image](file:OC中类与对象的继承层次关系.png)

