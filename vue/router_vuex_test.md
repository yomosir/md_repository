# vue核心技术相关

## vue-router

官方路由管理工具，管理单页面
1. router.js
	已存在home和about两个router组件
2. 创建新的router
	```
	//Info.vue
	<script>
		export default{
			name: "info"	
		}
  </script>
  //router.js
  {
		path: /info
		name: /info
		component: Info
	}
	
	//App.vue
	<router-link to="/info">Info</router-link>	
	```

## vuex
1. 单向数据流概念
2. 介绍
	- 多个视图依赖于同一个状态
	- 来自不同视同的行为需要变更同一条状态
	- 为Vue.js开发的状态管理模式
	- 组件状态集中管理
	- 组件状态改变遵循统一的规则
3. 例子 （store.js）
	- state  组件状态，集中管理
	- mutations 唯一能改变组件状态的方法集
	- actions

4. 页面引入
```
//Info.vue中：
<script>
	 import store from '@/sotre'
	 export default{
		method: {
			add(){
				//commit 中是方法的名称
				store.commit('increase')
			}
		}
	}
</script>
//router.js
	//状态
	state: {
		count: 0
	},
	//改变状态的方法
	mutations: {
		increase (){
			this.state.count ++
		}
	}
	
//About.vue
import store from '@/store'
export default{
	name: 'about',
	data(){
		return{
			msg: store.state.count
		}
	}
}
```

## 调试
1. `console.log() `,`console.error()`方法
2.  `alert()`
3.  `debugger` 断点，运行到当前位置
4.  将`console.log()`放入一个`output`方法，可以在测试工具调用 `this.output`
5.  `chrome`的`vue`插件
6.  `mounted` 组件挂载完成后的方法 ，`window.vue = this`,调用`output`方法