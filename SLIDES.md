# Node.JS 性能监控

---

# 内容

- 自我介绍
- JavaScript 性能分析原理
 - 插桩
 - 剖析
 - 内存
- Docker 在 Node.JS 项目中的运用
 - 前端打包
 - 搭建测试环境
- 监控系统开发
 - 采集
 - 数据库
 - 可视化
 - 报警
- 最近的项目
 - statsd-keystrokes
 - statsd-webpack-plugin
 - vis-bonjour-network
 - monitor-harddisk

---

# 关于我

--

 - ## .label.label-info[可视化]

--

 - ## .label.label-danger[APM]

--

 - ## .label.label-primary[全栈]

---
class: middle,center


# JavaScript 性能分析

[.fa.fa-github[] @wyvernnot](https://github.com/wyvernnot/javascript_performance_measurement)

---

## 怎么做

--

### Hook && Shimer && Monkey Patching

.well[
　在运行时动态地修改代码
]

--

示例

```
const http = require('http');
http.createServer( (req,res)=>{
  res.end('ok');
} )
```

---

## 正常启动

```sh
node server.js
```

--

## 加上监控模块

```sh
node -r oneapm server.js
```

---

### 发生了什么

```
node -h
```

```txt
Options:
  -v, --version         print Node.js version
  -e, --eval script     evaluate script
  -p, --print           evaluate script and print result
  -c, --check           syntax check script without executing
  -i, --interactive     always enter the REPL even if stdin
                        does not appear to be a terminal
* -r, --require         module to preload
```

---

## 前端埋点

<hr/>

<button>点击我买买买</button>
--

### 来一段 React 代码

```
class ProductItem extends React.Componet{

  render(){
    return <div>
      <img src="image-820yh402347y90.jpg"/>
      <button onClick={this.handleClick}>点击我买买买</button>
    </div>
  }

  handleClick(){
    // 加到购物车
  }
}
```



---

## ES7

### Decorator

```

*import {pageview,event} from 'blablabla-decorators';

class ProductItem extends React.Componet{

* @pageview
  render(){
    return <div>
      <img src="image-820yh402347y90.jpg"/>
      <button onClick={this.handleClick}>点击我买买买</button>
    </div>
  }

* @event('add_to_cart')
  handleClick(){
    // 加到购物车
  }
}
```

---

## 剖析

- ### 断层扫描

- ### 听诊

---

## 内存

- ### 分类
- ### GC

---
class: middle,center

# Docker 在项目中的运用
---

## 前端打包

- 通过　Webhook 来触发
- 运行在 Jenkins 上

---

## 搭建测试环境

### 参考 [12factor](http://12factor.net/zh_cn/)

- 快速

---

# 监控系统的开发

---

##　采集

---

### statsd

常见变量类型

--

- .label.label-default[gauge]

--

- .label.label-default[counter]

---

## 数据库

- 索引
- 标签
- 聚集
- 存储策略

---

## 可视化

- echarts
- highcharts
- d3

---

## 报警

- mailgun