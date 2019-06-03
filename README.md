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

## 自定义事件

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<my-components @my-event='doSomeThings()'></my-components>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let myComponents = {//局部组件通常我们用一个变量来存储
		data : function(){
			return {
				
			}
		},
		template : `
			<button @click="event()">i am component</button>
		`,
		methods : {
			event : function(){
				this.$emit('my-event');
			}
		}
	}
	let app = new Vue({
		el : '#app',
		components : {
			'my-components' : myComponents
		},
		data : {
			message : "hello"
		},
		methods : {
			doSomeThings : function(){
				console.log('do Some Things');
			}
		}
	});
</script>
</body>
</html>
```

先说说使用的步骤吧

1.子组件中先绑定一个事件（例如点击事件）

2.然后点击促发一个函数（我用的是event），然后在函数中调用this.$emit('my-event');//这个是自定义事件名

3.然后在使用的时候<my-components @my-event='doSomeThings()'></my-components>

4.在父组件中书写doSomeThings的逻辑

总结：

​	感觉这个自定义事件，并不是想象中的自定义一个点击事件，感觉就是子组件与父组件的通信方式之一（感觉自己之前想偏了）

## watch监听

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<input type="" name="" v-model='message'>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let app = new Vue({
		el : '#app',
		data : {
			message : "hello"
		},
		watch : {
			message : function(newValue,oldValue){//感觉优点坑的就是监听的函数与
            //属性名称是一样的，有点无语，但后来想想，既然是监听某个属性，没有必要多命名
				console.log(newValue,oldValue);
			}
		}
	});
</script>
</body>
</html>	
```

## computed

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		{{changeMessage}}
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let app = new Vue({
		el : '#app',
		data : {
			message : "hello"
		},
		computed : {
			changeMessage : function(){
				return this.message.toUpperCase();
			}
		}
	});
</script>
</body>
</html>
```

computed叫计算属性，个人理解，如果数据如需经过某些加工之后，才显示的话，使用计算属性。也许有人说，为什么不直接在methods中定义一个方法来加工数据呢？效果不是一样的吗？虽然效果一样，但computed是由缓冲的。看以下例子：

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		{{changeMessage}}
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let app = new Vue({
		el : '#app',
		data : {
			message : "hello"
		},
		computed : {
			changeMessage : function(){
				return this.message.toUpperCase() + Date.now();
			}
		},
		methods : {
			getMessage : function(){
				return this.message.toUpperCase() + Date.now();
			}
		}
	});

	setInterval(function(){
		console.log(app.changeMessage);//缓存的证据，可以证明只要this.message没有改变，返回值就不会变
	},500);

	setInterval(function(){
		console.log(app.getMessage());
	},500);


</script>
</body>
</html>
```

# Day 04(Vue学习)
## is

先上个代码

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<component-a></component-a>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let componentA = {
		data : function(){
			return {}
		},
		template : `
			<p>123</p>
		`
	}
	let app = new Vue({
		el : '#app',
		components : {
			componentA
		}
	});
</script>
</body>
</html>
```

上诉代码主要想表达组件的使用方法。我们想要使用一个组件，有局部和全局（上诉代码是局部组件）。为什么讲 is 之前先讲组件的使用呢？因为 is 就是关系到动态组件的使用。

下面先上一个静态 is 的用法

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<div is='componentA'>
			
		</div>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let componentA = {
		data : function(){
			return {}
		},
		template : `
			<p>静态is的使用</p>
		`
	}
	let app = new Vue({
		el : '#app',
		components : {
			componentA
		}
	});
</script>
</body>
</html>
```

现在使用动态 is

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<div :is='component'>
			
		</div>
		<button @click='toggle'>toggle</button>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let componentA = {
		data : function(){
			return {}
		},
		template : `
			<p>我是组件componentA</p>
		`
	}
	let componentB = {
		data : function(){
			return {}
		},
		template : `
			<p>我是组件componentB</p>
		`
	}
	let app = new Vue({
		el : '#app',
		data : {
			component : 'componentA'
		},
		components : {
			componentA,
			componentB
		},
		methods : {
			toggle : function(){
				this.component = (this.component === 'componentA') ? 'componentB' : 'componentA';
			}
		}
	});
