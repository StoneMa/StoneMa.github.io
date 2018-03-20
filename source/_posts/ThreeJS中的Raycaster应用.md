---
title: ThreeJS中的Raycaster应用
date: 2018-03-20 10:06:16
tags: Three.js
categories: 前端
---
### 总结Three.js中的Raycaster的使用经验
#### 先上图：
![Alt text](./0.png)
__   图中的热点查询自数据库，同时鼠标点击可以添加热点__

### 功能简介：
#### 这个项目是按照‘党’的需求做的，党给提出的需求是：
		1. 能够加载三维模型。
		2. 能切换三维模型。
		3. 能在模型上点击添加热点annotation。
		4. 热点上可以修改标注信息。
		5. 鼠标可以操作模型，包括放缩和旋转平移。
		6. 能够将标注信息存到数据库，每次加载模型时不需要重新标注。
按照上面的需求，我做的过程中，遇到的主要的难点就在模型的annotation添加和保存后再加载这块了，因为之前在
Sketchfab.com网站上看到过类似的功能，（PS：老外的技术还是领先的）用起来非常的丝滑，柔顺，便利。这简单的几个功能，不知道是多少工程师的心血，但是可以肯定的是，他们能实现的，我们肯定也能实现。此处省去N多日夜……最终，如愿以偿，虽然代码写得有些冗余，但是还是基本实现了想要的功能。

###  Raycaster是什么？
Raycaster是Three.js中的射线，通过它，可以计算出射线与物体的交点，从而进行模型拾取，或者选中，等其他功能。
我要实现的是鼠标双击模型上的某一点，然后在这个点处添加一个annotation，然后在鼠标推拽物体，旋转物体的时候让annotation随着模型一起运动。

### 思路：
 1. 鼠标双击是要选中模型上的点的，鼠标是屏幕上的点，是二维坐标，但是模型上的点是三维坐标，如何将这个坐标进行转换呢？
 2. 鼠标双击事件给annotaion添加title和content。
 3. annotation跟随模型运动，应该是实时渲染的结果。
### 解决方法与实现过程：
#### 1. 加载模型：
***
    objLoader = new THREE.OBJLoader();
    objLoader.setPath('./obj/');
    objLoader.load('zxj.obj', function (object) {
        object.traverse(function (child) {
            if (child.type === "Mesh") {
                child.geometry.computeBoundingBox();
                child.geometry.verticesNeedUpdate = true;
                child.material.side = THREE.DoubleSide;
            }
        });
        object.name = "zxj";  //设置模型的名称
        object.position.x = -100;//载入模型的时候的位置
        object.position.y = 0;
        object.position.z = -500;
        // 对模型的大小进行调整
        object.scale.x = 0.001;
        object.scale.y = 0.001;
        object.scale.z = 0.001;
        //写入场景内
        scene.add(object);
        objects.push(object);//仿照ThreeJS写法

    });
