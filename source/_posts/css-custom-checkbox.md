---
title: css 定制 checkbox
date: 2017-09-12 16:15:33
tags: 
categories: css checkbox
---

本文介绍如何通过css定制checkbox选择框

<!--more-->

> 实现思路：  
> 
> 
>  

纯css实现的主要手段是利用label标签的模拟功能。label的for属性可以关联一个具体的input元素，即使这个input本身不可被用户可见，有个与它对应的label后，用户可以直接通过和label标签交互来替代原生的input——而这给我们的样式模拟留下了空间。简而言之就是

> 隐藏原生input，样式定义的过程留给label （那为什么不直接改变checkbox的样式？因为checkbox作为浏览器默认组件，样式更改上并没有label那么方便，很多属性对checkbox都是不起作用的，比如background,而label在样式上基本和div一样’任人宰割’)      

而在选择事件上，由于css的“相邻选择符(E+F)”的存在，让我们可以直接利用html的默认checkbox，免去了js模拟选择的麻烦。

### 1. 同级label和input标签

> html

```
<div class="custom-checkbox">
  <input type="checkbox" class="magic-checkbox" id="input_id" v-on:click.stop/>
  <label v-bind:class="lableClass" for="input_id" v-on:click.stop>{{show_text}}</label>
</div>
```

> css

```
/*先隐藏真正的checkboxbox*/
.magic-checkbox {
  position: absolute;
  display: none;
}

/*//为与checkbox搭配的label使用样式,//相对定位，方便其内容的伪元素进行定位*/
.magic-checkbox + label {
  position: absolute;
  display: block;
  padding-left: 30px;
  cursor: pointer;
  vertical-align: middle;
  font-size: 14px;
  color: #aaa;
}

/*label添加:before伪元素，用于生成一个带边界的正方形，模拟复选框的轮廓*/
.magic-checkbox + label:before {
  position: absolute;
  top: 0;
  left: 0;
  display: inline-block;
  content: '';
  border: 1px solid #dadada;
  border-radius: 3px;
  background-color: white;
}
/*为checkbox添加:after伪元素，作用是生成一个√图形，模拟checkbox选中状态，未选中状态下会被隐藏*/
/*实现√图形很简单：设置一个长方形，去掉其上边界和左边界，剩下的2个边界旋转45度就得到√形状  */
.magic-checkbox + label:after {
    box-sizing: border-box;
    /*width: 6px;*/
    /*height: 12px;*/
    transform: rotate(45deg);
    -webkit-transform: rotate(45deg);
    -moz-transform: rotate(45deg);
    -o-transform: rotate(45deg);
    -ms-transform: rotate(45deg);
    border: 2px solid #fff;
    border-top: none;
    border-left: none;
    position: absolute;
    display: inline-block;
    content: '';
    opacity: 0.2;
  }
 /*单击label，隐藏的checkbox为checked状态，这时设置checked状态下搭配label的:before伪元素背景和边界颜色*/
  .magic-checkbox:checked + label:before {
    animation-name: none;
    border: white;
    background: #ffd54f;
  }
/*checked状态的checkbox搭配的label伪元素:after此时设置显示，那么√就显示出来了*/
  .magic-checkbox:checked + label:after {
    opacity: 1;
  }

  .magic-checkbox:disabled + label:before {
    background-color: #d8d8d8;
  }

  .magic-checkbox:disabled + label:after {
    opacity: 1;
  }

  /*针对login组件和avatar组件中的使用设计两中size的class：box-login、box-avatar*/
  .magic-checkbox + label.box-login:before {
    width: 18px;
    height: 18px;
  }

  .magic-checkbox + label.box-login:after {
    top: 2px;
    left: 7px;
    width: 6px;
    height: 12px;
  }

  .magic-checkbox + label.box-avatar:before {
    width: 12px;
    height: 12px;
  }
  .magic-checkbox + label.box-avatar:after {
    top: 1px;
    left: 4px;
    width: 4px;
    height: 8px;
  }
```

**问题：**   
同级的input使用id与label关联起来，当页面需要使用多个checkbox，特别在某种列表中循环生成多项，此时必须使用某种方式保证id的唯一，否则点击事件的触发会出错。     

解决方案：    
可以用 js 生成动态 id 吧。在循环中给每个 input 生成 id = "awesome"+i，同时把 label 的 for 属性值也设置成一样的。下面是简单的示例：    

	var body=document.getElementsByTagName('body')[0];
	        for(var i=0;i<3;i++){
	            var input=document.createElement("input");
	            input.id="input"+i;
	            var label=document.createElement("label");
	            label.setAttribute("for","input"+i);
	            label.innerHTML="点击";
	            body.appendChild(label)
	            body.appendChild(input)
	        }
   

但是页面中有多个列表需要展示checkbox时，逐一操作,保证唯一性，会很繁琐。      

*所以使用下面2中的方式定制checkbox。*

### 2. label内部嵌套input

> html

```
	<div>
	  <label class="checkbox-label" v-on:click.stop>
	    <input type="checkbox" class="checkbox-input"/>
	    <i v-bind:class="lableClass"></i>{{show_text}}
	  </label>
	</div>

```

> css

```
.checkbox-label{
  display: block;
  margin-bottom: 10px;
  position: relative;
}

.checkbox-input{
  position: absolute;
  display: none;
}
.checkbox-input + i{
  display: inline-block;
  /*width: 18px;*/
  /*height: 18px;*/
  margin-right: 10px;
  background: white;
  vertical-align: middle;
  content: '';
  border: 1px solid #dadada;
  border-radius: 3px;
}

.checkbox-input:checked + i {
  background-color: #ffd54f;
  border-color: transparent;
}

.checkbox-input + i:after {
  position: absolute;
  /*top: 4px;*/
  /*left: 7px;*/
  box-sizing: border-box;
  /*width: 6px;*/
  /*height: 12px;*/
  transform: rotate(45deg);
  -webkit-transform: rotate(45deg);
  -moz-transform: rotate(45deg);
  -o-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  border: 2px solid transparent;
  border-top: none;
  border-left: none;
  display: inline-block;
  content: '';
  opacity: 1;
}

.checkbox-input:checked + i:after {
  border-color: white;
}

.checkbox-input + i.box-login {
  width: 18px;
  height: 18px;
}

.checkbox-input + i.box-login:after {
  top: 4px;
  left: 7px;
  width: 6px;
  height: 12px;
}

.checkbox-input + i.box-avatar {
  width: 12px;
  height: 12px;
}
.checkbox-input + i.box-avatar:after {
  top: 6px;
  left: 4px;
  width: 4px;
  height: 8px;
}
```


### 3. retained issues
1）如何动态设置size 
目前是通过设置额外的class，如box-avatar、box-login，实现定制在不同应用场景，展示的size等具体效果。    
具体实现如下：
> //html   

```
//通过<i>标签的class 绑定到实际场景对应的style
<div>
  <label class="checkbox-label" >
    <input type="checkbox" class="checkbox-input" :disabled="disabled"/>
    <i v-bind:class="boxtype"></i>{{show_text}}
  </label>
</div>
```

> ts   

```
//声明一个枚举类型
export enum CheckboxType {
  box_login = 1,
  box_avatar = 2,
}

//在自定义的checkbox组件中声明一个prop，用于定制：type 
props: {
    type: CheckboxType,
  }
//根据使用该组件的地方传递过来的type，映射到实际的class类型，达到定制的效果
switch (this.$props.type) {
      case 1:
        this.boxtype = CheckboxType[1];
        break;
      case 2:
        this.boxtype = CheckboxType[2];
        break;
    }
```

希望发现更简便的定制方式。