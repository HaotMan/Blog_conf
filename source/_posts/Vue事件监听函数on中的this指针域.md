---
title: Vue事件监听函数on中的this指针域
date: 2018/10/27 20:57:17 
categories: 

- [前端, Vue]

tags: 

- 前端
- Vue
- grammar

permalink: vue-this-pointer

---

使用eventBus在两个组件A，B之间通讯：

<!--more--> 

```js
//定义全局eventBus用于事件传递
var bus = new Vue();
//A组件
var A = Vue.component({
	...
	
	data:{
		dataA:1,
	},
	//在钩子函数中将监听_event事件
	created:function(){
		var this_com = this;
		bus.$on('_event', function(value){
			this_com.dataA = value;
		})
	},
	
	...
})

//B组件
var B = Vue.component({
	...
	data:{
		dataB = 2;
	},
	methods:{
		changeA:function(){
			//触发事件_event
			bus.$emit('_event', this.dataB);
		},
	},
	template:`
		<div v-bind:click="this.changeA"></div>
	`
})
```
可以看到，在组件A监听事件_event事件的函数前需要提前保存this指针为this_com，因为在监听函数中的this不再指向A组件本身，而是指向事件监听者bus。

另一种方案是用箭头函数代替事件监听函数，因为箭头函数可以将指针域绑定到当前组件

```js
var A = Vue.component({
	...
	
	data:{
		dataA:1,
	},
	//在钩子函数中将监听_event事件
	created:function(){
		var this_com = this;
		bus.$on('_event', value=>{
			this_com.dataA = value;
		})
	},
	
	...
})
```