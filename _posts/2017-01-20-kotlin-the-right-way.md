---
layout: post
title: "Kotlin Common Practices"
description: "Kotlin is becoming more popular nowadays, I discovered many people coding the same old way they code Java. So here I am demonstrate some Kotlin common practices to help you write Kotlin the right way."
lang: zh
---

现在越来越多的人开始跟风使用Kotlin作为他们Android开发的第二语言，但在使用的时候却完全是Java的风格，给我的感觉就像连官方文档都没有看就拿起来开发了，真是可惜了这门语言，所以写这篇文章的目的就是列举使用Kotlin的正确姿势，帮助Android程序员在写代码时Thinking in Kotlin。

这篇文章会列举实战中的例子，并不会像官方文档一样一步步详细介绍Kotlin，想学习Kotlin请移步[这里](https://kotlinlang.org/docs/reference/)。

我找了一个Github上用Kotlin写的项目，[wechat\_no\_revoke](https://github.com/rarnu/wechat_no_revoke)（下面称作wnr），这个项目问题还是蛮多的，下面的文章主要就指出其中的问题（只是语言使用上的问题，不涉及它实现的功能–防微信撤销），并提供更好的解决方法。

### use let instead of if(sth != null)

wnr中充斥着`if (sth != null) { doSomething() }`（看`WechatRevokeHook.kt`），Kotlin提供的标准库，可以完美解决这种冗余代码：所有用if判断对象是否为null的都用let代替：

> sth?.let { doSomething() }

`let`体内的代码只有在sth不为null时才会运行。实际上这个`let`不是什么Kotlin编译器里的特殊语法，它只是一个普通的函数，你自己都可以实现一个`let`，下面是标准库里的实现：

> public inline fun <T, R> T.let(block: (T) -> R): R = block(this)

`let`是一个任意类型T上的扩展函数，参数是一个Lambda，然后再在Lambda上调用自己。这样说可能理解起来不那么容易，其实`let`是一个[scoping function](https://en.wikipedia.org/wiki/Scope_%28computer_science%29)，它保护了`let`体内的变量不泄露到外界，null检查只是附带的功能，举个例子：

```kotlin
DbConnection.getConnection().let { connection ->

}
// connection到这里就访问不到了
```

### use apply to simplify your code

在`WecahtDatebase.kt`中，有这样一段代码：

```kotlin
val v = ContentValues()

v.put("msgid", msgId)
v.put("content", msg)

if (talkerId != \-1) {
  v.put("talkerid", talkerId)
}
insert("message", "", v)
```

同样Kotlin标准库中提供了`apply`函数，专门用来应付这种命令式的代码。

```kotlin
ContentValues().apply {
  put("msgid", msgId)
  put("content", msg)
  if (talkerId != \-1) {
    put("talkerid", talkerId)
  }
  insert("message", "", this)
}
```

同样apply也不是什么神奇的东西，它只是个普通的函数：

> fun <T> T.apply(f: T.() -> Unit): T { f(); return this }

`apply`定义了一个所有类型上的扩展方法，调用`apply`的时候，会调用传进去的闭包，并返回在闭包上运行过的receiver对象。其实不是那么复杂，看下面的例子你就懂了：

```kotlin
//把string转为File对象，对此对象调用mkdirs()方法，最后返回此对象
File(dir).apply { mkdirs() }

//下面是等同的Java代码
File makeDir(String path) {
  File result = new File(path);
  result.mkdirs();
  return result;
}
```

能用一行解决的就不要用多行。

既然标准库说了这么多，就顺便说完吧：

```kotlin
//如果那要对同一个对象多次调用不同的方法，就用with
fun <T, R> with(receiver: T, f: T.() -> R): R = receiver.f()

val w = Window()
with(w) {
  setWidth(100)
  setHeight(200)
  setBackground(RED)
}

//用run表示链式调用（run是with和let的合体）
fun <T, R> T.run(f: T.() -> R): R = f()
"123".run { print(this) }
	.run { print("hehe") }   //输出"123hehe"

//用use得到与java try-with-resources一样的效果（资源会自动close），注意这里的use也只是个普通的函数而已，不像java一样要编译器用特殊的语法才能做到：

fun readProperties() = Properties().apply {
  FileInputStream("config.properties").use {
    fis ->
      load(fis)
    }
}

//下面是java 1.7及以上才有的try-with-resources
Properties prop = new Properties();
try (FileInputStream fis = new FileInputStream("config.properties")) {
  prop.load(fis);
}
// fis automatically closed
```

### use ? to indicate Nullable carefully

wnr中有这样一段代码（看`MessageUtil.kt`）：

```kotlin
fun extractContent(replace: String?, str: String?): String? {
  var _replace = replace!!
  var _str = str!!
  //... do something with \_replace and \_str
  return _replace
}
```

`!!`是程序员知道对象不可能为null时才用来强转为非null变量的（如果是null程序就崩了），而既然知道不可能为null，那为什么还要用`?`来表示参数可能为null呢，而且返回值居然带问号，excuse me？互相矛盾…无语。这样的问题充斥这整个项目，完全是乱的。

```kotlin
val str1 = "str1" //str1类型为\`String\`，不可能为null

var str2: String? = null //str2类型为\`String?\`，现在初始化是null，以后也可能是null

str2 = "some value"  //str2还是\`String?\`，只不过现在值不是null了

val str3 = str2!! //str3类型为\`String\`，这里这能在你确定str2不是null的情况下才能用，编译器并不能保证str2不是null
```

### use first class function instead of object

既然上面说到MessageUtil了，那顺便说说这个问题。在Kotlin中，函数也是第一公民，下面是wnr中的代码：

```kotlin
//MessageUtil.kt:
object MessageUtil {
  fun extractContent(......): String? {
  }
}

//Some other file:
content = MessageUtil.extractContent(replaceMsg, content)!!
```

就不说这个`!!`了，上面说过，全是乱的。我就说说比较明显的问题，这个object MessageUtil完全是多余的，在FP里，函数是第一公民，意味着你不必把方法写在类里，函数也是值，可以做参数，可以当返回值，可以独立于类存在（其实编译成class后函数也在类里，不过这对用户来说是隐藏的）。

### use primary constructor instead of java style constructor

下面的是wnr中的`WechatRevokeHook.kt`

```kotlin
class WechatRevokeHook {
  var \_v: WechatVersion? = null
  constructor(ver: WechatVersion) {
    _v = ver
  }
}
```

相信大多数人只要看过文档都不会写出这样的代码, primary constructor替代：

```kotlin
class WechatRevokeHook(val ver: WechatVersion) {
  ......
}
```

### use Delegates to initialize field

```kotlin
class MainActivity : Activity(),... {
  private var tvVersion: TextView? = null
  private var tvProj: TextView? = null
  private var tvRepo1: TextView? = null
  private var tvRepo2: TextView? = null

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.main)
    tvVersion = findViewById(R.id.tvVersion) as TextView?
    tvProj = findViewById(R.id.tvProj) as TextView?
    tvRepo1 = findViewById(R.id.tvRepo1) as TextView?
    tvRepo2 = findViewById(R.id.tvRepo2) as TextView?
  }
}
```

在下有一万种方法优化这坨代码（笑），首先，最简单的，用lazy delegate：

> val tvVersion by lazy { findViewById(R.id.tvVersion) as TextView }

这样tvVersion只有在第一个用的时候才会初始化，以前分离的声明与初始化合在的一起，只有一行，更加优美，便于理解，而且没有null的烦恼，tvVersion既是val（不会变），又是TextView（没有`?`，不可能是null），更加安全。

但是这里代码还是有点长，又要写lazy，又要强转View为TextView，这些代码我都不想写，有没有更简单的写法呢？答案是肯定的，只需要自己实现一个类似lazy的Delegate就可以了，注意，这里的lazy不是编译器里什么神奇的东西，它也是一个方法。

```kotlin
//ButterKnife.kt
public fun <V : View> Activity.bindView(id: Int)
	: ReadOnlyProperty<Activity, V> = required(id, viewFinder)

private val Activity.viewFinder: Activity.(Int) -> View?
    get() = { findViewById(it) }

private fun <T, V : View> required(id: Int, finder: T.(Int) -> View?)
	= Lazy { t: T, desc -> t.finder(id) as V? ?: viewNotFound(id, desc) }

// Like Kotlin's lazy delegate but the initializer gets the target and metadata passed to it
private class Lazy<T, V\>(private val initializer: (T, KProperty<\*>) -> V) : ReadOnlyProperty<T, V> {
    private object EMPTY
    private var value: Any? = EMPTY
    override fun getValue(thisRef: T, property: KProperty<\*>): V {
	if (value == EMPTY) {
	    value = initializer(thisRef, property)
	}
	@Suppress("UNCHECKED_CAST")
	return value as V
    }
}
```

这样再在Activity里，就可以这样用：

> val tvVersion by bindView<TextView>(R.id.tvVersion)

再也没有findViewById的烦恼啦。

上面那段代码来自Jake Wharton，是的，Jake Wharton用一个文件就解决了Butterknife Java版解决的问题，我曾经深入的研究过Java版Butterknife，还写了一个类似Butterknife的工具，Butterknife要在编译期用AnnotationProcessor处理java文件中的annotation，然后利用javapoet生成代码，在生成的代码中findViewById并进行绑定，其中涉及到的apt以及代码生成会影响到编译速度，而Kotlin版并没有这些问题，都是函数调用而已。

### DSL

就Kotlin展开的话，涉及到了Functional Programming和DSL，后者我可以简单介绍下，毕竟不同语言构造DSL的方式都不大相同。

不少人都用Retrofit，就拿Retrofit举个例子吧，下面是一个简单的Retrofit DSL，最终的效果是这样的：

```kotlin
fun httpService(base: String) =
	retrofit {
	    client {
		readTimeout = sc(100)
		connectTimeout = sc(100)
		headers {
		    "Api-Version" with "dim"
		    "Origin" with "hehe"
		    "Token" with PreferencesUtils.getToken()
		}
		sslCert(Application.getInstance().applicationContext) {
		    strong = true
		    certs = listOf(R.raw.https)
		}
	    }
	    hostUrl = ServerUtil.getCurrentServerBase();
	    baseUrl = base
	    converterFactories = listOf(GsonConverterFactory.create())
	}

val services: EnumPoll<TEST, Retrofit, String>
    get() = poll {
	mapping {
	    TEST.A with "baseUrlA/"
	    TEST.B with "baseUrlB/"
	}
	instance = ::httpService
    }

fun testRetrofitDSL() {
    testservice<ApiTest>().testBaidu().enqueue {
	onResponse { res ->
	    //doSomethingWith Res
	}
	onFailure { throwable ->
	    //doSomethingWhen failure
	}
    }
}
```

上面是合法的Kotlin代码，函数httpService返回一个Retrofit实例，client返回一个OkHttpClient，其中包含了header、https（SSL）等设置，services是包含Retrofit实例和baseUrl的HashMap，会重复利用有相同baseUrl的Retrofit对象，Api的baseUrl是通过annotation反射获得的，annotation的参数是个Enum，testRetrofitDSL是最终使用时的例子。还是蛮有意思的。

### 总结

Kotlin虽然入门简单，但是实际用起来还是需要使用者花点心思的，不然写出来的代码就是换个样子的Java。Functional Programming、DSL、Extention Function、Null Safety这些概念虽然不是什么新的点子，但Kotlin却用自己独特的实现方式，让这些特性都有很好的用武之地，不能好好利用就很可惜了。