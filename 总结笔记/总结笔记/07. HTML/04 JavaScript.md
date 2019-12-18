## 一、JavaScript

JavaScript包含三个部分：

1. ECMAscript JavaScript的语法（变量、函数、循环语句等语法）
2. DOM文档对象模型：操作html和CSS的方法；
3. BOM浏览器对象模型：操作浏览器的一些方法。

### 1. 嵌入页面的而方式

1. 行间事件（主要用于事件）

   ```html
   <input type="button" name="" onclick="alert('ok！');">
   ```

2. 页面script标签嵌入

   ```html
   <script type="text/javascript">        
       alert('ok！');
   </script>
   ```

3. 外部引入

   ```html
   <script type="text/javascript" src="js/index.js"></script>
   ```

### 2. 基础语法

#### 2.1 变量

JavaScript 是一种弱类型语言，javascript的变量类型由它的值来决定。 定义变量需要用关键字 'var'

```javascript
 var iNum = 123;
 var sTr = 'asd';

 //同时定义多个变量可以用","隔开，公用一个‘var’关键字

 var iNum = 45,sTr='qwe',sCount='68';
```

##### 2.1.1 变量类型

有5种基本数据类型：

1. number 数字类型
2. string 字符串类型
3. boolean 布尔类型 true 或 false
4. undefined undefined类型，变量声明未初始化，它的值就是undefined
5. null null类型，表示空对象，如果定义的变量将来准备保存对象，可以将变量初始化为null,在页面上获取不到对象，返回的值就是null

1种复合类型：

1. object

#### 2.2 获取元素对象

##### 2.2.1 `getElementById`

可以使用内置对象document上的`getElementById()`方法来获取页面上设置了id属性的元素，获取到的是一个html对象，然后将它赋值给一个变量，比如：

```html
<script type="text/javascript">
    var oDiv = document.getElementById('div1');
</script>
....
<div id="div1">这是一个div元素</div>
```

上面的语句，如果把javascript写在元素的上面，就会出错，因为页面上从上往下加载执行的，javascript去页面上获取元素div1的时候，元素div1还没有加载，解决方法有两种：

1. 将javascript放到页面最下边

   ```html
   ....
   <div id="div1">这是一个div元素</div>
   ....
   
   <script type="text/javascript">
       var oDiv = document.getElementById('div1');
   </script>
   </body>
   ```

2. 将javascript语句放到window.onload触发的函数里面,获取元素的语句会在页面加载完后才执行，就不会出错了

   ```html
   <script type="text/javascript">
       window.onload = function(){
           var oDiv = document.getElementById('div1');
       }
   </script>
   
   ....
   
   <div id="div1">这是一个div元素</div>
   ```

##### 2.2.2 `getElementsByTagName`

可以使用内置对象document上的`getElementsByTagName`方法来获取页面上的某一种标签，获取的是一个选择集，不是数组，但是可以用下标的方式操作选择集里面的标签元素。

```html
<script type="text/javascript">
    window.onload = function(){
        var aLi = document.getElementsByTagName('li');
        // aLi.style.backgroundColor = 'gold'; // 出错！不能同时设置多个li
        alert(aLi.length);
        aLi[0].style.backgroundColor = 'gold';
    }
</script>
....
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
    <li>6</li>
</ul>
```

#### 2.3 操作元素对象

获取的页面元素，就可以对页面元素的属性进行操作，属性的操作包括属性的读和写。

##### 2.3.1 操作属性的方法

1. `.`操作
2. `[ ]`操作

Html属性写法：

1. html的属性和js里面属性写法一样
2. “class” 属性写成 “className”
3. “style” 属性里面的属性，有横杠的改成驼峰式，比如：“font-size”，改成”style.fontSize”

##### 2.3.2 通过“.”操作属性

```html
<script type="text/javascript">

    window.onload = function(){
        var oInput = document.getElementById('input1');
        var oA = document.getElementById('link1');
        // 读取属性值
        var sValue = oInput.value;
        var sType = oInput.type;
        var sName = oInput.name;
        var sLinks = oA.href;
        // 写(设置)属性
        oA.style.color = 'red';
        oA.style.fontSize = sValue;
    }

</script>

<input type="text" name="setsize" id="input1" value="20px">
<a href="http://www.itcast.cn" id="link1">传智播客</a>
```

