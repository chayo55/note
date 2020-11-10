1. js基本语法
	- 定义变量
		- var ：定义全局变量
		- let ： 定义块级作用域的变量
		- const：定义常量
	- es6新语法
		- 模板字符串``
		```javascript
		console.log(`计算结果为：${a+b}`);
		```
	- js中遍历的几种方式
		```javascript
		let array = [10,20,,,30];
		let jsonObj = {"id":1,"name":"john","birthday":"1900/01/01"};
		```
		- for循环
		即可遍历数组，也可遍历对象
		```
		for(var i = 0; i < array.length; i++){
			console.log(array[i]);//10,20,undefined,undefined,30
		}
		
		```
		- forEach()
		不能遍历对象,遍历数组会跳过空值,不能中断循环`break`
		```javascript
		array.forEach(i=>{
			console.log(i);//10,20,30
			});
		```
		- for  of
		不能遍历对象
		```javascript
		for(var i of array){
			console.log(i);//10,20,undefined,undefined,30
		}
		```
		- for  in
		不建议遍历数组，会跳过空值。
		遍历对象时，只能得到key
		```javascript
		//遍历数组
		for(var i in array){
			console.log(i);//0,1,4
		}
		//遍历对象
		for(var i in jsonObj){
			console.log(i);//id name birthday
		}
		```
		
	- js中的对象
		```JavaScript
		function student(id, name){
			this.id = id;
			this.name = name;
			this.play = function(){console.log("this is play function");}
		}//每个新建对象都会创建一个play函数，浪费资源

		//如果像属性一样赋值操作，内部函数又不能共用，
		function student(id, name){
			this.id = id;
			this.name = name;
		}
		var s1 = student(1,"john");
		var s2 = student(2,"jack");
		s1.play = function(){console.log("this is play function");}
		s2.play();//s2.play is not a function 

		//共用函数可以利用每个js对象内的Prototype对象
		function student(id, name){
			this.id = id;
			this.name = name;
		}
		student.prototype.play=function(){console.log("this is play function");}

		//js中的重写
		function student(id, name){
			this.id = id;
			this.name = name;
		}
		student.prototype.play=function(){console.log("this is play function");}
		var s1 = student(1,"john");
		var s2 = student(2,"jack");
		s1.play=function(){console.log("nothing happen");}
		s1.play();//nothing happen
		```
	- js中的函数
	- js中的闭包
	   目的是保护变量不被污染又能重用
	   (有点类似 Java 中的 单例模式)
	   ```Javascript
		function bank(){
			var totalMoney = 100;
			return function (money){
				totalMoney -= money;
				console.log("取走："+money+"元，余额："+totalMoney+"元");
				return i;
			}
		}

		var doPay = Bank();
		doPay(10);//取走10元，余额90元
	   ```
2. Ajax
	- 原生js实现Ajax
		```javascript
		function doAjaxGet(){
			var xhr = new XMLHttpRequest();
			xhr.onreadystatechange = function(){
				if(xhr.readyState == 4 && xhr.status == 200){
					console.log(xhr.responseText);
				}
			}
			xhr.open("GET","/doAjaxGet",true);
			xhr.send();
		}
		```
	- JQuery
		- $.ajax
		```JavaScript
		$.ajax({
			 type:"POST",//可以省略，默认为get请求
			 url:url,
			 data:params,//可以省略(无需向服务端传递参数)
			 dataType:"text",//可以省略(由ajax函数内部基于返回值进行匹配处理)
			 async:true,//可以省略，默认为true，表示异步
			 success:function(result){//处理服务端返回的正常信息
				 $("#resultId").html(result); 
			 },
			 error:function(xhr){//处理服务端返回的异常信息
				 console.log(xhr.statusText);
				 $("#resultId").html(xhr.statusText); 
			 }
		 })
		```
		- $.get
		```JavaScript
		$.get(url, params, function(result){
			console.log(result);
		});
		```
		- $.getJSON
		- $.post
		- $.load
		- $.serialize()
		```
		let params = $("#FormId").serialize();
		```
	
	- 关于前后端数据传递，FormData与RequestPayload<br>简单来说，参数为对象时，[[HTTP#请求体|请求体]]内为RequestPayload；参数为字符串时，为FormData；但在最后提交请求时，RequestPayload会对非字符串做字符串转换
		- 前端提交的值
			- 传统\<form>表单提交时，```Content-Type=application/x-www-form-urlencoded```
			参数以 ```key1=value1&key2=value2``` 的形式体现
			- 传统ajax提交时，```Content-Type=text/plain```
			参数以 ```[object,object]``` 对象的形式，但提交时还是会做字符串处理
			- 在axios中，如果参数以字符串形式，```Content-Type自动转为 xxx-form-xxx```<br>如果参数以对象，最后实际也会以json字符串形式体现
			- **总之**，<br>
				1. 都只支持字符串形式提交给后端
				2. Form需要传递的格式为```key1=value1&key2=value2``` 类似**GET**请求的**queryString**形式
				3. 非Form一般为```Json.stringify(formDataObject)```形式
		- 后端如何取到值？
		如果以Json形式，后端需要解析Json格式，如果以字符串形式传递，后端需要分割字符串
		在[[SpringBoot#springMVC|springMVC]]中，如果参数为Object(实际为Json字符串传递)，```@RequestBody``` 注解可以自动解析为对应 对象。