# -Day 01(阅读ES6)
## let/const与var

let/const与var不同的是，let/const会创建一个块级作用域（通俗一点就是一个花括号）。在if/for中，var会存在变量提升的情况，但let/const不会。

举例
```javascript
for(var i = 0 ; i < 10 ; i++){

​	//TODOS

}

 console.log(i)//10
```
---------------------------

```javascript

for(let i = 0 ; i < 10 ; i++){

​	//TODOS

}

 console.log(i)//Uncaught ReferenceError: i is not defined

```
出现这情况是因为var在循环中，只会声明一次变量，经过10次循环，i的值会叠加到10，

而let在每次循环中，都会重新声明一次，并且存在块级作用域中，所以才访问不到

## 暂时性死区

暂时性死区的意思是，在变量声明之前，是无法使用该变量的(使用了就报错)

举例

```javascript

console.log(i);//undefined

var i = 0;

------

console.log(i);//Uncaught ReferenceError: Cannot access 'i' before initialization

let i = 0;
```

在var声明变量，可以提前使用，是因为变量提升。而在let中声明变量，因为暂时性死区，变量必须要声明才能使用，不然就报错！
暂时性死区的作用其实就是为了代码规范，让开发者习惯先声明，后使用变量的习惯
## const

const与let类似，但const声明过的变量，是不可改变的！

如果const声明的是一个常量（命名规范，全是大写字母，字母之间的间隔用下划线），那必须在声明的时候就赋值，并且之后该变量的值不可变

如果const声明的是一个引用类型的，该变量的地址不可改变，但值可以改变

举例

```javascript

const i = 1;

i = 2;

//Uncaught TypeError: Assignment to constant variable.

------

const b = {
 'a' : 1
}

console.log(b.a);

b.b = 2;

b.c = 3;

console.log(b);

//正确，不会有问题
```

## 箭头函数

箭头函数与ES5的function关键字的区别

1.箭头函数没有arguments属性(因为访问arguments的性能较差)，使用了剩余运算符替代

```javascript
let test2 = {
	'method' : (...args)=>{
		console.log(args);
	}
}
```

2.箭头函数没有prototype与constructor，所以不能使用new

3.箭头函数的this是绑定的，不像function的this那么飘渺不定的

举例

```javascript
let test = {
    'method' : function(){
        console.log(this);
    },
    'method2' : ()=>{
        console.log(this);
    },
    's' : {
        's' : {
            's' : {
                'method' : function(){
                    console.log(this);
                },
                'method2' : ()=>{
                    console.log(this);
                }
            }
        }
    }
}

test.method()//test对象
test.method2()//window对象
test.s.s.s.method()//第三个s对象
test.s.s.s.method2()//windows对象

```
## 补充（箭头函数的this到底指向谁？）

举例

如果都是使用箭头函数的话，this就指向最外层的windows

```javascript
const obj = {
    b: {
    	c: () => {console.log(this)}
	}
}
obj.b.c()  //打出的是window对象！！
```

如果箭头函数与function混合使用的化，那么就有点麻烦（所以建议，要么都是function，要么都是箭头函数）

```javascript
const obj = {
    a : function(){
        let b = {
            c: () => {console.log(this)}
        }
        b.c();    
    },
    b: {
        c: () => {console.log(this)}
    }
}
obj.a()  //打出的是obj对象！！
obj.b.c()  //打出的是window对象！！
```

## 补充（箭头函数的this是如何绑定的呢？）

先说以前的function，ES5的this绑定时在运行时绑定的，例如

```javascript
const obj = {
    'a' : 1,
    'method' : function(){
    console.log(this.a);
    }
}
obj.method();//运行的时候才确定this为obj对象
obj.method.call({'a' : 2});//使用call改变this的指向，为一个新的对象{'a': 2}
//箭头函数使用call绑定对象是无效的！，因为箭头函数就是为了减少this的复杂性，
//所以箭头函数使用call的话，第一个参数会给忽略掉的
```

而箭头函数的this是在编译的时候就确定了，箭头函数的this是通过继承获取的

