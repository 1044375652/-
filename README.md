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
手册地址<https://cn.vuejs.org/>
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
父子组件之间通信还可以用 $children与$parent 这两个简单粗暴的属性，但是官网不推荐使用，有弊端。$children不是响应式的（这个我没有测出来），并且是无序的。父子组件通信推荐使用 props 与 $emit 属性。


# Day 05(Vue学习)
## slot

直接上代码

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<component-b>
			<p>123</p>
			<p>456</p>
			<p>789</p>
		</component-b>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>

	<script type="text/javascript">
		let componentB = {
			data : function(){
				return {}
			},
			template : `
				<div>
					<p>header</p>
					<slot></slot>
					<p>footer</p>
				</div>
				
			`
		};
		let app = new Vue({
			el : '#app',
			components : {
				componentB
			}
		});
	</script>
</body>
</html>
```

slot翻译过来就是插槽。顾名思义，我们在组件的 template 定义了一个 <slot></slot>标签，然后在使用组件的时候，在组件内部添加了 <p>123</p><p>456</p><p>789</p>（接下来称为A），那么，我们添加的 A 会默认插入到 <slot></slot>标签内。如果想要动态指定插入的位置呢？

继续上代码

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<component-b>
			<p slot='first'>123</p>
			<p slot='first'>456</p>
			<p slot='second'>789</p>
		</component-b>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>

	<script type="text/javascript">
		let componentB = {
			data : function(){
				return {}
			},
			template : `
				<div>
					<p>header</p>
					<slot name='first'></slot>
					<p>footer</p>
					<slot name='second'></slot>
				</div>
				
			`
		};
		let app = new Vue({
			el : '#app',
			components : {
				componentB
			}
		});
	</script>
</body>
</html>
```

所以，想要动态指定位置，我们只需要设置 slot='想要插入位置的名称' 即可

## transition

我们可以通过 CSS 类名与 js 事件控制

先说说 CSS 类名控制
先上一个图
![image](https://cn.vuejs.org/images/transition.png)

> enter-class - string
>
> leave-class - string
>
> appear-class - string
>
> enter-to-class - string
>
> leave-to-class - string
>
> appear-to-class - string
>
> enter-active-class - string
>
> leave-active-class - string
>
> appear-active-class - string

上述是涉及到的类名，一般只用到 enter 与 active

上代码

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <style type="text/css">
    	.fade-enter-active,
    	.fade-leave-active{
    		opacity: 0;
    		transition: all 0.5s;
    	}
    	.fade-enter,
    	.fade-leave-active{
    		opacity: 0;
    	}
    </style>
</head>
<body>
	<div id="app">
		<transition name='fade'>
			<p v-show='show'>123</p>
		</transition>
		<button @click='show = !show'>toggle</button>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>

	<script type="text/javascript">
		let app = new Vue({
			el : '#app',
			data : {
				show : true
			}
		});
	</script>
</body>
</html>
```

- 首先得先给 transition 赋值一个自定义名称（以上是 fade）
- 然后依据 CSS 类名规范，依旧取名 fade-enter-active,fade-leave-active，fade-enter，fade-leave-active（分别对应动画不同的状态）
- 然后在类名中赋值 CSS 动画语句即可
- 自定义类名也是一样的（把 fade 换成自定义类名的名称，CSS 类名也要变

## transition 配合 animate.css 制作动画

直接上代码

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css">
</head>
<body>
	<div id="app">
		<transition 
		enter-active-class="animated bounce" leave-active-class='animated bounceOutRight'>
			<p v-show='show'>123</p>	
		</transition>	
		
		<button @click='show = !show'>toggle</button>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>

	<script type="text/javascript">
		let app = new Vue({
			el : '#app',
			data : {
				show : true
			}
		});
	</script>
</body>
</html>
```

- 首先得引入 animate.css
- 然后根据不同得状态，定义类名
  - 例如
  - enter-active-class ：进入的动画
  - leave-active-class：离开的动画
  - 还有很多...
- 加类名的时候，一定要加上 animated 不然动画无效
- animate 的 github 地址 <https://github.com/daneden/animate.css/>
- animate 的动画演示地址 <https://daneden.github.io/animate.css/>


## transition 与 component

我们可以动态切换 component 并且带上动画，上代码

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css">
</head>
<body>
	<div id="app">
		<transition
		enter-active-class="animated bounce" 
		leave-active-class='animated bounceOutRight'>
			<p :is='currentComponent'></p>	
		</transition>	
		
		<button @click='changeComponent'>toggle</button>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		let componentA = {
			template : `
				<p>I am componentA</p>
			`
		};
		let componentB = {
			template : `
				<p>I am componentB</p>
			`
		};
		let app = new Vue({
			el : '#app',
			data : {
				currentComponent : 'componentA'
			},
			components : {
				componentA,
				componentB
			},
			methods : {
				changeComponent : function(){
					this.currentComponent = (this.currentComponent === 'componentA') ? 'componentB' : 'componentA';
				}
			}
		});
	</script>
</body>
</html>
```

细心的同学会发现，这个动画有点突兀，即我还没出去，新的先进来的。

vue 里的 transition 有一个mode（模式），默认是 'in-out'，意思就是先进来再出去，如果向先出去，然后在进来的话，在 transition 里添加 mode= 'out-in'即可


## transitiion 的问题

上代码

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css">
</head>
<body>
	<div id="app">
		<transition
		enter-active-class="animated bounce" 
		leave-active-class='animated bounceOutRight'>
			<p v-if='show'>123</p>	
			<p v-else>456</p>	
		</transition>	
		
		<button @click='show = !show'>toggle</button>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		let componentA = {
			template : `
				<p>I am componentA</p>
			`
		};
		let componentB = {
			template : `
				<p>I am componentB</p>
			`
		};
		let app = new Vue({
			el : '#app',
			data : {
				show : true
			}
		});
	</script>
</body>
</html>
```

仔细一看好像没什么问题，但是运行跑起来，问题就来了。我的动画效果呢？

经过搜索一番，没有找到答案，但是在手册中找到，应该是答案的答案，上链接<https://cn.vuejs.org/v2/guide/conditional.html>

在官网的条件渲染中，提到 key 管理复用的元素。所以，出现以上效果是因为 Vue 为了尽可能高效渲染元素，通常会采用复用元素的策略。所以要想不让他复用，我们可以通过设置不同的 key 或 不要用两个 p 标签即可。


## transition 与 JS 钩子

> <transition
> v-on:before-enter="beforeEnter"
> v-on:enter="enter"
> v-on:after-enter="afterEnter"
> v-on:enter-cancelled="enterCancelled"
> v-on:before-leave="beforeLeave"
> v-on:leave="leave"
> v-on:after-leave="afterLeave"
> v-on:leave-cancelled="leaveCancelled"
>
> <!-- ... -->
> </transition>

引入官网的一段代码，我们可以通过不同阶段的事件绑定自定义的函数，然后在自定义的函数里制作动画效果即可

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<transition
		v-on:before-enter="beforeEnter"
		v-on:enter="enter"
		v-on:after-enter="afterEnter"
		v-on:enter-cancelled="enterCancelled"

		v-on:before-leave="beforeLeave"
		v-on:leave="leave"
		v-on:after-leave="afterLeave"
		v-on:leave-cancelled="leaveCancelled"
		v-bind:css='false'
		>
			<p v-show='show'>123</p>
		</transition>	
		
		<button @click='show = !show'>toggle</button>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		let app = new Vue({
			el : '#app',
			data : {
				show : true
			},
			methods: {
				beforeEnter: function (el) {
					console.log('beforeEnter');
					console.log(el);
				},
				enter: function (el, done) {
					console.log('enter');
					console.log(el);
					done();
				},
				afterEnter: function (el) {
					console.log('afterEnter');
					console.log(el);
				},
				enterCancelled: function (el) {
					console.log('enterCancelled');
					console.log(el);
				},
				beforeLeave: function (el) {
					console.log('beforeLeave');
					console.log(el);
				},
				leave: function (el, done) {
					console.log('leave');
					console.log(el);
					done();
				},
				afterLeave: function (el) {
					console.log('afterLeave');
					console.log(el);
				},
				leaveCancelled: function (el) {
					console.log('leaveCancelled');
					console.log(el);
				},
				enterdo : function(){
					console.log('我是enter do');
				},
				leavedo : function(){
					console.log('leave do');
				}
			}			
		});
	</script>
</body>
</html>
```

注意：当只用 JavaScript 过渡的时候，**在 enter 和 leave 中必须使用 done 进行回调**。否则，它们将被同步调用，过渡会立即完成。

如果在 enter 与 leave 中没有调用 done()的话，不会出现 afterEnter 与 afterLeave；但是会出现 enterCancelled 与 leaveCancelled

> 
>
> ```html
> <!DOCTYPE html>
> <html>
> <head>
>     <title></title>
> </head>
> <body>
> 	<div id="app">
> 		<button @click="show = !show">
> 			Toggle
> 		</button>
> 		<transition
> 			v-on:before-enter="beforeEnter"
> 			v-on:enter="enter"
> 			v-on:leave="leave"
> 			v-bind:css="false"
> 		>
> 			<p v-if="show">
> 				Demo
> 			</p>
> 		</transition>
> 	</div>
> 	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
> 	<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>
> 	<script type="text/javascript">
> 		let app = new Vue({
> 			el: '#app',
> 			data: {
> 				show: false
> 			},
> 			methods: {
> 				beforeEnter: function (el) {
> 					el.style.opacity = 0;
> 					el.style.transformOrigin = 'left';
> 				},
> 				enter: function (el, done) {
> 					Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 });
> 					Velocity(el, { fontSize: '1em' }, { complete: done });
> 				},
> 				leave: function (el, done) {
> 					Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 });
> 					Velocity(el, { rotateZ: '100deg' }, { loop: 2 });
> 					Velocity(el, {
> 						rotateZ: '45deg',
> 						translateY: '30px',
> 						translateX: '30px',
> 						opacity: 0
> 						}, { complete: done });
> 				}
> 			}
> 		});
> 	</script>
> </body>
> </html>
> ```
>
> 引用官网的一个例子

## 自定义指令

为什么需要自定义指令呢？

有的情况下，你仍然需要对普通 DOM 元素进行底层操作，这时候就会用到自定义指令。

举个例子，页面加载完毕，就对输入框聚焦（只能使用vue哦）

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<input type="" name="" v-focus>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		Vue.directive('focus',{
			inserted : function(el,binding){
				el.focus();
			}
		});
		let app = new Vue({
			el : '#app'
		});
	</script>
</body>
</html>
```

