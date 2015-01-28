---
layout: post
title: "Run script after Angularjs ng-repeat finishing"
date: 2015-01-28 21:50
comments: true
categories: [Angularjs]
---

这几天在做一个小项目的时候，用到了[Freewall](http://vnjs.net/www/project/freewall/)这个很nice的插件。它可以很方便的替你实现很多不错的grid layout。但在用到[Pinterest style layout](http://vnjs.net/www/project/freewall/example/pinterest-layout.html)的时候遇到了一个问题。官方给出的example如下:<!--more-->


``` html
<div id="freewall" class="free-wall">
    <div class="brick">
        <img src="i/photo/1.jpg" width="100%">
        <div class="info">
            <h3>Freewall</h3>
            <h5>Pinterest layout</h5>
        </div>
    </div>
    <div class="brick">
        <img src="i/photo/2.jpg" width="100%">
        <div class="info">
            <h3>Freewall</h3>
            <h5>Pinterest layout</h5>
        </div>
    </div>
    ....
</div>
<script type="text/javascript">
    var wall = new freewall("#freewall");
    wall.reset({
        selector: '.brick',
        animate: true,
        cellW: 200,
        cellH: 'auto',
        onResize: function() {
            wall.fitWidth();
        }
    });

    wall.container.find('.brick img').load(function() {
        wall.fitWidth();
    });
</script>
```


这里省略了样式的代码，直接看这些最核心的部分。一看很简单，就一堆自己的图片加上用`freewall`初始化一下整个图片墙就行了。可是我自己的代码怎么都没法在网页加载的初期就能`fitWidth`。但是可以响应`onResize`。我的代码如下。(就是把example的[html转成jade](http://html2jade.org/)，偷懒了^_^)


``` jade
extends layout

block content
  #bgimg(ng-if='!currTrend')
    img(src='/images/bgImg.png', width='100%')
  #bgimg(ng-if='currTrend')
    #freewall.free-wall(ng-hide='!searched')
      .brick(ng-repeat='tweetinfo in trendinfo')
        img(ng-src='{{tweetinfo.imageURL}}', width='100%')
        .info
          ...

    script(type='text/javascript').
      var wall = new freewall("#freewall");
      wall.reset({
        selector: '.brick',
        animate: true,
        cellW: 200,
        cellH: 'auto',
        onResize: function() {
          wall.fitWidth();
        }
      });
      wall.container.find('.brick img').load(function() {
        wall.fitWidth();
      });
```


思来想去可能是我用了Angularjs的ng-repeat动态从服务器端读图片过来的问题，我试了下，一个个的手打这些图片列表是可以出来`fitWidth`的效果的。综合以上分析，既然用ng-repeat的时候是可以响应`onResize`的，说明脚本肯定是被执行了，但是整个图片墙没有去`fitWidth`只能说明这些`brick`是异步添加的。实际运行的时候，脚本早就先运行完了，才慢慢的从服务器端通过http读过来的图片。这样`wall.container.find('.brick img').load`之前执行了也没什么效果。那么怎么才能同步这些脚本呢？查来查去，发现可以用Angularjs里面的`directive`来实现这一过程。

首先给循环中的每个`brick`添加一个检测ng-repeat完成的事件发射器，并将其封装成一个叫`repeatBrick`的directive。


``` javascript
angular.module('YourModule',[])
  .controller('YourCtrl', function ($scope, $http) {
   // do other things
})
.directive('repeatBrick', function() {
  return function(scope, element, attrs) {
    if (scope.$last){		// detect the last brick
      scope.$emit('LastBrick');			// emit the `LastBrick` event
    }
  }
})
```


然后在自定义一个`freewall`上的directive。在里面监听`LastBrick`事件，并在其回调函数中执行之前的脚本就行了。


``` javascript
.directive('theFreewall', function() {
  return function(scope, element, attrs) {
    scope.$on('LastBrick', function(event){		// listen to the `LastBrick` event
      var wall = new freewall("#freewall");		   // run the script in callback function 
      wall.reset({
        selector: '.brick',
        animate: true,
        cellW: 200,
        cellH: 'auto',
        onResize: function() {
          wall.fitWidth();
        }
      });
      wall.container.find('.brick img').load(function() {
        wall.fitWidth();
      });
    });
  };
});
```


当然这时候在模板中还要加上这两个directives。


``` jade
extends layout

block content
  #bgimg(ng-if='!currTrend')
    img(src='/images/bgImg.png', width='100%')
  #bgimg(ng-if='currTrend')
    #freewall.free-wall(ng-hide='!searched' the-freewall)		// add `theFreewall` directive
      .brick(ng-repeat='tweetinfo in trendinfo' repeat-brick)	// add `repeatBrick` directive
        img(ng-src='{{tweetinfo.imageURL}}', width='100%')
        .info
          ...
```


这样就能确保在读取完这些图片之后立刻执行脚本进行布局。总结来说就是传统的html+js是顺序执行的，但是一旦引入模板引擎或者像Angularjs这样的指令之后，很可能就变成异步执行的了，这是个得注意的小坑。


