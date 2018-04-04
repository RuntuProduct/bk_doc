# 数据方案实现

> 本页面使用了 markdown 的 flowchart 增强语法，建议在本地使用 vscode 的 Markdown Preview Enhanced 进行预览

## 首先需要知道的情报

> 每个微信小程序都可以有自己的本地缓存，可以通过 wx.setStorage（wx.setStorageSync）、wx.getStorage（wx.getStorageSync）、wx.clearStorage（wx.clearStorageSync）可以对本地缓存进行设置、获取和清理。同一个微信用户，同一个小程序 storage 上限为 **10MB**。localStorage 以用户维度隔离，同一台设备上，A 用户无法读取到 B 用户的数据。

> 注意： 如果用户储存空间不足，我们会清空最近最久未使用的小程序的本地缓存。我们不建议将关键信息全部存在 localStorage，以防储存空间不足或用户换设备的情况。

根据小程序在本地存储上的设计，我们假定本地存储中的数据是不可靠的，所以：

- 本地存储中的数据只作展示作用和缓存作用
- 本地存储不保存体量巨大的数据（流水列表），只保存账单列表、当前账单汇总等少量信息
- 本地存储实现简单的请求池功能（对于不需要实时知道请求结果的请求），请求缓存到本地，后台进行更新

## 数据存储流程

```flow
st=>start: 接收到数据存储请求
en=>end: 数据存储完成

con1=>condition: 判断网络请求是否需要走本地缓存流程
op1=>operation: 通过网络进行请求，并返回数据
op2=>operation: 根据请求附带的存储回调函数更新本地缓存
op3=>operation: 将请求体缓存到本地缓存中的请求池，&#10;请求池会定时执行请求池中的请求，&#10;更改后台状态

st->con1
con1(no, right)->op1->en
con1(yes)->op2->op3->en
```

## 数据读取流程

和数据存存储流程相似，都是在请求函数中增加一层拦截，判断是否先走本地缓存处理