***
#### 2.如何才能选中鼠标点击的点？
在Three.js中还是能找到一些例子的：[Three.js](https://threejs.org/examples)
__需要注意的地方：鼠标双击进行事件绑定，要监听鼠标双击事件，这里切记不要使用全局鼠标事件监听，而是应该一个一个进行绑定，否者会引起html标签中的鼠标事件失效。__
***
    $('#test').click(onDocumentMouseDown);
	//document.addEventListener('mousedown', onDocumentMouseDown, false); // 这里是个坑啊，一定要注意 鼠标单击事件如果绑定给全局document，其他需要鼠标单击的控件，全部失效
***
##### 鼠标双击事件：
鼠标获取的点是屏幕坐标，因屏幕大小而定，这里要改成设备坐标系，也就是把屏幕坐标转换成从-1到1的标准坐标系，然后使用Camera位置和Mouse点击的点来定位射线方向，通过判断射线是否经过模型来实现在模型上添加点。
我们每次鼠标双击事件就是添加一个热点，所以要将热点存储到一个数组中rays[];
*** 
	var rays = []; //记录多次双击之后的射线与模型的外表面交点
	var annos = new Array(); //热点定义成数组
	var intersects; // 射线与模型的交点，这里交点会是多个，因为射线是穿过模型的，与模型的所有mesh都会有交点，但我们选取第一个，也就是intersects[0]。
	function ondblClick(event) {
	    var clientX;
	    var clientY;
	    event.preventDefault();
	    //获取屏幕坐标,鼠标点击的位置的坐标
	    clientX = event.clientX;
	    clientY = event.clientY;
	    // 获取'转换后的设备坐标（即屏幕标准化后的坐标从 -1 到 +1）:
	    mouse.x = (event.clientX / renderer.domElement.clientWidth) * 2 - 1; //获取鼠标点击的位置的坐标
	    mouse.y = - (event.clientY / renderer.domElement.clientHeight) * 2 + 1;
		raycaster.setFromCamera(mouse, camera);//从相机发射一条射线，经过鼠标点击位置 mouse为鼠标的二维设备坐标，camera射线起点处的相机
	    // intersects 是后者的返回对象数组，
	    intersects = raycaster.intersectObjects(objects[0].children, false); //intersects是交点数组，包含射线与mesh对象相交的所有交点

    /* 如果射线与模型之间有交点，才执行如下操作 */
    if (intersects.length > 0) {
        if (annos.length > 20){
            alert("到达热点上限");
            return;
        }
        else {
            $('#myModal').modal(); //调用模态框
            //加载模态框
            rays.push(intersects[0]); //将射线与模型的第一个交点push到rays数组中.
        }
    }
    }
***
这样，就可以把选出来的点放入到rays[]中了。如何给这些点添加样式呢？我们需要有多少个rays交点创建多少个annotation：创建结点，append到页面，并push到annos数组中。
***
    var div; // 创建anno content的div
    var sp;
    var strong;
    var p;
    var num; // annotation的数量
    sp = document.createElement('p');
    strong = document.createElement('strong');
    p =  document.createElement('p');
    strong.type = 'text';
    div = document.createElement('div');
    div.className = 'annos';
    div.style.background = 'rgba(0, 0, 0, 0.8)';
    document.body.appendChild(div);
    annos.push(div);
***
#### 3.我们使用Ajax向后台传值，把坐标存起来
***
    $.ajax({
        type: 'POST',
        url: './addAnnos.do',
        data:{
            // 'id'     : num, //回头这个地方的id可以都去掉，让数据库的id自增长
            'id'     : $('#recipient-id').val(),
            'title'  : $('#recipient-name').val(),
            'content': $('#message-text').val(),
            'x_point': x_point,
            'y_point': y_point,
            'z_point': z_point
        },
        // 考虑一下，如何把当前div对应的交点的下x,y,z值一起传过去
        success: function(data){
            $(div).attr('data-attr', $('#recipient-id').val());//操作伪dom中的内容，但是不能使用类选择器
            $(div).on('click', function hideAnno() {
                $(div).find("*").toggle('fast');
                if (div.style.background != '') {
                    div.style.background = '';
                } else {
                    div.style.background = 'rgba(0, 0, 0, 0.8)';
                }
            });
            // var v = window.getComputedStyle(div,'::before').getPropertyValue('content');
            // annos[i] 初始化结束后进行赋值
            $('#recipient-id').val("");
            $('#recipient-name').val("");
            $('#message-text').val("");
        }
    });
***
##### css页面：注意伪dom结点的处理
***
	.annos {
	  position: absolute;
	  top: 0;
	  left: 0;
	  z-index: 1;
	  margin-left: 15px;
	  margin-top: 15px;
	  padding: 1em;
	  width: 200px;
	  color: #fff;
	  border-radius: .5em;
	  font-size: 12px;
	  line-height: 1.2;
	  -webkit-transition: opacity .5s;
	  transition: opacity .5s;
	}
	.annos::before {
	  content: attr(data-attr); 
	  position: absolute;
	  top: -30px;
	  left: -30px;
	  width: 30px;
	  height: 30px;
	  border: 2px solid #fff;
	  border-radius: 50%;
	  font-size: 16px;
	  line-height: 30px;
	  text-align: center;
	  background: rgba(0, 0, 0, 0.8);
	}
***

#### 4.更新屏幕中annotation的位置：
annotation的位置是实时渲染的，所以这个方法是对所有的annotation进行位置更新，最终应该放在render渲染器中。
***
	function updateAnnosPosition() {
	    for (var i = 0; i < annos.length; i++){
	        var canvas = renderer.domElement;
	        //var vector = new THREE.Vector3(clientX,clientY,-1);
	        // var vector = new THREE.Vector3(intersects[i].point.x, intersects[i].point.y, intersects[i].point.z);
	        var vector = new THREE.Vector3(rays[i].point.x, rays[i].point.y, rays[i].point.z);
	        vector.project(camera);
	        //这个位置的写法有问题
	        vector.x = Math.round((0.5 + vector.x / 2) * (canvas.width / window.devicePixelRatio)); // 控制annotation跟随物体一起旋转
	        vector.y = Math.round((0.5 - vector.y / 2) * (canvas.height / window.devicePixelRatio));
	        annos[i].style.left = vector.x + "px";
	        annos[i].style.top = vector.y + "px";
			annos[i].style.opacity = spriteBehindObject ? 0.25 : 1;
	    }
	}
	//渲染器
	function render() {
    renderer.render(scene, camera);
    updateAnnotationOpacity(); // 修改注解的透明度
    //updateScreenPosition(); // 修改注解的屏幕位置
    if (annos != null){
        updateAnnosPosition();
    }
}	
***
这就是整个的实现过程了，而后就是将数据存入数据库，并同样将数据库中的数据取出来重新放入一个数组 ，再将对应的x,y,z坐标复原到一个annotation中。利用render进行渲染.