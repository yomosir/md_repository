# vue的基础知识
需要node,cnpm,vue-cli等环境

### 认识vue
1. 推荐使用BootCDN来引入vue.js

```
<script src="https://cdn.bootcss.com/vue/2.5.21/vue.min.js"></script>
```

2. vue基本语法
```javascript
new Vue({
    el: '.class',//绑定的位置
    data:{//这是vue在页面上的数据。可在页面通过{{msg}}取到数据
        msg: 'hello vue' 
    }
})
```

3. 文件结构,模板语法
    template，script，样式 ---一个vue文件内的结构
   
```javascript
new vue({
    el: '#App',//绑定的位置
    data:{
        msg: 'hello vue',
        template: '<div>template</div>',
        url: 'www.abidu.com'
    }
    methond:{
        submit: function(){
        }
    }
})

v-html : 可以让template在页面上按照html渲染
v-bind 绑定属性  如： v-bind:href='url' 可以使用：`:href`
v-on 绑定事件  如：v-on:click="submit()" 可以使用：`@click`

```
 
 4. 计算属性与侦听器
 
- 计算属性：`computed`<一个变量，异步场景> 
- 侦听器：`watch` <监听多个变量，数据联动>
```javaScript
new Vue({
    el: '#app',
    data: {
        msg: 'hello world'
    },
    watch: {//侦听器
        //当msg属性发生改变时，回调用这个函数
        msg: function(newval,oldval){
            console.log('newval is' + newval)
            console.log('oldval is' + oldval)
        }
    },
    computed: {//计算属性
        //调用时会输出return值，会将方法内的变量发生变化时都会调用，值会监听实例内属性，也就是new Vue内部
        msg1: function(){
            return 'computed: ' + this.msg
        }
    }
})
```
 
 
### 基本概念
- 条件渲染

`v-if`,`v-else , v-else-if`,`v-show`

```
<div v-if="count > 0">
	count大于0 ，count是 {{count}}
</div>
<div v-else>
	count值是：{{count}}
</div>
<div v-show="count == -1">
	show:{{count}}
</div>
```

- 列表渲染

`v-for`,`v-for & v-if`,`v-for 高阶用法`

```
<div v-for="item in list">
	{{item}} // 其中list是一个数组或者可遍历的元素
</div>
```

- 事件处理
- class,style绑定

```
<div 
	:class=['active','add',{'another': item.age < 30]
	:style="styleMsg">
	
</div>
<script>
	new Vue({
		data:{
			styleMsg: {
				color: 'red',
				textShadow: '0 0 5px green',
			},
		}
	});
</script>
```