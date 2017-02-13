#3D货架例子

[查看例子](http://demo.web101.cn/3D_Shelves)

> 其实很早就知道three.js，看着比网页功能革命性的效果，再加老外写的功能太炫，给自己产生一种无名的不可越心态，一直也不怎么关注，偶尔公司的小应用用css3也能满足过去，为什么会用一个货架的做例子，我早期公司，是家开发3方物流应用，公司有几个美国库房，实现机器人作业，要在屏幕上模拟出物品在那个区域那个货架上，物品上包装箱能生成条形码，直接用扫描枪，也就是机器作业的库房不能让人随意进入的，一切的操作都在管理办公室内完成，这几天有时间看了官方教程，终于入门了，下面分享个人一点学习心得，希望能给你带来一些帮助。

##three.js现在支持的模型文件非常的多，查看官方源码js\loaders目录还就道
###支持模型文件类型（不是搞3d专业）我称为模型

 ```javascript 

  SVG,KMZ,Play,TTF,Baby,PLYL,3MF,OBJ


```

  很多，我的这个是obj，是blender导出的，
 当然你可以用其他3d软件，导出的模型大多支持，这里说的是复杂的模型，这个货架如果直接在three.js内手工写几何，也能实现，复杂地方是4边的多孔立柱,遗憾的是obj文件太多，建模是如果这些孔创建的不规则，影响导出的obj文件大小，这里的规则就是建模时你看到的一条条细线，一定要对称有章法，这样导出的文件小很多。

 ## 入门抽象知识关键点

 * 渲染器 ---多种svg,webgl,Canvas

 * 场景   ---Scene 

 * 摄像机 Camera

 * 灯光  --light --很多种，环境光，定向光，辅助关灯

 * 材质 --MATERIAL ----很多种

 * 网格  --Mesh

 * 几何  --geometry --很多种，如圆，方，锥，等

 理解了以上7点，在找相关的属性配置和方法
 的应用你就可以入门了，关键处是建模，现在3d软件很多，找个适合自己理解的用，three.js提供的编辑器不是很方便创建物体，在加上容易在浏览器内崩掉，建议用3d软件辅助建模，分块在.js程序内去组装这样文件也小。

 如下是本实例的源码,每一行我都有注释

 ###本实例要引入的js文件

 *  three.min.js --这是压缩的不用说了
 * OBJLoader.js  --这个是加载obj文件，并行处理一些东西，官方的

 * 要加载什么样的模型相对应的处理js,可以在官方源码目录内js/loaders下找到

 ```javascript

 var container;
    var camera, scene, renderer;
    var mouseX = 0,mouseY = 0;
    var windowHalfX = window.innerWidth / 2;
    var windowHalfY = window.innerHeight / 2;
 

    init();
    // animate();
    function init() {

        container = document.createElement('div');
        document.body.appendChild(container);

        //定义一个透视摄像机--three.js还有一个平视摄像机
        //这个按动画要求创建
        camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000);
        //摄像机位置z轴，就是远进--把镜头推近了物体就变大，相反变小
        camera.position.z = 360;

        //必须要光-3维

        // 创建环境光 ---必用
        scene = new THREE.Scene();
        var ambient = new THREE.AmbientLight(0x101030);
        scene.add(ambient);

        //创建定向光  -- 必用
        var directionalLight = new THREE.DirectionalLight(0xffeedd);
        directionalLight.position.set(0, 0, 1);
        scene.add(directionalLight);


        //加载进度事件
        var onProgress = function(xhr) {
            if (xhr.lengthComputable) {
                var percentComplete = xhr.loaded / xhr.total * 100;
                var nubs = Math.round(percentComplete, 2);
                var loadtxt = '完成:' + nubs + '%';
                var loading = document.querySelector("#loading");
                loading.innerText = loadtxt;
                if (nubs >= 99) {
                    loading.parentNode.removeChild(loading);
                    //显示切换材质按钮
                   document.querySelector(".ReMaterial").style.display = 'block';
                }

            }
        };
        var onError = function(xhr) {};


        //--------------------------------------------

        //修改-object材质           
      
        var ReOBJMaterial = function(object,recolor) {
            //创建材质
            var MeshPhon =  new THREE.MeshPhongMaterial({
                color: recolor
            });
             
             //修改material
                object.traverse(function(child) {
                if (child instanceof THREE.Mesh) { 
                    child.material = MeshPhon;
                }
            });
        }

       
        //-------------------------------------------



        // 加载一个OBJLoader文件，
        //three.js支持很多种3d模型类型
        //.obj类型未必最好，如果模型创建的不规则
        //导出的文件大了要命，如下货架4个边柱的孔
        //创建时这些孔会不规则，当然有时可以一条条
        //线修正，那样导出的文件会小很多。
        //创建导出文件大小和创建模型的熟练有关。             

        var loader = new THREE.OBJLoader();      
        loader.load('Shelves1.obj', function(object) {        
           
            //修改object--属性
            ReOBJMaterial(object,0xFFBF00);

            object.name="Shelves";

            object.position.y = -40;

            object.position.x = 0;

            object.position.z = -60;

            object.rotation.x = -45;
            //写入场景内
            scene.add(object);

            //动画时每次渲染--修改gruop值
            function render() {
                //每次修改物体(组)旋转z轴
                object.rotation.z += 0.01;
                //重绘渲染器
                renderer.render(scene, camera);
                //h5特有的，循环一次，这么写就是循环后又执行
                //就变成无限循环了。
                requestAnimationFrame(render);
            }

           //加载完成后回调
            render();

         //是加载文件的的事件

        }, onProgress, onError);



        //创建WebGL渲染器，three.js有多种渲染器
        renderer = new THREE.WebGLRenderer();
        //当时设备分辨率
        renderer.setPixelRatio(window.devicePixelRatio);
        //画面大小-用窗口大小
        renderer.setSize(window.innerWidth, window.innerHeight);
        //事先已经在body内创建过一个div,
        //现在，在这个div内创建画布-
        //renderer.domElement就是返回的canvas
        container.appendChild(renderer.domElement);

        //在window上添加resize事件       
        window.addEventListener('resize', onWindowResize, false);



        //给修改材质加事件---切换皮肤，目前只改色，可以配置
        //材质MATERIAL属性
        var ReMaterialDom = document.querySelectorAll(".ReMaterial span");
        [].forEach.call(ReMaterialDom, function(el) {
            //给每个一个span加点击事件
            el.addEventListener('click', function(e) {

                var typecolor=e.target.className;
                if(typecolor == "queot"){
                  typecolor=0x00D900;
                }
                if(typecolor == "red"){
                  typecolor=0xff0000;
                }
                 if(typecolor == "blue"){
                  typecolor=0x007FFF;
                }
                //获取场景内的object
                var object = scene.getObjectByName("Shelves");
                ReOBJMaterial(object,typecolor);

            }, false);
        });

        //窗口大小改变事件
        function onWindowResize() {
            windowHalfX = window.innerWidth / 2;
            windowHalfY = window.innerHeight / 2;
            //修改摄像机视野--高宽
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            //修改渲染器--高宽
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
    }


 ```


[查看例子](http://demo.web101.cn/3D_Shelves)
