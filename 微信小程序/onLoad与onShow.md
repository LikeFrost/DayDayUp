## onLoad 与 onShow

二者都是进入页面时执行，区别如下：

- **触发次数不同**：

  - onLoad 是在页面加载时触发，在页面的整个生命周期中只触发一次；
  - onShow 是在页面显示/切入前台时触发，在页面的生命周期中可触发多次，即每次切入页面时都会触发；

- **是否携带参数**：

  - onLoad 可以接收参数，使用方式：

    ```js
    //pageA:
    let id = '12345', name = 'xiaoming';
    wx.pro.navigateTo({url:`/pages/pageB?id=${id}&name=${name}`});
    
    //pageB:
    onLoad:function(options){
        console.log(options.id);    //'12345'
        console.log(options.name);  //'xiaoming'
    }
    ```

    使用场景：页面带参跳转；

  - onShow 不可以接收参数，通常用于二级页面对一级页面的内容做出修改并返回一级页面的情况；

    场景示例：pageA 为列表展示页面，在页面加载时触发 request 请求获取列表项；点击单个列表项跳转到 pageB 进行修改，修改完毕后上传到后台服务器并返回 pageA，此时 pageA 需要重新获取列表，故使用 onShow；



**那么，既要传参，又要跳转后刷新怎么办呢？**

场景示例：pageA 为列表展示页面，在页面加载时触发 request 请求获取列表项；点击单个列表项跳转到 pageB 进行修改，修改完毕后上传到后台服务器并返回 pageA，同时 pageB 回传 id。

首先是要刷新，所以一定要用 onShow，传参可以使用如下方式：

```js
let pages = getCurrentPages();    //获取页面栈
let prePage = pages[pages.length - 2];   //此页面的上一个页面
let list = prePage.data.listData;    //获取上一个页面的列表
list.name = newName;    //修改名称
prePage.upData({
    listData:list
})    //回传数据（传参）
wx.pro.navigateBack()  //回到上一页面
```