##### 2.3.3 通过“[ ]”操作属性

```html
<script type="text/javascript">

    window.onload = function(){
        var oInput1 = document.getElementById('input1');
        var oInput2 = document.getElementById('input2');
        var oA = document.getElementById('link1');
        // 读取属性
        var sVal1 = oInput1.value;
        var sVal2 = oInput2.value;
        // 写(设置)属性
        // oA.style.val1 = val2; 没反应
        oA.style[sVal1] = sVal2;        
    }

</script>

......

<input type="text" name="setattr" id="input1" value="fontSize">
<input type="text" name="setnum" id="input2" value="30px">
<a href="http://www.itcast.cn" id="link1">传智播客</a>
```

##### 2.3.4 innerHTML

innerHTML可以读取或者写入标签包裹的内容

```html
<script type="text/javascript">
    window.onload = function(){
        var oDiv = document.getElementById('div1');
        //读取
        var sTxt = oDiv.innerHTML;
        alert(sTxt);
        //写入
        oDiv.innerHTML = '<a href="http://www.itcast.cn">传智播客<a/>';
    }
</script>

<div id="div1">这是一个div元素</div>
```

#### 2.4 函数

##### 2.4.1 函数定义与执行

```html
<script type="text/javascript">
    // 函数定义
    function fnAlert(){
        alert('hello!');
    }
    // 函数执行
    fnAlert();
</script>
```

##### 2.4.2 变量与函数预解析

JavaScript解析过程分为两个阶段，先是编译阶段，然后执行阶段，在编译阶段会将function定义的函数提前，并且将var定义的变量声明提前，将它赋值为undefined。

```html
<script type="text/javascript">    
    fnAlert();       // 弹出 hello！
    alert(iNum);  // 弹出 undefined
    function fnAlert(){
        alert('hello!');
    }
    var iNum = 123;
</script>
```

##### 2.4.3 匿名函数

定义的函数可以不给名称，这个叫做匿名函数，可以将匿名函数直接赋值给元素绑定的事件来完成匿名函数的调用。

```html
<script type="text/javascript">
    window.onload = function(){
        var oBtn = document.getElementById('btn1');
        // 直接将匿名函数赋值给绑定的事件
        oBtn.onclick = function (){
            alert('ok!');
        }
    }
</script>
```

##### 2.4.4 函数传参

```html
<script type="text/javascript">
    function fnAlert(a){
        alert(a);
    }
    fnAlert(12345);
</script>
```

##### 2.4.5 return关键字

函数中return关键字的作用：

1. 返回函数执行的结果
2. 结束函数的运行
3. 阻止默认行为

```html
<script type="text/javascript">
    function fnAdd(iNum01,iNum02){
        var iRs = iNum01 + iNum02;
        return iRs;
        alert('here!');
    }

    var iCount = fnAdd(3,4);
    alert(iCount);  //弹出7
</script>
```

#### 2.5 条件语句

##### 2.5.1 if else

```javascript
var iNum01 = 3;
var iNum02 = 5;
var sTr;
if(iNum01>iNum02) {
    sTr = '大于';
}
else {
    sTr = '小于';
}
alert(sTr);
```

##### 2.5.2 多重if else语句

```javascript
var iNow = 1;
if(iNow==1) {
    ... ;
}
else if(iNow==2) {
    ... ;
}
else {
    ... ;
}
```

##### 2.5.3 switch语句

多重if else语句可以换成性能更高的switch语句

```javascript
var iNow = 1;

switch (iNow){
    case 1:
        ...;
        break;
    case 2:
        ...;
        break;    
    default:
        ...;
}
```

#### 2.6 数组及操作方法

数组就是一组数据的集合，javascript中，数组里面的数据可以是不同类型的。

##### 2.6.1 定义数组的方法

```javascript
//对象的实例创建
var aList = new Array(1,2,3);

//直接量创建
var aList2 = [1,2,3,'asd'];
```

##### 2.6.2 操作数组中数据的方法

1. 获取数组的长度：`aList.length;`

   ```javascript
   var aList = [1,2,3,4];
   alert(aList.length); // 弹出4
   ```

2. 用下标操作数组的某个数据：`aList[0];`

   ```javascript
   var aList = [1,2,3,4];
   alert(aList[0]); // 弹出1
   ```