相应的钩子函数

- bind
- inserted
- update
- componentUpdated
- unbind

他们的执行顺序

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<input type="" name="" v-focus>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		Vue.directive('focus',{
			bind : function(el,binding){
				console.log('bind');
			},
			inserted : function(el,binding){
				console.log('inserted');
			},
			update : function(el,binding){
				console.log('update');
			},
			componentUpdated : function(el,binding){
				console.log('componentUpdated');
			},
			unbind : function(el,binding){
				console.log('unbind');
			}
		});
		let app = new Vue({
			el : '#app'
		});
	</script>
</body>
</html>
```

bind > inserted，update 与 componentUpdated 是在指令更新的时候才触发的，unbind 是在指令解除的时候触发的。

有一个问题，就拿上面输入框聚焦的代码来说，我聚焦的代码 el.focus(); 只能放到 inserted 才有用，而在 bind 里却无效，而官方文档说在 bind 里就是用来初始化的，这就有点弄迷糊了。

使用方法：

局部自定义指令

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<p v-color="'red'">123</p>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		let app = new Vue({
			el : '#app',
			directives : {
				color : {
					bind : function(el,binding){
						el.style.color = binding.value;
					},
				}
			}
		});
	</script>
</body>
</html>
```



全局自定义指令

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<p v-color="'red'">123</p>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		Vue.directive('color',{
			bind : function(el,binding){
				el.style.color = binding.value;
			}
		});
		let app = new Vue({
			el : '#app'
		});
	</script>
</body>
</html>
```

在钩子函数中，还有钩子参数，

- el ：指令所绑定的元素，可以用来直接操作 DOM 
- binding ： 一个对象，包含以下属性：
  - name：指令名，不包含 v- 前缀
  - value：指令绑定的表达式的结果，例如 v-color='red == “” '，那么 value 就是 false
  - oldValue：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。
  - expression：指令绑定的表达式，例如 v-color='red == “” '，那么 expression 就是 red == “”
  - modifiers：一个包含修饰符的对象，例如，.lazy修饰符，modifiers就是包含修饰符的容器
- vnode：嘿嘿
- oldVnode：嘿嘿

> ```html
> <!DOCTYPE html>
> <html>
> <head>
>     <title></title>
> </head>
> <body>
> 	<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
> 	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
> 	<script type="text/javascript">
> 		Vue.directive('demo', {
> 			bind: function (el, binding, vnode) {
> 				var s = JSON.stringify
> 				el.innerHTML =
> 					'name: '       + s(binding.name) + '<br>' +
> 					'value: '      + s(binding.value) + '<br>' +
> 					'expression: ' + s(binding.expression) + '<br>' +
> 					'argument: '   + s(binding.arg) + '<br>' +
> 					'modifiers: '  + s(binding.modifiers) + '<br>' +
> 					'vnode keys: ' + Object.keys(vnode).join(', ')
> 				}
> 		});
> 
> 		new Vue({
> 			el: '#hook-arguments-example',
> 			data: {
> 				message: 'hello!'
> 			}
> 		});
> 	</script>
> </body>
> </html>
> ```

自定义指令还有简写的方式

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
	<div id="app">
		<input type="" name="" v-focus>
	</div>
	<script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
	<script type="text/javascript">
		let app = new Vue({
			el : '#app',
			directives : {
				focus : function(el,binding){
					el.focus();			
				}
			}
		});
	</script>
</body>
</html>
```

