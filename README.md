# axios

## Project setup
```javascript
//if (!isAuth()) return
//对需要 token 的请求，请求前本地先进行判断
//如果本地没有 token 的，直接return，不在就行请求

import axios from 'axios'
import NProgress from 'nprogress' //引入nprogress进度条
import { message } from 'ant-design-vue'
import Router from '@/router/index'
import { BASE_URL, getToken, removeToken } from '../index'

import 'nprogress/nprogress.css' //这个样式必须引入
// 创建axios实例，添加基路径
const API = axios.create({
  baseURL: BASE_URL
})

let num = 0

// 添加请求拦截器
API.interceptors.request.use(config => {
  if (num === 0) {
    NProgress.start()
  }
  ++num
  //解构处请求地址
  const { url } = config
  //请求头添加token
  if (!url.startsWith('/sys/login')) {
    config.headers.Token = getToken()
  }
  //将处理后的 config 返回
  return config
})

// 添加响应拦截器
API.interceptors.response.use(
  res => {
    if (res.data.code === 600) {
      removeToken()
      message.error('登录过期，重新登录')
      Router.push('/login')
    } else if (res.data.code === 200) {
    } else {
      message.error(res.data.msg || '请求数据失败，未知原因，请联系后端人员')
    }

    --num
    if (num <= 0) {
      NProgress.done()
    }
    return res.data
  },
  error => {
    --num
    if (num <= 0) {
      NProgress.done()
    }
    message.error('发出的请求针对的是不存在的记录，服务器没有进行操作')
    return Promise.reject(error)
  }
)

export { API }
```
