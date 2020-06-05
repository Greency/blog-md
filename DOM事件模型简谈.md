#### 1. 事件模型
##### 1.1 事件传播图示
> 理解了这张图，以后有关事件模型的问题基本都可以解决！

![image](https://greency.github.io/images-site/youdaoyunbiji/7_event.png)

##### 1.2 事件模型中的三个阶段（按顺序执行）
```
1. 捕获阶段
2. 目标阶段
3. 冒泡阶段
```
#### 2. 事件注册方式
```
//1
element.addEventListener('click',  function(e){
    e  //event对象
}, false);

//2
element.onclick = function(e){ e  //event对象 };

//3
<div onclick="handleClick(this)"></div>
function handleClick(e){ e  //element：<div onclick="handleClick(this)"></div> }
```
#### 3. addEventListener(eventName, callback, options, useCapture)
> options，useCapture是可选项

3.1 当options.capture或者useCapture为true时，代表此事件会在“捕获阶段”就被触发。
> 何为“捕获阶段触发”？指的是：在事件传播到“目标阶段”后，就会先执行“捕获阶段”的事件回调，然后再执行“目标阶段”的事件回调，最后执行“冒泡阶段”的事件回调。
```
<div class="par par1">
    <div class="par par2">
        <div class="par par3"></div>
    </div>
</div>

document.querySelector('.par1').addEventListener('click', function(e){
    console.log('par1'); 
});

document.querySelector('.par2').addEventListener('click', function(e){
    console.log('par2'); 
}, true);

document.querySelector('.par3').addEventListener('click', function(e){
    console.log('par3'); 
});

//测试一：点击class为par3的时候，输出如下：
par2  //useCapture为true，所以会在事件传播到“目标阶段”时，就执行“捕获阶段”的事件回调。
par3  //执行“目标阶段”的事件回调
par1  //最后执行“冒泡阶段”的事件回调
```

#### 4. Event对象
##### 4.1 preventDefault
> 作用：阻止事件本身默认的行为，但是这个事件依然还是会继续传播，并且对应的事件回调依然会执行

###### 4.1.1 何为默认行为
1. 复选框选中后，默认有一个"勾号"，且checked会变为true；
2. 按钮点击后，默认会刷新页面

> 举例说明：
```
<button onclick="test()">获取checked</button>
<input type="checkbox" id="id-checkbox" />

document.querySelector("#id-checkbox").addEventListener("click", function(event) {
    console.log('checked');
    event.preventDefault();  //阻止默认行为
});

function test(){
    console.log('isChecked: ', document.querySelector('#id-checkbox').checked);
}

//当点击“复选框”时：
①“勾号”未出现，console输出"checked"
②点击“获取checked”按钮时，获取的复选框checked永远false

//当注释掉“event.preventDefault”时：
①勾号出现，console输出"checked"
②点击“获取checked”按钮时，获取的复选框checked为true

//注意，从上面两种情况可以看出，事件绑定的回调函数都会被执行。
```

###### 4.2 stopPropagation
> 作用：阻止（捕获/冒泡）事件的继续传播

> 我们知道，事件的传播有三个阶段：捕获阶段=》目标阶段=>冒泡阶段；当在某一阶段使用stopPropagation时，后续的事件/阶段将不再触发。举例说明：
```
<div class="par par1">
    <div class="par par2">
        <div class="par par3"></div>
    </div>
</div>

document.querySelector('.par1').addEventListener('click', function(e){
    console.log('par1'); 
});

document.querySelector('.par2').addEventListener('click', function(e){
    console.log('par2'); 
}, true);

document.querySelector('.par3').addEventListener('click', function(e){
    e.stopPropagation();
    console.log('par3'); 
});

//点击class为par3的元素时，输出如下：
par2  //捕获阶段触发
par3  //到达目标阶段，
      //但是由于这里调用了stopPropagation阻止了事件的传播，
      //所以处于冒泡阶段的par1并未被触发。
```

#### 完毕
> 以上就是有关事件模型的简单解析，也是日常使用中经常接触的部分；觉得有收获的话，请点个赞！thx~