经过验证，简写的方式，相当于bind

## 使用 vue-cli 构建项目

- 先去百度下载 node
- 现在的 node 都会自带 npm
- 下载好 node 之后，在终端输入 node -v 以及 npm -v 检查是否安装好
- 输入 npm install vue-cli -g
- 安装好了之后，使用 vue init webpack 项目名称
- 好了之后，进入你项目目录（与package.json同级的目录）
- 然后输入 npm install 安装依赖
- 好了之后输入 npm run dev就可以跑起来了
- 个人建议，想使用vue-cli构建项目，前提要对es6了解，并且了解vue，不然看不懂

## 阅读 Vue Router
手册地址<https://router.vuejs.org/><br/>
动态路由匹配

假如，我们有一个 `User` 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染，那么可以这样

```javascript
const router = new VueRouter({
    routes : [
        {
            path:'/user/:id',
            component : User
        }
    ]
});
```

在路径后添加冒号加参数即可。但是这有个问题，就是你如果仅仅只访问 /user，没有带上id，他是识别不了的。所以一般我们都会设置一个错误页面（当访问到不存在的 url 时，需要用到）。

```javascript
const router = new VueRouter({
    routes : [
        {
            path:'/user/:id',
            component : User
        },
        {
        	path : '*',
            component : Error
        }
    ]
});
```

但是注意了，匹配所有的路由，必须放到最后面，因为匹配的优先级是：谁先定义的，谁的优先级就最高。

既然有传参数的方法，那么接受参数的方法也是有的。例如上述代码，跳转到 User 组件，那么在 User 组件里使用 this.$route.params.id 就能获取到值。

## 嵌套路由

先简单的说，我们使用 vue-cli 初始化之后，在 src 目录下会出现 App.vue 这个文件，上代码

```html
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

当你服务跑起来之后，你会发现我原本没那么多代码，为什么会显示那么多呢？。是因为 router-view，我们在 router 下的 index.js 里配置了路由，当访问根（‘/’）的时候，会加载 Hello World 组件。所以才出现那么多东西的。所以总结一下，router-view 就是根据不同的路由，然后显示不同的组件。问题来了，如果 HelloWorld 中也使用了 router-view ，那么这里面的内容是由谁定义的？又怎么定义呢？（这就是嵌套路由）

第一个问题，还是在 index.js 里面定义即可

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import One from '@/components/One'
import Two from '@/components/Two'
import Error from '@/components/Error'

Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/123/:id',
      name: 'One',
      component: One,
      children: [
        {
          path: 'two',
          component: Two
        },
        {
          path: '',
          component: One
        }
      ]
    },
    {
      path: '*',
      component: Error
    }
  ]
})

```

第2个问题，如何定义，例如上述代码

```javascript
export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/123/:id',
      name: 'One',
      component: One,
      children: [
        {
          path: 'two',
          component: Two
        }
      ]
    }
  ]
})
```

在由 router-view 的组件里面添加一个 children ，然后跟之前添加 routes 是一样的。有个小细节，path 使用的是相对路径，即不带 '/'，因为带上 '/'就是绝对路径了，会找不到的

## 导航

我们有声明式导航和编程式导航

先说说声明式

<router-link :to="...">

有点像 HTML 标签，编译之后，在 DOM元素上 <router-link :to="..."> 和 a 标签没有区别

编程式

利用 this.$router.push 方法

```javascript
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

**注意：如果提供了 path，params 会被忽略**

this.$router.push 与 this.$router.replace 类似，但 this.$router.push 会往 history 里添加一条记录，但 this.$router.replace 不会

this.$router.go 其实就是浏览器的前进，后退。

# Day O6(阅读 Vue Router)
## 命名路由

上代码

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'

Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    }
  ]
})

```

这里面的 name : 'HelloWorld' 就是 命名路由

之前我们使用 router-link 跳转是这样的

```html
<router-link :to='{path : "/"}'>根据path跳转到根目录</router-link>
```

现在可以使用这样了

```html
<router-link :to='{name : "One"}'>根据name跳转到根目录</router-link>
```

## 命名视图

视图是 router-view。

有种情况，就是你的组件中有很多 router-view，那么，如何指定哪个视图加载哪个组件呢？

解决办法就是给视图加上名称

上代码

```html
<template>
  <div id="app">
    <!--    <img src="./assets/logo.png">-->
    <router-view name='a'/>
    <router-view name='b'/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>

```

上述代码中，我们分别给两个视图添加了名称（a，b），然后在设置路由的时候这样设置

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import One from '@/components/One'
import Two from '@/components/Two'

Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/123',
      name: 'One',
      components: {
        a: One,
        b: Two
      }
    }
  ]
})

```

## 重定向

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import One from '@/components/One'
import Two from '@/components/Two'
import Error from '@/components/Error'

Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/123',
      name: 'One',
      component: One,
      redirect: '/456'   
    }
  ]
})

```

redirect 不仅可以设置成 path，还可以设置依据 name 来重定向，甚至可以是一个方法（但返回值是一个 url 哦 ）

```javascript
redirect : {name : '某个 name '}
```

```javascript
redirect : function(route){
	//TODO
	return '/url'
}
```

## 别名

我们可以为 path 设置别名

```javascript
{
    path: '/123',
    name: 'One',
    component: One,
    alias: '/456'
}
```

设置别名之后，访问 /123 与 /456 是一样的

## keep-alive

可以使用来缓冲组件

```html
<keep-alive>
	<your-component></your-component>
</keep-alive>
```

keep-alive 常与 router-view 配合使用

```html
<keep-alive>
	<router-view></router-view>
</keep-alive>
```

但是我们直接这样使用，会把所有组件都缓冲下来了。万一我们有些需要缓存，有些又不需要缓冲，则可以这样