3. `join()`将数组成员通过一个分隔符合并成字符串

   ```javascript
   var aList = [1,2,3,4];
   alert(aList.join('-')); // 弹出 1-2-3-4
   ```

4. `push()` 和 `pop() `从数组最后增加成员或删除成员

   ```javascript
   var aList = [1,2,3,4];
   aList.push(5);
   alert(aList); //弹出1,2,3,4,5
   aList.pop();
   alert(aList); // 弹出1,2,3,4
   ```

5. `unshift()`和 `shift()` 从数组前面增加成员或删除成员

   ```javascript
   var aList = [1,2,3,4];
   aList.unshift(5);
   alert(aList); //弹出5,1,2,3,4
   aList.shift();
   alert(aList); // 弹出1,2,3,4
   ```

6. `reverse()` 将数组反转

   ```javascript
   var aList = [1,2,3,4];
   aList.reverse();
   alert(aList);  // 弹出4,3,2,1
   ```

7. `indexOf()` 返回数组中元素第一次出现的索引值

   ```javascript
   var aList = [1,2,3,4,1,3,4];
   alert(aList.indexOf(1));
   ```

8. `splice()` 在数组中增加或删除成员

   ```javascript
   var aList = [1,2,3,4];
   aList.splice(2,1,7,8,9); //从第2个元素开始，删除1个元素，然后在此位置增加'7,8,9'三个元素
   alert(aList); //弹出 1,2,7,8,9,4
   ```

##### 2.6.3 多维数组 

多维数组指的是数组的成员也是数组的数组。

```
var aList = [[1,2,3],['a','b','c']];

alert(aList[0][1]); //弹出2;
```

#### 2.7 循环语句

##### 2.7.1 for循环

```javascript
for(var i=0;i<len;i++) {
    ......
}
```

##### 2.7.2 while循环

```javascript
var i=0;

while(i<8) {
    ......
    i++;
}
```

#### 2.8 字符串处理方法

##### 2.8.1 字符串合并操作`+ `

```javascript
var iNum01 = 12;
var iNum02 = 24;
var sNum03 = '12';
var sTr = 'abc';
alert(iNum01+iNum02);  //弹出36
alert(iNum01+sNum03);  //弹出1212 数字和字符串相加等同于字符串相加
alert(sNum03+sTr);     // 弹出12abc
```

##### 2.8.2 `parseInt()`

将数字字符串转化为整数

```javascript
var sNum01 = '12';
var sNum02 = '24';
var sNum03 = '12.32';
alert(sNum01+sNum02);  //弹出1224
alert(parseInt(sNum01)+parseInt(sNum02))  //弹出36
alert(sNum03)   //弹出数字12 将字符串小数转化为数字整数
```

##### 2.8.3 `parseFloat()`

将数字字符串转化为小数

```javascript
var sNum03 = '12.32'
alert(parseFloat(sNum03));  //弹出 12.32 将字符串小数转化为数字小数
```

##### 2.8.4 `split()`

把一个字符串分隔成字符串组成的数组

```javascript
var sTr = '2017-4-22';
var aRr = sTr.split("-");
var aRr2= sTr.split("");

alert(aRr);  //弹出['2017','4','2']
alert(aRr2);  //弹出['2','0','1','7','-','4','-','2','2']
```

##### 2.8.5 `charAt()`

获取字符串中的某一个字符

```javascript
var sId = "#div1";
var sTr = sId.charAt(0);
alert(sTr); //弹出 #
```

##### 2.8.6 `indexOf()`

查找字符串是否含有某字符

```javascript
var sTr = "abcdefgh";
var iNum = sTr.indexOf("c");
alert(iNum); //弹出2
```

2.8.7 `substring()`

截取字符串 用法： substring(start,end)（不包括end）

```javascript
var sTr = "abcdefghijkl";
var sTr2 = sTr.substring(3,5);
var sTr3 = sTr.substring(1);

alert(sTr2); //弹出 de
alert(sTr3); //弹出 bcdefghijkl
```

##### 2.8.8 `toUpperCase()`

 字符串转大写

```javascript
var sTr = "abcdef";
var sTr2 = sTr.toUpperCase();
alert(sTr2); //弹出ABCDEF
```