</script>
</body>
</html>
```

## 组件通信
先上一个图
![image](https://github.com/1044375652/-/blob/master/component.svg)

先简单的说以下（浅入）

父组件向子组件通信，涉及到props属性。子组件向父组件通信，涉及到$emit属性的使用。

先是父组件向子组件通信，代码如下

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<component-a my-prop='父组件向子组件通信'></component-a>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let componentA = {
		props : ['my-prop'],
		data : function(){
			return {}
		},
		template : `
			<p>{{myProp}}</p>
		`
	}
	let app = new Vue({
		el : '#app',
		data : {
			component : 'componentA'
		},
		components : {
			componentA
		}
	});
</script>
</body>
</html>
```

子组件的props可以是数组（上述代码就是数组），也可以是对象。使用对象的话，可以设置很多东西，例如类型检测、自定义验证和设置默认值

代码演示

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<component-a my-prop='父组件向子组件通信'></component-a>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let componentA = {
		props : {
			myProp : {
				type : String,
				default : 0,
				required : true,
				validator : function(value){
					console.log(value);
					return true;
				}
			}
		},
		data : function(){
			return {}
		},
		template : `
			<p>{{myProp}}</p>
		`
	};
	let app = new Vue({
		el : '#app',
		data : {
			component : 'componentA'
		},
		components : {
			componentA
		}
	});
</script>
</body>
</html>
```

错误演示

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<component-a my-prop='父组件向子组件通信'></component-a>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let componentA = {
		props : {
			myProp : {
				type : 'String',
				default : 0,
				required : true,
				validator : function(value){
					console.log(value);
					return true;
				}
			}
		},
		data : function(){
			return {}
		},
		template : `
			<p>{{}}</p>
		`
	};
	let app = new Vue({
		el : '#app',
		data : {
			component : 'componentA'
		},
		components : {
			componentA
		}
	});
</script>
</body>
</html>
//报错 Uncaught TypeError: Right-hand side of 'instanceof' is not an object
//原因 type : 'String',应该是 type : String,
```

上述就是父组件向子组件通信的其中一个方法；

下面是子组件向父组件通信的一种方法（其实就是使用自定义事件）

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<component-a @my-event='getMessage'></component-a>
		<p>{{message}}</p>
	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	let componentA = {
		data : function(){
			return {
				childMessage : ''
			}
		},
		template : `
			<div>
				<input type='text' v-model='childMessage'>
				<button @click='sumbitMessage()'>点击向父组件通信</button>
			</div>		
		`,
		methods : {
			sumbitMessage : function(){
				this.$emit('my-event',this.childMessage);
			}
		}
	};
	let app = new Vue({
		el : '#app',
		data : {
			message : ''
		},
		components : {
			componentA
		},
		methods : {
			getMessage : function(message){
				console.log(message);
				this.message = message;
			}
		}
	});
