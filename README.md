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
所以箭头函数使用call的话，第一个参数会给忽略掉的
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


