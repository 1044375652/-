# -Day 01(阅读ES6)
## let/const与var

let/const与var不同的是，let/const会创建一个块级作用域（通俗一点就是一个花括号）。在if/for中，var会存在变量提升的情况，但let/const不会。

举例

for(var i = 0 ; i < 10 ; i++){

​	//TODOS

}

 console.log(i)//10

---------------------------

for(let i = 0 ; i < 10 ; i++){

​	//TODOS

}

 console.log(i)//Uncaught ReferenceError: i is not defined

出现这情况是因为var在循环中，只会声明一次变量，经过10次循环，i的值会叠加到10，

而let在每次循环中，都会重新声明一次，并且存在块级作用域中，所以才访问不到