```javascript
const obj = {
    'method' : ()=>{
        const obj = {
            'method' : ()=>{
            	console.log(this);//输出都是window对象
            }
            obj.method();
        }
    }
}
obj.method();
```
## yield关键字（先看这里，不然下面的迭代器部分看不懂）

在MDN上，对yield的第一句解释就是：

The `yield` keyword is used to pause and resume a generator function.
yield这个关键字是用来暂停和恢复一个遍历器函数（的运行）的。

个人理解，yield与return非常相似，都有返回值的功能，但是，

return是结束，yield是暂停（关键）。

既然yield是暂停，那么他肯定能回到暂停前的部分。

举例

```javascript
var foo = function *() { 
    var x = 1;
    var y = yield (x + 1);//标记一
    var z = yield (x + 2);//标记二
    return z;//标记三
}();
var a = foo.next();
var b = foo.next();
var c = foo.next();
```

接下来分析以下这段程序：

在第一个a = foo.next()的时候，程序运行到yield (x + 1)，yield的特性，暂停（不会运行下面的代码）且返回值，所以a.value = 2

到 b = foo.next()，程序从结束的地方开始（即 yield (x + 1)）开始，运行 yield (x + 2)，然后同理暂停（不会运行下面的代码）且返回值，所以b.value = 3

到 c = foo.next()，程序从结束的地方开始（即 yield (x + 2)）开始，return z，然后程序结束了

还有一个是yield传参的问题

举例

```javascript
var foo = function *() { 
    var x = 1;
    var y = yield (x + 1);//标记一
    var z = yield (x + y);//标记二
    return z;//标记三
}();
var a = foo.next();
var b = foo.next(1);
var c = foo.next(2);
```

第一次运行没有什么特殊之处，

第二次运行我们给next传了一个参数，1.首先同理，我们会回到之前暂停的地方，yield (x + 1)，本来接下来应该是运行 var z = yield(x + y)的，但是我们传了一个参数，1，程序就会把上一个 yield ( x + 1)赋值为1（整个部分），然后y = 1了，然后才运行下面的 var z = yield(x + y)，所以b.value = 2



最后，yield经常搭配生成器一起使用，接下来会说到，借用一句话 “yield这个关键字是用来暂停和恢复一个遍历器函数（的运行）的”（经典）

## 迭代器

为什么要使用迭代器？

这是一个循环语句的问题

举例

```javascript
var colors = ['red','green','blue'];
    for(var i = 0;i < colors.length;i++){
    console.log(colors[i]);
}
```

我们这个循环，是通过i变量作为数组的索引，从而找到元素的。问题就来了，万一哪天有很多循环层嵌套，一不小心弄错了索引，这就麻烦了。

迭代器的出现就是为了消除这种复杂性以及减少循环中的错误

什么是迭代器呢？

用ES5语法来模拟创建一个迭代器

```javascript
function createIterator(items){
    var i = 0;
    return {
        next : function(){
            var done = (i >= items.length);
            var value = !done ? items[i++] : undefined;
            return {
                value : value,
                done : done
            }
        }
    }
}
var iterator = createIterator([1,2,3]);
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```

所以，我们可以这样定义，迭代器是一个拥有next方法的对象，并且在调用next方法后，会返回一个{value : xxx,done : true|false}的结果对象

也许看起来很复杂，但是只要写过一次迭代器的代码，然后接下来的循环就可以无限复用了


## 生成器

是什么？生成器是一个返回迭代器的函数。前面我们手动创建过一个迭代器的函数，那个就是生成器

利用ES6我们可以很快速的创建一个生成器

```javascript
function *createIterator(items){
    for(let i = 0 ;i < items.length;i++){
        yield items[i];//yield 上面提到过
    }
}
var iterator = createIterator(['red','green','blue']);
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```

## 可迭代对象

是什么？就是拥有迭代器的对象。在ES6中，所有的集合对象（数组，Set，Map）和字符串都是可迭代对象，可迭代对象都绑定了默认的迭代器

不得fo不提及以下ES6的 for ... of 循环

```javascript
let colors = ['red','green','blue'];
for(value of colors){
    console.log(value);
}
```

