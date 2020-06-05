- #### 前言
> 我们先看这样一段代码
```
a();  //报错：a is not function
b();  //正常执行

var a = function(){};  //声明a变量，并将函数赋值给变量，简称赋值式函数
function b(){};  //声明式函数
```

- #### js代码执行顺序
> js分为两个主要的阶段：编译期和执行期。

> 以一个<script></script>为单位划分代码块。

1. 读取一个代码块的js代码
2. 语法分析（括号匹配等），有语法错误则重新进入步骤1。
3. 代码块间的编译期：对声明的变量和函数进行处理；声明的变量不会被赋值（也就是说undefined）。
4. 在代码块中按照顺序执行代码；如果代码错误，此代码块中后续代码将不再执行，然后立即进入步骤1。
5. 没有了代码块，执行完成。

- #### 具体执行
```
<script>
        //代码块1
        b1();
        a1();
        var a1 = function(){
            console.log('this is a1');
        };
        function b1(){
            console.log('this is b1');
        }
    </script>
    <script>
        //代码块2
        b2();
        var a2 = function(){
            console.log('thi is a2');
        }
        function b2(){
            console.log('this is b2')
        }
    </script>
    
    //按照执行js的执行顺序来模拟一遍
    1.读入代码块1
    2.语法分析没问题
    3.编译代码，声明变量a1(此时值为undefined)和函数b1, 没有问题
    4.执行代码，执行b1()，由于已经被声明，调用是没有问题。执行a1()，由于a1的值为undefined，所以报错。
    5.读入代码块2；语法分析没问题；
    编译代码：声明变量a2(此时值为undefined)和函数b2，没有问题；
    执行代码：执行b2()，由于已经被声明，调用是没有问题。
    
    //综上所述，上述代码输出如下：
    this is b1
    报错：a1 is not a function
    this is b2
```