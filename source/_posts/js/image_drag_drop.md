---
title: 图片拖拽上传
date: 2017-08-29 17:46:25
updated: 2017-11-25 09:40:00
categories: 编程
tags: [JS,HTML,前端]
---
这个东西一直拖着，没有详细了解过，现在动手试一下。
<!--more-->
实现一个图片拖拽进入页面后显示在页面上的功能。

代码很简单，就不详细解释了，需要注意的地方就是要记得preventDefault。

然后分别用了window.URL和FileReader实现显示的效果。
```htmlbars
<!DOCTYPE html>
<html>
<head>
	<title>拖拽</title>
	<style type="text/css">
		#container{
			height: 200px;
			line-height: 200px;
			border:1px dashed #333;
			text-align: center;
			font-size: 20px;
			color: #999;
		}
		#result{
			height: 200px;
			width: 80%;
			margin: 50px auto;
			border:1px dotted #333;
		}
	</style>
</head>
<body>
<div id="container">将图片拖到这里</div>
<div id="result"></div>
</body>
<script>
	function init(){
		document.ondragenter = function(e){
			e.preventDefault();
		}
		document.ondragover =function(e){
			e.preventDefault();
		}
		document.ondrop = function(e){
			e.preventDefault();
		}
		/*document.ondragleave = function(e){
			e.preventDefault();
		}*/
	}
	init();
	var container = document.querySelector('#container');
	var result = document.querySelector("#result");
	function show1(file){
		window.URL = window.URL || window.webkitURL;
		var image = window.URL.createObjectURL(file);
		var name = file.name;
		var size = Math.floor(file.size/1024);
		var str = "<img src='"+image+"'><p>图片名称："+name+"</p><p>大小："+size+"KB</p>";
		result.innerHTML = str;
	}
	function show2(file){
		var reader = new FileReader();
		reader.readAsDataURL(file);
		var name = file.name;
		var size = Math.floor(file.size/1024);
		reader.onload = function(e){
        	result.innerHTML = '<img src="'+this.result+'" alt=""/><p>图片名称：'+name+'</p><p>大小：'+size+'KB</p>'
    	}
	}
	function show(file){
		show2(file);
	}

	container.addEventListener('drop',function(e){
		e.preventDefault();
		var fileList = e.dataTransfer.files;
		if(fileList.length == 0 ){
			return false;
		}
		if(fileList[0].type.indexOf('image') === -1){
			return false;
		}
		show(fileList[0])

	})
</script>
</html>
```