```javascript
export default {
  name: 'App',
  meta: {
    keepAlive: true
  }
}
```

甚至里面还有三个钩子给我们使用，（beforeRouteLeave，beforeRouteEnter，beforeRouteUpdate），他们有三个钩子参数（to,from,next），参数的含义很容易理解，关键是 next，我们在做完一些逻辑处理的时候，要调用 next()，不然不会跳转

## 学习 Vuex（官网<https://vuex.vuejs.org/>）

为什么会出现 Vuex ？

Vuex 的出现解决了一下问题：

1. 组件之间的数据通信
2. 使用单向数据流的方式进行数据的中心化管理

举个例子

现在有 A，B，C，D，E，...，N 个组件，A 为根组件，B，C  为 A 的 子组件 ，D，...，N，为 B 的子组件（单二叉树），现在问题来了。如果 A 向 B C 通信，很简单，通过 props 与 $emit 就可以了。B 向 C 通信，麻烦了一点，但也不是不可以做到，B 把信息传给 A ，然后 A 把信息传给 C。那就来了一个极端情况，如果 C 想想 B 的第 N 个 组件通信呢？完了。。所以就出现了 Vuex 来解决这个问题。

数据的中心化又是什么意思呢？

假设有四个组件，他们共享用户信息。如果有一个用户修改了他的信息，那么我就需要同步把最新的信息传给其他组件（如果不更新的话，就出现数据不一致的错误了），这还仅仅是四个组件，如果来了更多的组件呢？所以，我们要把一些数据放到中心，这样实现共享与一致。

需要放到中心的数据一般是：

- 需要共享的数据
- 服务端返回的数据

总结来说，vuex就是用于中大型项目的，小型项目用不上

# Day 07
## 学习 Vuex
### Vuex五大核心属性

- State
- Getter
- Mutation
- Action
- Module

State，根据个人的理解，其实就是对应着以前 Vue 的 data。只不过这个 state 是一个公共的，中心的，所以才能实现数据的中心化。所以，state 其实就是存储我们数据的一个属性。

Getter，根据个人的理解，其实就是对应着以前 Vue 的 computed 属性。引用官网的一句话

> 有时候我们需要从 store 中的 state 中派生出一些状态，例如对列表进行过滤并计数：

熟不熟悉，getter 其实就是如果我们想对 state 渲染（派生）一下才输出，就使用到 getter 属性。

先说 Module ，顾名思义，就是他的中文意思。如果到了后期，我们代码量十分巨大，那么把所有共享的数据都弄到一个 state 上，无疑，这是臃肿的，所以，我们可以划分一下 Module（模块），这样看起来就挺好的。

Mutation，根据个人的理解，其实就是对应着以前 Vue 的 methods。但是，他是同步的（即使顺序执行的），所以，只能在 Mutation 中进行同步操作，如果想要进行异步操作呢？只能用到 Action。

Action，根据个人的理解，其实就是对应着以前 Vue 的 methods。但是，他是异步的。

就有一个疑问，同步异步为什么要分开来呢？因为 vue 有一个工具，叫 dev-tool ，这个工具的用途就是来追踪 vue 状态的，如果使异步操作，我们是不清楚他是什么时候执行的，即无法追踪。所以，我们如果要使用异步操作，必须在 Action 中编写相应的代码，然后给 Mutation 中执行，这样，我们才能追踪到每一个状态。

# Day 08（Vue项目，附上大神的github地址https://github.com/bailicangdu/vue2-happyfri）

## 第一次的update

- 第一个个问题

  - ```javascript
    component: r => require.ensure([], () => r(require('../page/item/index.vue')), 'item')
    ```

  - 第一个看到这个不会，经过百度后，这个是 vue-router 的进阶内容 --- 理由懒加载

  - 出现路由懒加载的原因是：当打包构建应用时，JavaScript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

  - 路由懒加载有两种

  - veu-router 官网文档的

  - ```javascript
    const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
    const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
    const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
    ```

  - github 上 webpack 风格的

  - ```javascript
    const Foo = r => require.ensure([], () => r(require('./Foo.vue')), 'group-foo')
    const Bar = r => require.ensure([], () => r(require('./Bar.vue')), 'group-foo')
    const Baz = r => require.ensure([], () => r(require('./Baz.vue')), 'group-foo')
    ```

  - 最后

  - ```javascript
    const router = new VueRouter({
      routes: [
        { path: '/foo', component: Foo }
      ]
    })
    ```

  - 个人建议 vue-router 官方文档，因为太简便了
  # Day 09

## 第二次的update

这次作者完成了什么？

- 完成了选择题数据的初始化
- 基本页面的渲染

遇到坑的地方：

- 第一个

  - num not found vue-style-loader!!css-loader!!

  - 是因为我在 vue 的 style 中用了 less 语法，但是并没有引入相应的解析器，所以报错

  - ```css
    style lang="less"
    ```

  - 解决

  - ```javascript
    "less": "^3.9.0",
    "less-loader": "^5.0.0",
    ```

  - 引入 less 解析器依赖即可

- 第二个

  - 看到 fatherComponent == 'home' 很疑惑，不过幸好作者写的挺有语义化的，这个的目的就是依据不同的 fatherComponent ，显示不同的 component。如何依据呢？当其他组件引用时，通过 props 通信来判断，从而知道显示那个组件

- 第三个

  - 使用 mapState 方法报错，报错说没有这个 mapState 方法。吓得我赶紧回去看文档，没有拼写错误，难道是文档是错的？不不不，如果你想使用 mapState ，前提的引用 vuex，经过了这次，对引用更有感觉了

  - ```javascript
    import {mapState} from 'vuex'
    ```

  - 感觉这个花括号就是 vuex，然后想引入某个方法，在花括号里面填方法名称即可

- 第四个

  - 对这个语法表示很陌生 {{itemDetail[itemNum-1].title}}

  - 因为依稀记得，在 vuex 中，不能直接 {{itemDetail[itemNum-1].title}} 获取数据，必须的通过 getter 属性来获取数据。

  - 但是，作者已经在 computed 中使用了 mapState 方法了，所以这并不是原始 vue 获取数据的意思

  - ```
    {{itemDetail[itemNum-1].title}}
    
    computed: mapState({
        itemNum: state => {
          return state.itemNum
        },
        level: state => state.level,
        itemDetail: state => state.itemDetail
      })
      下面我们来解析一下代码：
      在调用 itemDetail，其实就调用了 computed 的 itemDetail 方法，这个方法返回了 state.itemDetail 对象，同理 [itemNum-1] 也调用了 computed 的itemDetail 方法
    ```