##### 2.8.9 `toLowerCase()`

字符串转小写

```javascript
var sTr = "ABCDEF";
var sTr2 = sTr.toLowerCase();
alert(sTr2); //弹出abcdef
```

##### 2.8.10 字符串反转

```javascript
var str = 'asdfj12jlsdkf098';
var str2 = str.split('').reverse().join('');

alert(str2);
```

#### 2.9 类型转换

##### 2.9.1 直接转换

`parseInt()`与`parseFloat()`

```javascript
alert('12'+7); //弹出127
alert( parseInt('12') + 7 );  //弹出19 
alert( parseInt(5.6));  // 弹出5
alert('5.6'+2.3);  // 弹出5.62.3
alert(parseFloat('5.6')+2.3);  // 弹出7.8999999999999995
alert(0.1+0.2); //弹出 0.3000000000000004
alert((0.1*100+0.2*100)/100); //弹出0.3
alert((parseFloat('5.6')*100+2.3*100)/100); //弹出7.9
```

##### 2.9.2 隐式转换"=="和 "-"

```javascript
if('3'==3) {
    alert('相等');
}

// 弹出'相等'
alert('10'-3);  // 弹出7
```

##### 2.9.3 NaN 和 isNaN

```javascript
alert( parseInt('123abc') );  // 弹出123
alert( parseInt('abc123') );  // 弹出NaN
```

#### 2.10 定时器

定时器在javascript中的作用：

1. 制作动画
2. 异步操作
3. 函数缓冲与节流

定时器类型及语法

```javascript
/*
    定时器：
    setTimeout  只执行一次的定时器 
    clearTimeout 关闭只执行一次的定时器
    setInterval  反复执行的定时器
    clearInterval 关闭反复执行的定时器

*/

var time1 = setTimeout(myalert,2000);
var time2 = setInterval(myalert,2000);
/*
clearTimeout(time1);
clearInterval(time2);
*/
function myalert(){
    alert('ok!');
}
```

#### 2.11 变量作用域

变量作用域指的是变量的作用范围，javascript中的变量分为全局变量和局部变量。

1. 全局变量：在函数之外定义的变量，为整个页面公用，函数内部外部都可以访问。
2. 局部变量：在函数内部定义的变量，只能在定义该变量的函数内部访问，外部无法访问。

```javascript
<script type="text/javascript">
    //全局变量
    var a = 12;
    function myalert() {
        //局部变量
        var b = 23;
        alert(a);
        alert(b);
    }
    myalert(); //弹出12和23
    alert(a);  //弹出12    
    alert(b);  //出错
</script>
```

#### 2.12 封闭函数

封闭函数是javascript中匿名函数的另外一种写法，创建一个一开始就执行而不用命名的函数。

一般定义的函数和执行函数：

```javascript
function myalert(){
    alert('hello!');
};

myalert();
```

封闭函数：

```javascript
(function myalert(){
    alert('hello!');
})();
```

还可以在函数定义前加上“~”和“!”等符号来定义匿名函数

```javascript
!function myalert(){
    alert('hello!');
}()
```

##### 2.12.1 封闭函数的好处

封闭函数可以创造一个独立的空间，在封闭函数内定义的变量和函数不会影响外部同名的函数和变量，可以避免命名冲突，在页面上引入多个js文件时，用这种方式添加js文件比较安全，比如：

```javascript
var iNum01 = 12;
function myalert(){
    alert('hello!');
}
(function(){
    var iNum01 = 24;
    function myalert(){
        alert('hello!world');
    }
    alert(iNum01);
    myalert()
})()
alert(iNum01);
myalert();
```

#### 2.13 常用内置对象

##### 2.13.1 document

```javascript
document.getElementById //通过id获取元素
document.getElementsByTagName //通过标签名获取元素
document.referrer  //获取上一个跳转页面的地址(需要服务器环境)
```

##### 2.13.2 location

```javascript
window.location.href  //获取或者重定url地址
window.location.search //获取地址参数部分
window.location.hash //获取页面锚点或者叫哈希值
```

##### 2.13.3 Math

```javascript
Math.random 获取0-1的随机数
Math.floor 向下取整
Math.ceil 向上取整
```

### 5. 调试程序的方法

1. alert
2. console.log
3. document.title

