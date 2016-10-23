---
tags: Echarts
categories: 前端
title: Echarts使用地图时发现地图行政区划没更新
date: 2016年09月27日 星期二
---
### 使用地图时发现地图行政区划没更新

在做一个项目的demo时，老大让我用echarts在首页做一个上海地图，上面显示各个区的数据。

略一看，挺简单的嘛！下载js文件，在项目中加入链接，将echarts的示例复制一个过来。诶，json方式有问题，那就试试js方式吧。可以了！（lixer：既然js方式可以还去管json方式为什么不行干什么。）

好了，接下来就更改更改，添加一些想要的东西。标题怎么加啊，地图上某个区域怎么加点击事件啊，，，只能去看api了，倒腾了2个小时，终于搞定了。下面是成果：

<!-- more -->

```
<script src="../js/echarts/echarts.min.js"></script> //这里需要下载完全版的，或者定制时将map选上
<script src="../js/echarts/shanghai.js"></script>
<div id="map_main" style="margin-left:40%;height:550px;"></div>
<script>
var myChart = echarts.init(document.getElementById('map_main'));
option = {
    title: {
        text: '上海未成年人分布情况',
        left: 'center'
    },
    tooltip: {
        trigger: 'item'
    },
    visualMap: {
        min: 0,
        max: 2500,
        left: 'right',
        top: 'bottom',
        text: ['人数最多','人数最少'],
        calculable: true
    },
    series: [
        {
            name: '未成年人数',
            type: 'map',
            mapType: 'shanghai',
            roam: true,				//地图缩放
            label: {
                normal: {
                    show: false
                },
                emphasis: {
                    show: true
                }
            },
            data:[
				{name: '崇明县',code:'1',value: Math.round(Math.random()*1000)},
				{name: '宝山区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '嘉定区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '青浦区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '杨浦区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '虹口区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '普陀区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '静安区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '黄浦区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '卢湾区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '长宁区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '徐汇区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '浦东新区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '松江区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '闵行区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '金山区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '奉贤区',code:'1',value: Math.round(Math.random()*1000)},
				{name: '南汇区',code:'1',value: Math.round(Math.random()*1000)}
			]
        }
    ]
};
myChart.setOption(option);
myChart.on('click', function (params) {
    $(".check",window.parent.document).fadeIn(500);
    $(".check .pop-box",window.parent.document).animate({top:20,opacity:"1"},500);
    $(".check .pop-box iframe",window.parent.document).remove();
    $(".check .pop-box",window.parent.document).append("<iframe src='person/personinfo/list1.html?quxian="+params.name+"' id='check-iframe' name='check-frame' frameborder='0' width='100%' height='570'></iframe>");
});
</script>
```

这就完了。不对啊，突然发现地图上静安区和闸北区，可是现在这两个区合并了啊，怎么办？

1、去官网找找有没有最近更新的上海地图。找了半天，没有啊。只好又去github上从源代码中找找，可是还是不是最新的，应该是没更新了，我如是想到。

2、我试试自己改改json文件，好吧，上网查找canvas，echarts地图实现，找到关键词geoJson，去查geoJson。弄了半天发现echarts的json中路径是压缩了的根本没法改啊。

3、怎么办？去找地图文件，然后转成geoJson数据。只好去找了，找了半天，浏览起echarts的github的issue来了，无聊，心累，突然看见[一个issue](https://github.com/ecomfe/echarts/issues/2349)下看到ECharts 地图数据在线生成工具，很兴奋啊。终于搞定了。

解决问题了后，我心里琢磨着这个官方工具怎么没记载呢，回去有将echarts3文档浏览了一次，没找到，我就比较纳闷儿了，怎么会没有呢。最后，我在一个大版本之前的echarts2中发现了，让我一顿好找。

最后吐槽一下echarts，文档超难读，并且还不全。