- 第五个

  - 箭头函数的使用
  - 我在 vue 中使用了箭头函数，发现 this.$store 是 undefined?
  - 这就涉及到了 this 的指向，当你使用了箭头函数，this 就绑定了最外层的对象，即 vue 对象，而我想要的 this.$store 是 vuex 对象的属性，所以就是 undefined，解决办法，想要 vuex 对象使用 function 来声明方法
  - 但是有个小细节，在 vuex 的扩展方法中，例如 mapState 方法，他的第一个参数就是 state(即是 this.$store.state)，所以，在这里使用箭头函数也是可以的

- 第六个

  - ```javascript
    this.$store.dispatch 是什么？
    ```

  - 他类似于 mutations 的 commit 提交，但是， mutations 是同步的，并且，this.$store.dispatch 它是用来分发 actions 的，然后我们在 actions 中才能调用 mutations 的 commit 提交

  - 总而言之，this.$store.dispatch 他是一个异步方法
 

## 第三次的update

这次作者完成了什么？

- 完善了选项（添加了唯一标识）
- 添加了把选择好的选项加入到容器中
- 发送 AJAX 请求

遇到坑的地方：

- ajax请求忘了（复习！）

  - <https://wangdoc.com/javascript/bom/xmlhttprequest.html> 这里讲的很不错，接下来我简单总结一下发起 ajax 请求的步骤

  - 创建 XMLHttpRequest 实例

  - 发出 HTTP 请求

  - 接收服务器传回的数据

  - 更新网页数据

  - ```javascript
    let xhr;
    //这里是创建 XMLHttpRequest 实例
    if(window.XMLHttpRequest){
       xhr = new XMLHttpRequest()
       }else{
           xhr = new ActiveXObject()
       }
    //设置监听事件，看是否请求完毕
    xhr.onreadystatechange = function(){
      // 通信成功时，状态值为4
      if (xhr.readyState === 4){
        if (xhr.status === 200){
          console.log(xhr.responseText);
        } else {
          console.error(xhr.statusText);
        }
      }
    };
    
    xhr.onerror = function (e) {
      console.error(xhr.statusText);
    };
    //通过以上定义好了之后，调用open,send 才是真正发起请求
    xhr.open('GET', '/endpoint', true);
    xhr.send(null);
    ```

  - 有些要说的地方

    - onreadystatechange 是 XMLRequest的一个监听事件，监听此次请求是否完成，相对应的，有这个属性readyState

      - 0，表示 XMLHttpRequest 实例已经生成，但是实例的`open()`方法还没有被调用。
      - 1，表示`open()`方法已经调用，但是实例的`send()`方法还没有调用，仍然可以使用实例的`setRequestHeader()`方法，设定 HTTP 请求的头信息。
      - 2，表示实例的`send()`方法已经调用，并且服务器返回的头信息和状态码已经收到。
      - 3，表示正在接收服务器传来的数据体（body 部分）。这时，如果实例的`responseType`属性等于`text`或者空字符串，`responseText`属性就会包含已经收到的部分信息。
      - 4，表示服务器返回的数据已经完全接收，或者本次接收已经失败。

    - 然后通过 status 才能判断是否成功（上面的 readyState 只是判断请求是否完毕）

    - 然后有点疑惑的地方就是 open 和 send

    - open方法：用于指定 HTTP 请求的参数，或者说初始化 XMLHttpRequest 实例对象。它一共可以接受五个参数。

      - ```javascript
        void open(
           string method,
           string url,
           optional boolean async,
           optional string user,
           optional string password
        );
        ```

      - `method`：表示 HTTP 动词方法，比如`GET`、`POST`、`PUT`、`DELETE`、`HEAD`等。

      - `url`: 表示请求发送目标 URL。

      - `async`: 布尔值，表示请求是否为异步，默认为`true`。如果设为`false`，则`send()`方法只有等到收到服务器返回了结果，才会进行下一步操作。该参数可选。由于同步 AJAX 请求会造成浏览器失去响应，许多浏览器已经禁止在主线程使用，只允许 Worker 里面使用。所以，这个参数轻易不应该设为`false`。

      - `user`：表示用于认证的用户名，默认为空字符串。该参数可选。

      - `password`：表示用于认证的密码，默认为空字符串。该参数可选。

    - send方法：用于实际发出 HTTP 请求。它的参数是可选的，如果不带参数，就表示 HTTP 请求只有一个 URL，没有数据体，典型例子就是 GET 请求；如果带有参数，就表示除了头信息，还带有包含具体数据的信息体，典型例子就是 POST 请求。

    - 所以呢，open 其实就是建立连接的，并且初始化，而 send 方法就是塞数据的
    
# Day 10

## 第四次的update

这次作者完成了什么？

- 记录了做题的时间

遇到坑的地方：

- 对一些 webpack 的知识点不会

  - 需要学习一波 webpack 知识

- html 的元数据

  - 介绍元数据之前，先引用一个地址 

  - > <https://segmentfault.com/a/1190000004279791>

  - 这里说的很清楚，接下来我总结一下自己的收获

  - meta 的作用：meta常用于定义页面的说明，关键字，最后修改日期，和其它的元数据。这些元数据将服务于浏览器（如何布局或重载页面），搜索引擎和其它网络服务。

  - 观看了大神的讲解之后，只要有两个属性 name,http-equiv

  - name,http-equiv 都是设置参数的名称，参数的值是设置在 content 中的
  
# Day 11（webpack的学习）

## webpack是什么？

- WebPack可以看做是**模块打包机**：它做的事情是，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其转换和打包为合适的格式供浏览器使用。

## webpack能干什么

- 代码转换
- 文件优化
- 代码分割
- 模块合并
- 自动刷新
- 代码校验
- 自动发布

## Javascript模块化

在 webpck 中，识别三种：AMD，CommandJS（只能在服务端运行，即 node.js），ES6 module

- AMD语法

  - ```javascript
    define("module", ["dep1", "dep2"], function(d1, d2) {
      return someExportedValue;
    });
    require(["module", "../file"], function(module, file) { /* ... */ });
    ```

  - 因为 CommonJS 不适用于浏览器环境，所以出现了 AMD 规范。该规范**异步**加载依赖，可以再声明的时候指定需要加载的依赖，并且需要当做参数传入，对于依赖的模块提前执行，依赖前置（即在运行前，就把需要的依赖加载好了）。

- CommandJS

  - ```javascript
    // 导入
    const moduleA = require('./moduleA');
    
    // 导出
    module.exports = moduleA.someFunc;
    ```

  - node.js 采用的就是 CommonJS 规范，使用 `require` 的方法**同步**加载依赖，一个文件就是一个模块

