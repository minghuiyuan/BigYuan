## Golang 可变长参数

今天遇到一个难懂的函数：
```
func FuncName1(para1 string, para2 int, isValidFuncs ...func(*para4.para5) bool) func(obj interface{}) bool {
	return func(obj interface{}) bool {
		switch t := obj.(type) {
		case *para4.para5:
			namespace, err := para2.Get(t.GetNamespace())
......

			return isValidFuncs[0](t)
		default:
			runtime.HandleError(fmt.Errorf("unable to handle object %+v with type %v", obj, t))
			return false
		}
	}
}
```
第一次看到这个函数很懵逼，因为刚学Golang的我看到函数的参数"..."，又看到...func(...)等,很晕，所以就专门花时间把Golang的可变参数列表给研究一下。

现在先把上例的函数给详解一下:
- 函数的第一参数为string类型,第二个参数是int类型，后面的是func类型，而且他是支持传入多个func,就类似于func的slice.
- 函数返回值为一个返回为bool类型的func,而且传入参数为一个可以转化为任意类型的interface{}类型
  
### golang 支持可变长参数列表
比如：
```
func Testfunc(num int, args ...interface{}){
    fmt.Println(num, args)
}
```
代码逻辑很简单：

1. Testfunc 接受一个 int 参数，一个不定长的参数，并且类型为 interface{}
2. args 做为 slice，使用 ... 语法糖打散后传入 TestArgs

执行main.go函数调用：
```
func main(){
    nums:=[]int64{1,2,3,4}
    Testfunc(0,nums)
}
```
结果出错，为什么？？？

> 因为类型不匹配，Golang要求类型必须相同

可变参数就是一个占位符，你可以将1个或者多个参数赋值给这个占位符，这样不管实际参数的数量是多少，都能交给可变参数来处理，我们看一下可变参数的声明：
```
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```
可变参数使用**name ...Type**的形式声明在函数的参数列表中，而且需要是参数列表的**最后一个参数**，这点与其他语言类似；

可变参数在函数中将转换为对应的**[]Type**类型，所以我们可以像使用slice时一样来获取传给函数的参数们；

有一点值得注意，golang的可变参数*不需要强制绑定参数*的出现。

我们只有先指定至少一个固定的形参（num）才能使用...可变参数，在golang中是不需要这样做的：
```
func sum(nums ...int) int {
    //todo
}
```

### 传递参数给...可变参数

传递参数给带有可变参数的函数有两种形式，第一种与通常的参数传递没有什么区别，拿上一节的sum举个例子：

除了参数的个数是动态变化的之外和普通的函数调用是一致的。

第二种形式是使用...运算符以变量...的形式进行参数传递，这里的变量必须是与可变参数类型相同的slice，而不能是其他类型（没错，数组也不可以），看个例子：
```
numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
sum(numbers...) // 和sum(1, 2, 3, 4, 5, 6, 7, 8, 9. 10)等价
```
这种形式最常用的地方是在内置函数append里：

```
result := []int{1, 3}
data := []int{5, 7, 9}
result = append(result, data...) // result == []int{1, 3, 5, 7, 9}
```
是不是和python的解包操作很像，没错，大部分情况下你可以把...运算符当做是golang的unpack操作，不过有几点不同还是要注意的：

第一，只能对slice类型使用...运算符：
```
arr := [...]int{1, 2, 3, 4, 5}
sum(arr...) // 编译无法通过
```

你会见到这样的报错信息：cannot use arr (type [5]int) as type []int in argument to sum

这是因为可变参数实际是个slice，...运算符是个语法糖，它把前面的slice直接复制给可变参数，而不是先解包成独立的n个参数再传递，这也是为什么我只说...运算符看起来像unpack的原因。

第二个需要注意的地方是不能把独立传参和...运算符混用，再看个例子：

```
slice := []int{2, 3, 4, 5}
sum(1, slice...) // 无法通过编译
```
 这次你会见到一个比较长的报错：

```
too many arguments in call to sum
    have (number, []int...)
    want (...int)
```
这是和前面所说的原因是一样的，...运算符将不定参数直接替换成了slice，这样就导致前一个独立给出的参数不再算入可变参数的范围内，使得函数的参数列表从(...int)变成了(int, ...int)，最终使得函数类型不匹配编译失败。

正确的做法也很简单，不要混合使用...运算符给可变参数传参即可。

 





> ref : https://www.cnblogs.com/apocelipes/p/9861315.html