![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/0QvjnAaQy8dRnXo5/img/df0320c9-8724-4f1f-994d-bf43ca29e81b.png)



之前在做 前端错误日志统计 项目的时候，因为要使用到 axios 进行接口的请求，当时没有为后期扩展做处理， 现在回过头来看，打算把 axios 在 vue 中的封装和使用整理一下。



## 一、Axios的简单介绍

基于Promise的HTTP客户端，用于浏览器(xmlHttpRequest)和node.js(http)，支持拦截和响应.



## 二、Axios的封装

#### 安装

```
yarn add axios
```



#### 引入

```
import { axios } from 'axios';
import QS from 'qs'; // 用qs模块，序列化post类型的数据
import { Message } from 'element-ui'; // 根据自身情况 引入合适的提示组件 
```



#### 配置不同环境下的baseURL

```
if (process.env.NODE_ENV == 'development') {
    axios.defaults.baseURL = 'https://xbbdev.com'
} else if (process.env.NODE_ENV == 'production') {
	  axios.defaults.baseURL = 'https://xbbdev.com'
}
// 使用webpack等工具时 可以使用proxy代理 这里不进行设置也可
```

#### 

#### 设置请求头

在post请求时，我们需要加上一个请求头

```
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8'
```



#### 设置请求超时

```
axios.defaults.timeout = 10000;
```



#### 设置请求拦截

请求拦截是为了处理一些特殊事件，例如某些页面需要登陆后才能访问，提交的参数需要二次处理等，在发出请求时进行一个拦截

```
// 先获取需要的参数 例如localStorage、vuex
import store from '@/store/index'

// 设置请求拦截器
axios.interceptors.request.use(
	config => {
  	// 每次请求都会进行一定的处理
    const token = store.state.token
    // 将token设在请求头中
    token && (config.headers.Authorization = token)
    return config
  },
  error => {
  	return Promise.reject(error)
  }
)
```



#### 设置响应的拦截

```
axios.interceptors.response.use(
	response => {
  	// 根据返回状态码进行处理
    if (response.status === 200) {
   		return Promise.resolve(response)
    } else {
    	return Promise.reject(response)
    }
  },
  error => {
    // 任意非2xx的状态码
    // Do something
    $Message({
    	message: error.message,
      type: 'error'
    })
  	return Promise.reject(error);
  }
)
```



#### 封装get方法和post方法（可选）

get:

```
// get方法请求
// 传入请求地址和参数
export function get(url, params) {
	return new Promise((response, resolve) => {
  	axios.get(url, {
    	params
    }).then(res => {
    	resolve(res.data)
    }).catch(err => {
    	reject(err.data)
    })
  })
}
```



post:

```
export function post(url, params) {
	return new Promise((resolve, reject) => {
  	axios.post(url, QS.stringfy(params))
    .then(res => {
    	resolve(res.data)
    })
    .catch(err => {
    	reject(err.data)
    })
  })
}
```

get和post的区别在于，get的第二个参数是一个{ params: {} },

而post的第二个参数就是一个参数对象 {} 接下来就可以愉快的使用啦

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/0QvjnAaQy8dRnXo5/img/e3cb1c00-50ef-45f0-b71d-53b175e3be9b.gif)（ps: 第一个参数是url）



## 三、使用axios对api管理

速速使用一下封装的方法：

```
import { get, post } from './http'
// 先封装api
export const getOmg = (url, param) => post(url, param)

// use use
import { getOmg } from '@/request/api'
~~~~.vue环境~~~~
methods: {
	onLoad() {
  	getOmg('www.xxx.com', {
    	bussinessType: 1,
      userid: 1
    }).then(res => {
			// do something
		})
  }
}
```



到这里就基本上完结了，对于响应拦截器则还可以进行一些错误优化：

```

```