- ES6 module（webpack 推荐）

  - ```javascript
    // 导入
    import { readFile } from 'fs';
    import React from 'react';
    // 导出
    export function hello() {};
    export default {
      // ...
    };
    ```

## CSS设计模式

### CSS采用设计模式，就是为了以后的维护

- OOCSS
  - 减少对 HTML 结构依赖
  - 增加 CSS 的 Class 重复使用
- SMACSS
- Atomic CSS
  - 感觉就是 bootstrap 的思想。把功能分成一个一个的类名，例如，想要红色，就命名一个类，专门负责红色，然后需要的时候，添加这个类名即可
- MCSS
- AMCSS
- BEM（Block Element Modifier）
  - .block { /* styles */ } // 代表块*
  - *.block__element { /* styles */ } // 代表元素*
  - *.block--modifier { /* styles */ } // 代表修饰语
  - BEM的思想是在命名类名的时候，根据是块（Block），还是块的元素（子元素，Eleme），还是描述块的状态（Modifier），来决定命名规则。
  - 这样的话，以后维护的时候，能更快找到相对于的元素，以及他的含义

## 打包自己第一个wepkack程序

- 使用 npm install yarn -g 命名安装 yarn

- 新建一个文件夹(webpack-demo)

- yarn add webpack webpack-cli -D（在 webpack 4.0 后，需要安装 webpack-cli，-D的意思是上线不打包 webpack 以及 webpack-cli 模块）

- 进入 webpack-demo 中，新建一个 src 文件夹，然后新建一个 index.js

- ```javascript
  console.log(123)
  ```

- 然后在命令行中输入 npx webpack

- 然后会多出一个 dist 文件夹，里面的 main.js 就是打包好后的文件

## 样式处理（1）

- 问题：我们在 src 文件夹中的 html 中直接引用 css 文件，是不生效的。因为当打包后，在 dist 文件夹中并没有相应的 css 文件。所以，得引入插件

- yarn add css css-loader -D

- 然后在 webpack.config.js 文件中

- ```javascript
  module : {
  	rules:[//解析顺序是从右往左
  		{
  			test: /\.css$/
  			use : [{
  				loader: 'style-loader' //style-loader是把 css 插入到 head 标签中
  			},'css-loader'//css-loader 是解析 @import 这种语法的]
  		}
  	]
  }
  ```

- ```
  其中,use:[这里可以是数组，还可以是对象]
  ```

- 上述代码还会出现一些现象，就是 style-loader 会把样式添加到 head 标签的最下面（即优先级最高），那如果想把自己写的样式放到 head 标签的最下面，该怎么办？

- ```
  在 use : [使用对象]，其中，对象中有一个 insertAt : 'top' 属性，设置即可
  ```

- 上面演示了如何加载 css-loader，其实其他的，例如 sass，less 也是一样的


## 样式处理（2）

- 问题：上面将的是在 head 标签中添加 style，但是，能不能打成一个文件呢？使用 link 来引入呢？答案是可以的

- 添加插件 yarn add mini-css-extract-plugin -D

- 然后 index.js 中依旧是

- ```javascript
  require('./index.css')
  ```

- 在 webpack.config.js 中

- ```
  const MinCssExtractPlugin = require('mini-css-extract-plugin')
  plugins :[
  	new MinCssExtractPlugin({
  		filename : 'main.css'//打包出来的css文件名
  	})
  ]
  然后在
  module : {
  	rules: [
              {
                  test: /\.css$/,
                  use: [MinCssExtractPlugin.loader, 'css-loader']//把style-loader替换成MinCssExtractPlugin.loader
              }
          ]
  }
  ```

- 还有一个问题，当我们使用css3的时候，希望加上浏览器的前缀

- ```javascript
  添加两个插件
  yarn add postcss-loader autoprefixer -D
  然后
  module : {
  	rules: [
              {
                  test: /\.css$/,
                  use: [MinCssExtractPlugin.loader, 'css-loader'，'postcss-loader']//把style-loader替换成MinCssExtractPlugin.loader
              }
          ]
  }
  这样还不行，因为postcss-loader需要一个配置文件，叫postcss.config.js
  在配置文件中填写：
  module.exports = {
    plugins : [
        require('autoprefixer')(['last 2 versions','>0.01%'])
    ]
  };
  ```

## 压缩css与js文件

- ```javascript
  yarn add terser-webpack-plugin optimize-css-assets-webpack-plugin -D
  然后
  const OptimizeCss = require('optimize-css-assets-webpack-plugin');
  const TerserWebpackPlugin = require('terser-webpack-plugin');
  
  module.exports = {
      optimization: {
          minimizer: [
              new OptimizeCss(),//压缩css的
              new TerserWebpackPlugin({//压缩js的
                  cache:true,//是否缓冲
                  parallel: true,//并发打包
                  sourceMap:true//源文件与打包出来的文件相映射
              })
          ]
      },
      ...
  };
  
  ```

  # Day 12

## 转化为 es5语法

- 需要三个loader
  - babel-loader
  - @babel/preset-env
  - @babel/core

- ```javascript
  let path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const MiniCssExtractPlugin = require('mini-css-extract-plugin');
  
  module.exports = {
      mode: "development",
      entry: './src/index.js',
      output: {
          filename: "bundle.js",
          path: path.resolve(__dirname, 'dist')
      },
      plugins: [
          new HtmlWebpackPlugin({
              template: "./src/index2.html",
              filename: "index2.html"
          }),
          new MiniCssExtractPlugin({
              filename: "main.css"
          })
      ],
      module: {
          rules: [{
              test: /\.css$/,
              use: [
                  MiniCssExtractPlugin.loader,
                  'css-loader', 
                  'postcss-loader'
              ]
          },{
              test:/\.js$/,
              use : [{
                  loader: "babel-loader",
                  options: {
                      presets:['@babel/preset-env'],
                      plugins:['@babel/plugin-proposal-class-properties']
                  }
              }]
          }]
      }
  };
  
  ```

- 学到这里，发现了几个非常重要的网站
- webpack官网：<https://www.webpackjs.com/>
- npm官网（介绍各种插件用法）：<https://www.npmjs.com/>
- babel官网（介绍各种babel用法）：<https://babeljs.io/>

## 语法校验