</script>
</body>
</html>
```

有个地方需要注意一下

```javascript
<component-a @my-event='getMessage()'></component-a> --- 第一种
<component-a @my-event='getMessage'></component-a> --- 第2种
```

以前一直使用的是第一种，因为一直没有传递参数，发现没什么所谓，但是后来需要传递参数的时候，就出现问题了。第一种会一直无法接收到参数，因为  getMessage() 就没有传参数。而第2中就可以接收到参数了。


## 其他几种通信方式
>引用掘金上的：
用过 Vue 技术栈开发项目过的开发者对这样一个组合(props与$emit)肯定不会陌生，这种组件通信的方式是我们运用的非常多的一种。props 以单向数据流的形式可以很好的完成父子组件的通信。
所谓单向数据流：就是数据只能通过 props 由父组件流向子组件，而子组件并不能通过修改 props 传过来的数据修改父组件的相应状态。至于为什么这样做，Vue 官网做出了解释：
所有的 prop 都使得其父子 prop 之间形成了一个单向下行绑定：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。额外的，每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。

props与$emit通信其实是由缺陷的，如果出现了父亲要向孙子或更下一级通信，那么就十分麻烦
例如:
![image](https://user-gold-cdn.xitu.io/2019/2/28/16933d8755b48f7a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
<br/>A向B，B向C,B向D通信使用props,$emit是不错的选择，但如果A向C,A向D，就不是那么好了。
所以，出现了以下这种方式通信
* 1 $attrs 和 $listeners
* 2 v-model
* 3 $parent 和 $children
* 4 Vuex 状态管理

## $attrs 和 $listeners
$attr

官网的解释

> 包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (`class` 和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

$listeners

> 包含了父作用域中的 (不含 `.native` 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件——在创建更高层次的组件时非常有用。

上述文字得配合代码才看的懂

上代码

```javascript
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">

	</div>

<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
<script type="text/javascript">
	Vue.component('A', {
		template: `
			<div>
				<p>this is parent component!</p>
				<B :messageC = "messageC" :messageB="messageB" v-on:CData="CData" v-on:BData="BData"></B>
			</div>
		`,
		data() {
			return {
				messageB: 'B',
				messageC: 'C' //传递给c组件的数据
			}
		},
		methods: {
		// 执行B子组件触发的事件
			BData(val) {
				console.log(`这是来自B组件的数据：${val}`);
			},

			// 执行C子组件触发的事件
			CData(val) {
				console.log(`这是来自C组件的数据：${val}`);
			}
		}
	});

	// 组件B
	Vue.component('B', {
		template: `
			<div>
				<input type="text" v-model="myMessage" @input="passData(myMessage)">
				<C v-bind="$attrs" v-on="$listeners"></C>
			</div>
		`,
		props: ['messageB'],
		data(){
			return {
				myMessage: this.messageB
			}
		},
		methods: {
			passData(val){
			//触发父组件中的事件
				this.$emit('BData', val);
			}
		}
	});

	// 组件C
	Vue.component('C', {
		template: `
			<div>
				<input type="text" v-model="$attrs.messageC" @input="passCData($attrs.messageC)">
			</div>
		`,
		methods: {
			passCData(val) {
			// 触发父组件A中的事件
				this.$emit('CData',val);
			}
		}
	});

	var app=new Vue({
	el:'#app',
	template: `
		<div>
			<A />
		</div>
	`
	});
</script>
</body>
</html>
```

配合代码一起理解

$attr其实对应得就是props，那这句

> 包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定

我们观察一个 B 组件，B组件的 props 仅仅接收了 messageB 属性，但是在传参数的时候，传了 messageB 与 messageC 。所以，$attr 包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定，即是 messageC。

那么 $listeners 也是一样的，不过 $listeners 对应的是自定义事件罢了。

## $children与$parent

```html
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<div id="app">
		<p>父组件的{{message}}</p>
		<button @click='click()'>change son</button>
		<component-b></component-b>
	</div>
	<script src="https://cdn.bootcss.com/vue/2.6.10/vue.js"></script>
	<script type="text/javascript">
		let componentB = {
			data : function(){
				return {
					message : '123'
				}
			},
			methods : {
				click : function(){
					this.$parent.message = Date.now();
				}
			},
			template : `
				<div>
					<p>{{message}}</p>
					<button @click='click()'>change father</button>
				</div>
			`
		}
		let app = new Vue({
			el : '#app',
			data : {
				message : 'father message'
			},
			methods : {
				click : function(){
					this.$children[0].message = Date.now();//修改子组件的message
				}
			},
			components : {
				'componentB' : componentB
			}
		});
	</script>
</body>
</html>
```

