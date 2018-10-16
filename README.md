## 属性动画和帧动画
web中的动画主要分为`属性动画`和`帧动画`两种，`属性动画`是通过改变dom元素的属性如宽高、字体大小或者transform的scale、rotate等属性，**在一段时间内，属性值按照时间函数变化**来实现的。`帧动画`是通过**在一段时间内按照一定速率替换图片**的方式来实现，这个和传统的动画方式一致。

`帧动画`和`属性动画`各有优缺点：`属性动画`不需要加载什么资源，只需要不断改变属性值，触发浏览器的重新计算和渲染就可以了。`帧动画`能够实现更为复杂的动画效果，比如游戏角色的技能特效等，但需要加载一些图片。

web中的交互动画特效一般都比较简单，所以`属性动画`用的更多，`帧动画`比较少。游戏中的动画效果追求绚丽，基本都会用`帧动画`，部分会结合`属性动画`。
## AE和Spine
`AE`全称After Effets，是Adobe公司推出的用于处理视频和图形的软件。ui界面中的动画效果很多都是用`AE`来做的。
![](https://upload-images.jianshu.io/upload_images/5077517-58c087243497f8ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Spine`是针对软件和游戏中的2d动画的，制作动画比`AE`更专业。游戏中用的比较多。

![](https://upload-images.jianshu.io/upload_images/5077517-6ed59e0ef6c281ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Lottie和Spine Runtime
Lottie是Airbnb推出的可以解析AE导出的包含动画信息的json文件的库，支持Android、iOS，React Native等平台。

Spine Runtime是Spine提供的Spine导出的动画解析的库，支持各种游戏引擎，如egret、cocos2d-x等。

![](https://upload-images.jianshu.io/upload_images/5077517-9a828ffdbc600975.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Lottie渲染时需要提供一系列的图片，渲染不同帧的时候会使用组合不同的图片。Spine Runtime使用一个小图片合成的大图片，渲染时会取不同的部分来渲染。

![Lottie](https://upload-images.jianshu.io/upload_images/5077517-762fbd516cd2926a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Spine Runtime](https://upload-images.jianshu.io/upload_images/5077517-8d9ac9086e26576e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此外，Lottie支持svg、dom、canvas三种渲染方式，而Spine Runtime只支持canvas。

实际开发中，Lottie在应用中用的多，Spine Runtime在游戏中用的多。但并不代表他们不能在另外的场景中使用。

## 在web应用中使用Spine Runtime

需求中涉及到动画，设计师没有使用AE，使用Spine来设计的。导出的文件也是Spine特有的格式，于是我就对Spine进行了调研。

经过调研我发现Spine的Runtime中有Html Canvas，这就是他可用在web应用中的基础。

我把demo下下来看了一下，通过阅读代码，替换对应的资源文件，删减部分无用代码之后，对Spine Canvas Runtime的使用有了一些心得。

##### 动画资源
Spine导出的文件有3个，xxx.atlas、xxx.json、xxx.png

![](https://upload-images.jianshu.io/upload_images/5077517-ee2d4d17096c3ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

xxx.json是动画的描述文件，分为skeleton、bones、slots、skins、animations这5部分

![](https://upload-images.jianshu.io/upload_images/5077517-f505540f0d17d77b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们没必要去详细了解，只需要知道这里的animations下有一个叫做animation的动画就可以了。

xxx.png是图片文件，因为图片整合到了一起，所有有一个xxx.atlas文件来描述哪个小图片在什么地方。

![](https://upload-images.jianshu.io/upload_images/5077517-a72be4f887027262.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

资源就这3个文件，接下来就是动画实现的代码了。

##### 动画实现代码

经过分析，整体流程就是加载资源后，通过不断的重绘来显示一帧帧的图片，图片的更新是通过时间的毫秒数来驱动的。

不断重绘的逻辑：

![](https://upload-images.jianshu.io/upload_images/5077517-241a4c9faa6a4fb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5077517-f16139c98e942fda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

改变绘制内容的逻辑：
![](https://upload-images.jianshu.io/upload_images/5077517-4dfe760ca93ce161.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每次绘制传入两次绘制的时间差，spine runtime会计算出当前应该渲染的内容是什么。

上面是核心的不断重绘的机制和更新渲染内容的机制，整体的流程如下：

![](https://upload-images.jianshu.io/upload_images/5077517-8e16db5c73357685.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先加载资源，然后不断re-render。

整体代码如下，相关资源和代码见文章最后的github链接。

```html

<!-- saved from url=(0068)http://esotericsoftware.com/files/runtimes/spine-ts/examples/canvas/ -->
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=windows-1252">
<script src="./js/spine-canvas.js"></script>
<style>
	* { margin: 0; padding: 0; }
	body, html { height: 100% }
	canvas { position: absolute; width: 100% ;height: 100%; }
</style>
</head>
<body>
<canvas id="canvas" width="398" height="588"></canvas>

<script>


var lastFrameTime = Date.now() / 1000;
var canvas, context;
var assetManager;
var skeleton, state, bounds;
var skeletonRenderer;

// var skelName = "spineboy-ess";
var skelName = "pk_list_flash";
// var animName = "walk";
var animName = "animation";


function init () {
	canvas = document.getElementById("canvas");
	canvas.width = window.innerWidth;
	canvas.height = window.innerHeight;
	context = canvas.getContext("2d");

	skeletonRenderer = new spine.canvas.SkeletonRenderer(context);
	// enable debug rendering
	skeletonRenderer.debugRendering = false;
	// enable the triangle renderer, supports meshes, but may produce artifacts in some browsers
	skeletonRenderer.triangleRendering = false;

	assetManager = new spine.canvas.AssetManager();

	assetManager.loadText("assets/" + skelName + ".json");
	assetManager.loadText("assets/" + skelName.replace("-pro", "").replace("-ess", "") + ".atlas");
	assetManager.loadTexture("assets/" + skelName.replace("-pro", "").replace("-ess", "") + ".png");

	requestAnimationFrame(run);
}

function run () {
	if (assetManager.isLoadingComplete()) {
		var data = loadSkeleton(skelName, animName, "default");
		skeleton = data.skeleton;
		state = data.state;
		bounds = data.bounds;
		requestAnimationFrame(render);
	} else {
		requestAnimationFrame(run);
	}
}

function loadSkeleton (name, initialAnimation, skin) {
	if (skin === undefined) skin = "default";

	// Load the texture atlas using name.atlas and name.png from the AssetManager.
	// The function passed to TextureAtlas is used to resolve relative paths.
	atlas = new spine.TextureAtlas(assetManager.get("assets/" + name.replace("-pro", "").replace("-ess", "") + ".atlas"), function(path) {
		return assetManager.get("assets/" + path);
	});

	// Create a AtlasAttachmentLoader, which is specific to the WebGL backend.
	atlasLoader = new spine.AtlasAttachmentLoader(atlas);

	// Create a SkeletonJson instance for parsing the .json file.
	var skeletonJson = new spine.SkeletonJson(atlasLoader);

	// Set the scale to apply during parsing, parse the file, and create a new skeleton.
	var skeletonData = skeletonJson.readSkeletonData(assetManager.get("assets/" + name + ".json"));
	var skeleton = new spine.Skeleton(skeletonData);
	skeleton.flipY = true;
	var bounds = calculateBounds(skeleton);
	skeleton.setSkinByName(skin);

	// Create an AnimationState, and set the initial animation in looping mode.
	var animationState = new spine.AnimationState(new spine.AnimationStateData(skeleton.data));
	animationState.setAnimation(0, initialAnimation, true);
	animationState.addListener({
		event: function(trackIndex, event) {
			// console.log("Event on track " + trackIndex + ": " + JSON.stringify(event));
		},
		complete: function(trackIndex, loopCount) {
			// console.log("Animation on track " + trackIndex + " completed, loop count: " + loopCount);
		},
		start: function(trackIndex) {
			// console.log("Animation on track " + trackIndex + " started");
		},
		end: function(trackIndex) {
			// console.log("Animation on track " + trackIndex + " ended");
		}
	})

	// Pack everything up and return to caller.
	return { skeleton: skeleton, state: animationState, bounds: bounds };
}

function calculateBounds(skeleton) {
	var data = skeleton.data;
	skeleton.setToSetupPose();
	skeleton.updateWorldTransform();
	var offset = new spine.Vector2();
	var size = new spine.Vector2();
	skeleton.getBounds(offset, size, []);
	return { offset: offset, size: size };
}

function render () {
	var now = Date.now() / 1000;
	var delta = now - lastFrameTime;
	lastFrameTime = now;

	resize();
	
	state.update(delta);
	state.apply(skeleton);
	skeleton.updateWorldTransform();
	skeletonRenderer.draw(skeleton);

	requestAnimationFrame(render);
}

function resize () {
	var w = canvas.clientWidth;
	var h = canvas.clientHeight;
	if (canvas.width != w || canvas.height != h) {
		canvas.width = w;
		canvas.height = h;
	}

	// magic
	var centerX = bounds.offset.x + bounds.size.x / 2;
	var centerY = bounds.offset.y + bounds.size.y / 2;
	var scaleX = bounds.size.x / canvas.width;
	var scaleY = bounds.size.y / canvas.height;
	var scale = Math.max(scaleX, scaleY) * 1.2;
	if (scale < 1) scale = 1;
	var width = canvas.width * scale;
	var height = canvas.height * scale;

	context.setTransform(1, 0, 0, 1, 0, 0);
	context.scale(1 / scale, 1 / scale);
	context.translate(-centerX, -centerY);
	context.translate(width / 2, height / 2);
}

(function() {
	init();
}());

</script>
</body></html>
```
## 总结

web中的动画有属性动画和帧动画两种，帧动画常用的库有Lottie和Spine Runtime，用哪一种取决于动效师使用的是AE还是Spine，其中Spine多用于游戏的动画。

从图片资源的管理方式、支持的渲染方式和平台这几个方面比较了Lottie和Spine Runtime的区别：Spine 多用于游戏，图片资源整合到一起并且提供atlas文件来标明对应图片位置，支持canvas的渲染方式，支持各种游戏引擎。Lottie多用于应用，图片资源分开存放，支持canvas、svg、dom三种渲染方式，并且支持Android、ios、React Native等平台。仅从canvas角度看，两者区别并不大。

因为动效师选择了Spine来设计动效，所以我调研了Spine Runtime的动画实现方案，研究了Spine的动画资源和Spine Cavas Runtime的代码实现、运行流程，全部代码见[github]()。

