```javascript
let path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    devServer: {
        contentBase: path.join(__dirname, 'dist'),
        compress: true,
        port: 8080
    },
    mode: "development",
    entry: './src/index.js',
    output: {
        filename: "bundle.js",
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index2.html",
            filename: "index.html"
        }),
        new MiniCssExtractPlugin({
            filename: "main.css"
        })
    ],
    module: {
        rules: [
            {
                test : /\.js/,
                include: path.resolve(__dirname,'src'),
                exclude: /node_modules/,
                use: {
                    loader: "eslint-loader"
                },
                enforce: "pre"
            },
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader', // css-loader 用来解析@import这种语法,
                    'postcss-loader'
                ]
            }, {
                include: path.resolve(__dirname, 'src'),
                exclude: /node_modules/,
                test: /\.js$/,
                use: [{
                    loader: "babel-loader",
                    options: {
                        presets: ['@babel/preset-env'],
                        plugins: [
                            ["@babel/plugin-proposal-decorators", {"legacy": true}],
                            ["@babel/plugin-proposal-class-properties", {"loose": true}],
                            ['@babel/plugin-transform-runtime']
                        ]
                    }
                }]
            }]
    }
};

```



- 语法校验是通过 eslint 工具实现的，官网：<https://eslint.org/>（在官网可以找到 demo，并且下载配置文件）

## 全局变量的引入

- 要使用到 expose-loader
- webpack.ProvidePlugin

```javascript
let path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const webpack = require('webpack');
module.exports = {
    devServer: {
        contentBase: path.join(__dirname, 'dist'),
        compress: true,
        port: 8080
    },
    mode: "development",
    entry: './src/index.js',
    output: {
        filename: "bundle.js",
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index2.html",
            filename: "index.html"
        }),
        new MiniCssExtractPlugin({
            filename: "main.css"
        }),
        new webpack.ProvidePlugin({
            $:'jquery'//这里是把 jquery 暴露给每个模块
        })
    ],
    module: {
        rules: [
            {
                test: require.resolve('jquery'),
                use: [{//这里是把 jquery 暴露给window
                    loader: 'expose-loader',
                    options: 'jQuery'
                }, {
                    loader: 'expose-loader',
                    options: '$'
                }]
            },
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader', // css-loader 用来解析@import这种语法,
                    'postcss-loader'
                ]
            }, {
                include: path.resolve(__dirname, 'src'),
                exclude: /node_modules/,
                test: /\.js$/,
                use: [{
                    loader: "babel-loader",
                    options: {
                        presets: ['@babel/preset-env'],
                        plugins: [
                            ["@babel/plugin-proposal-decorators", {"legacy": true}],
                            ["@babel/plugin-proposal-class-properties", {"loose": true}],
                            ['@babel/plugin-transform-runtime']
                        ]
                    }
                }]
            }]
    }
};
```

## 图片处理

- 用 js 引入图片

  - 需要用到 file-loader

  - 在webpack.config.js添加

    - ```javascript
      rules: [
                  {
                      test: /\.(png|jpg|gif)$/,
                      use: 'file-loader'
                  },
      			...            
      		]
      ```

    - ```javascript
      import logo from './logo.png'
      let img = new Image();
      img.src = logo;
      document.body.appendChild(img);
      ```

- css引入图片

  - css引入图片没那么麻烦，因为css-loader已经帮我们处理好了，直接引用相应的图片就可以了

- html引入图片

  - 添加一个loader -> yarn add html-withimg-loader

  - ```javascript
    {
        test:/\.html$/,
        use:'html-withimg-loader'
    }
    ```

  - 即可

上述的问题，都是因为文件打包后，图片不会自动跟随一起打包，所以会产生图片找不到的情况，但是只要添加一些loader即可。

现在有一个需求，当图片大小不超过某个限制的时候，使用base64编码

- yarn add url-loader -D //这个是base64编码依赖的loader

- ```json
  {
      test: /\.(png|jpg|gif)$/,
      use: [
          {
              loader: "url-loader",
              options: {
                  limit: 1 * 1024,
                  fallback: 'file-loader'
              }
          }
      ]
  }
  ```

  
## 打包文件分类

之前我们打包的文件都是统一在dist文件夹下，可是我想依据文件类型从而把他放到不同的文件夹下呢？

```javascript
module.exports = {
    ...
    ,
    output: {
        filename: "js/bundle.js",//在这里设置js文件放置的文件夹
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index2.html",
            filename: "html/index.html"//在这里设置htmk文件放置的文件夹
        }),
        new MiniCssExtractPlugin({
            filename: "css/main.css"//在这里设置css文件放置的文件夹
        })
    ],
    module: {
        rules: [
            {
                test: /\.html$/,
                use: 'html-withimg-loader'
            },
            {
                test: /\.(png|jpg|gif)$/,
                use: [
                    {
                        loader: "url-loader",
                        options: {
                            limit: 1 * 1024,
                            fallback: 'file-loader',
                            outputPath : 'img/'//在这里设置图片文件放置的文件夹
                        }
                    }
                ]
            },
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader', // css-loader 用来解析@import这种语法,
                    'postcss-loader'
                ]
            }, {
                include: path.resolve(__dirname, 'src'),
                exclude: /node_modules/,
                test: /\.js$/,
                use: [{
                    loader: "babel-loader",
                    options: {
                        presets: ['@babel/preset-env'],
                        plugins: [
                            ["@babel/plugin-proposal-decorators", {"legacy": true}],
                            ["@babel/plugin-proposal-class-properties", {"loose": true}],
                            ['@babel/plugin-transform-runtime']
                        ]
                    }
                }]
            }]
    }
};

```

我们或许还有一个需求，就是从cdn中加载我们的文件，那么

```javascript
module.exports = {
    ...
    ,
    output: {
        filename: "js/bundle.js",//在这里设置js文件放置的文件夹
        path: path.resolve(__dirname, 'dist'),
        publicPath: "https://www.baidu.com"//这里设置cdn的地址，但是，他会把所有文件的引入都设置成cdn的方式，不过我们只想某些文件引用。
    },
};

```

```javascript
module.exports = {
    ...
    ,
    plugins: [
        new MiniCssExtractPlugin({
            filename: "css/main.css",
            publicPath: "https://www.baidu.com"//往需要的地方添加 publicPath 属性即可
        })
    ],
    module: {
        rules: [
            ...
            ,
            {
                test: /\.(png|jpg|gif)$/,
                use: [
                    {
                        loader: "url-loader",
                        options: {
                            limit: 1 * 1024,
                            fallback: 'file-loader',
                            outputPath : 'img/',
                            publicPath ：'cdn地址' 
                        }
                    }
                ]
            },
                ...]
    }
};
```

## 打包多页应用

有时候我们会打包多页应用，那说明是有多个入口，以及多个出口