for ... of 循环的原理就是利用了迭代器，首先在 value of colors中，其实就是判断 done是否为false，

然后输出的value，其实就是迭代对象的value属性

所以，我们可以写出以下代码

```javascript
function createIterator(items){
    let i = 0;
    return {
        next : ()=>{
            let done = (i >= items.length);
            let value = !done ? items[i++] : undefined;
            return value;
        }
    }
}
let iterator = createIterator(['red','green','blue']);
while(value = iterator.next()){
console.log(value);
}
```

所以，可迭代对象其实都有一个默认迭代器，可用 Symbol.iterator方法获得

```javascript
let colors = ['red','green','blue'];
let iterator = colors[Symbol.iterator]();
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```

## 解构

是什么？就是按照一定的格式，从数组或对象或函数的返回值中，提取值，并把它赋值到对应的变量上去

## 扩展运算符

```javascript
let arr = [1,2,3,4];
let arr2 = [...arr,5,6,7];
console.log(arr2);
```

顺便提一下数组的深拷贝

为什么会出现浅拷贝？

因为数组是用堆去保存的，相等的时候只是把存放的地址拷贝过去了，两个指向了同一个地址，所以在改变其中一个的值，其他的也跟着改变了。

所以如何深拷贝？

最原始的办法就是变量 source数组，然后赋值到 target 数组上

可以利用原始的 for循环，for... in 循环（在for ... in 与 for ... of循环中，for ... of 循环的性能好，因为 for ... in 循环会把对象的prototype一起复制了，而 for ... of 仅复制当前对象的属性），或返回值为数组的方法（map，concat，slice）

上述所说的前提条件，该数组的值都是基本类型。如果有引用类的话，都变成了浅拷贝

所有又出现了一种方法 JSON 对象的 parse 和 stringify 

```javascript
let arr = [1,2,3];
let arr2 = JSON.parse(JSON.stringify(arr)); 
```

但这也有问题，拷贝过后的function以及construtor会丢失

最终办法是递归！（目前未写的出来）


## 剩余运算符

最常用的就是当作箭头函数的参数

```javascript
let method = (...args)=>{
    console.log(args);
}
```

# -Day 02(阅读ES6)
## Promise

首先得提一下，javascript是单线程的，但是如果要实现异步编码，如果做到呢？在很久很久以前，我们是通过回调函数来实现的

例如

```javascript
function method(cb){
    setTimeout(function(){
        console.log('method');
        cb();
    },1000);
    ...
}

function method2(){
	console.log('method2');
}
method(method2);
```

这样就简单的实现了异步编程的效果（在method完成后，才调用method2，而不影响method内其余代码的运行）

但是问题就来了，上面仅仅只是一级回调而已，如果出现多级回调呢？代码就会变得十分难看(传说中的回调地狱)。所以，就出现了promise。

promise干了什么呢？他封装了回调方式的异步编码，使得代码更加有次序，并且做了更多的处理（例如捕获异常等）

# Day 03(Vue学习)
## 组件

全局组件

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<my-components></my-components>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	Vue.component('my-components',{//第一个参数为组件名（注意，组件名采用中划线分割）
		data : function(){
			return {
				
			}
		},
		template : `
			<p>i am component</p>
		`
	});
	let app = new Vue({
		el : '#app',
		data : {
			message : "hello"
		}
	});
</script>
</body>
</html>
```



局部组件



```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<my-components></my-components>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let myComponents = {//局部组件通常我们用一个变量来存储
		data : function(){
			return {
				//为什么局部组件采用这种方式返回data?
                 //如果我们要用到局部组件，那么实例化的每一个局部组件都是一个独立的个体
                 //如果仅仅是 data : {...}这样返回的话，更改其中一个data，就会引起其他				//组件的变化，所以采用function(){返回一个新的对象}
			}
		},
		template : `
			<p>i am component</p>
		`
	}
	let app = new Vue({
		el : '#app',
		components : {
			'my-components' : myComponents
		},
		data : {
			message : "hello"
		}
	});
</script>
</body>
</html>
```