```javascript
let path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    mode: "development",
    entry: {//多个入口
        home: './src/home.js',
        other: './src/other.js'
    },
    output: {
        //[name]，就是home和other
        filename: "[name].js",
        path: path.resolve(__dirname, 'dist')
    },
    plugins: [
        //多个html页面
        new HtmlWebpackPlugin({
            template: "./src/index.html",
            filename: "index.html",
            chunks: ['other']//这里的意思是我需要引入哪个js模块
        }),
        new HtmlWebpackPlugin({
            template: "./src/index2.html",
            filename: "index2.html",
            chunks: ['home']
        }),
    ]
};

```

## 配置source-map

我们会有一个需求，当我们的js文件经过打包处理之后，如果报错了，是找不到报错的地方的（因为都被压缩在一行里），那该怎么办呢？配置source-map（源码映射）即可

```javascript
let path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    mode: "production",
    entry: './src/home.js',
    output: {
        filename: "index.js",
        path: path.resolve(__dirname, 'dist')
    },
    //源码映射
    devtool: "source-map",//会单独生成一个source-map文件，当出错了，会标识当前报错的列和行
    //devtool: "eval-source-map",//不会单独生成一个source-map文件，当出错了，会标识当前报错的列和行
    // devtool: "cheap-module-source-map",
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index.html",
            filename: "index.html"
        })
    ],
    module: {
        rules: [
            {
                test : '/\.js$/',
                use: {
                    loader: "babel-loader",
                    options: {
                        presets:['@babel/preset-env']
                    }
                }
            }
        ]
    }
};
```

一共有四种devtool模式

- source-map
- eval-source-map
- module-source-map
- module-eval-source-map


## watch用法

watch是干什么用的呢？可以用watch来实时监控我们的文件，当发生变化之后，自动build，而不再需要人工手动的npm run build

```javascript
module.exports = {
    ...
    ,
    watch: true,//开启watch功能
    watchOptions: {//监控的选项
    	aggregateTimeout : 500//类似与输入框的防抖功能。当我们一直修改同一个文件的时候，打包时间会延长，直到不再修改。这样的好处是避免了一些无谓的打包
    	ignored : /node_moudules/,//忽略某些文件不用监控，因为我们一定不会修改某些文件
    	poll : 1000//几秒检查一次
    },
    ...
};
上述代码的时间的单位都是毫秒
```

## 防抖

### 以下文章参考过薄荷前端<https://github.com/BooheeFE/weekly>

#### 解释：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。

直接上代码

```html
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
<input type="text" name="" id="app">
<script type="text/javascript">
	// 以下举了一个输入框做例子
	let app = document.getElementById('app');

	// 模拟用户发起ajax请求
	function ajax(content){
		console.log("request : " + content);
	}

	// 加上事件监听
	app.addEventListener('keyup',function(e){
		debounce(e.target.value);
	});


	// 就是这里防抖
	// 防抖利用了setTimeout方法
	function debounceAjax(func,delay){
		let that = this;
		return function(args){
			// 当在限定的时间内再次触发，就会把上一次的函数的执行去掉
			clearTimeout(func.id);
			// 这里用了一个id,来接受setTimeout返回的参数。（每次调用setTimeout都会返回一个参数）
			func.id = setTimeout(function(){
				// 重新设定需要执行的函数
				func(args);
				// 网上是这样写的
				// func.call(that,args);
			},delay);
		}
	}

	// 这里有个需要注意的地方，用变量debounce接受debounceAjax的返回值，而debounce的返回值是一个函数，然后我们才能在事件监听上传递我们的参数给ajax方法
	let debounce = debounceAjax(ajax,500);
</script>
</body>
</html>
```

所以，防抖，个人感觉就是执行最新的函数。例如，5 s 内如果有事件不断触发，那只会执行最后一次，即触发完最新的函数后，在规定的时间内，没有其他事件触发，就执行这个方法。

## 节流

### 以下文章参考过薄荷前端<https://github.com/BooheeFE/weekly>

#### 解释：在规定的时间内，只会触发第一次的函数，剩下事件的触发，都不会被调用，直到规定的时间结束

直接上代码

```html
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
<input type="text" name="" id="app">
<script type="text/javascript">
	let app = document.getElementById('app');

	app.addEventListener('keyup',function(e){
		throttle(e.target.value);
	});

	// 模拟要进行的操作
	function ajax(content){
		console.log(content);
	}

	// 这就是节流
	function throttleAjax(func,delay){
		// 用来记录用户第一次输入的时间，当一个周期结束后，重新赋值
		let last;
		return function(args){
			// 接受用户输入的时间，用来判断是否在一个周期内
			let now = Date.now();
			if(last && now < last + delay){// 当用户输入的时间距离第一次输入的时间还在一个周期内，就说什么也不干

			}else{
				// 当超过周期了，就把最新的时间重新赋值
				last = now;
				// 然后调起函数
				func(args);
				// 网上是这样写的
				// func.apply(上下文,arguments);
			}	
		}
	}

	function throttleAjax2(func,delay){
		let last;
		let now = Date.now();
		if(last && now < last + delay){
			
		}else{
			last = now;
			return function(args){
				func(args);
			}
		}
	}

	let throttle = throttleAjax(ajax,1000);

</script>
</body>
</html>
```

节流就是在一个时间周期内，只会执行用户第一次的输入，并且在这个周期内，其他输入不给予理会

## 研究一段代码（来自节流的疑惑）

```html
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
<input type="text" name="" id="app">
<script type="text/javascript">
	let app = document.getElementById('app');
	app.addEventListener('keyup',function(e){
		throttle(e.target.value);
	});
	function ajax(content){
		console.log(content);
	}
	function throttleAjax(func,delay){
		let last;
		return function(args){
			let now = Date.now();
			if(last && now < last + delay){

			}else{
				last = now;
				func(args);
			}	
		}
	}
	function throttleAjax2(func,delay){
		let last;
		let now = Date.now();
		if(last && now < last + delay){
			
		}else{
			last = now;
			return function(args){
				func(args);
			}
		}
	}
	let throttle = throttleAjax(ajax,1000);
</script>
</body>
</html>
```

#### throttleAjax是观看网上的代码后写的

#### throttleAjax2是自己写的

#### 他们有什么区别呢？

#### 我们看调用

#### let throttle = throttleAjax(参数，参数) | let throttle = throttleAjax2(参数，参数)

#### 在第一个函数中，我返回的是一个函数的引用，逻辑的判断都在返回的函数里面。但我们循环调用的时候，因为接收的是函数的引用，并且判断的逻辑也在函数里。所以，每次循环都会进行判断。

#### 而第二个函数，是先经过逻辑判断，最后才返回一个函数的引用。但是，放到一个循环中，仅仅只是函数的引用，里面们没有判断的逻辑了，导致不能判断！！！

