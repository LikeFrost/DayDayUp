## 生成带 base64 二维码的分享海报

踩了两天的坑，终于爬出来了，记录一下。

由于要**自定义**分享海报上的内容，所以首先要有一个 canvas（行内设置宽高，在css内设置的话内部图片默认缩放）：

```html
<view class="bg">
    <canvas canvas-id="canvas" id="canvas" type="2d" style="height:366px;width:352px" class="cvs"></canvas>
</view>
<button class="ai-button" bind:tap="shareImg">分享</button>
```

js 中获取 canvas 节点：

```js
    const query = wx.createSelectorQuery()   //新写法
        query.select('#canvas')
        .fields({node:true,size:true})
        .exec((res)=>{
            const canvas = res[0].node;
            const dpr = wx.getSystemInfoSync().pixelRatio  //获取设备像素比
            canvas.width = res[0].width * dpr
            canvas.height = res[0].height * dpr
            const ctx = canvas.getContext('2d');
        })
```

绘制图片，在上述写法中要先定义图片节点才能绘制：

```js
let img = canvas.createImage();
img.src = "/images/Subtract.svg";
img.onload = () =>{
    ctx.drawImage(img,1,1,canvas.width,canvas.height);
}
```

添加文字：

```js
ctx.textBaseline = "top";
ctx.textAlign = "center"
ctx.font = `600 ${24*dpr}px 黑体`;
ctx.fillStyle="#333333";         //设置字体属性
ctx.fillText(this.data.labName,res[0].width * dpr/2,24*dpr);  //绘制文字
```

将后台获取到的 base64 转化为 canvas 能够绘制的临时文件：

```js
const buffer = wx.base64ToArrayBuffer(this.data.keyImg), //将 base64 转化为 buffer
filePath = `${wx.env.USER_DATA_PATH}/temp_image.png`;   //临时路径
wx.getFileSystemManager().writeFile({    //写文件
    filePath,
    data: buffer,
    encoding: 'base64',
    success:()=>{
        const code = canvas.createImage();
        code.src = filePath;
        code.onload=()=>{
            ctx.drawImage(code,25*dpr,268*dpr,88*dpr,88*dpr)
        }
    }
})
```

全部绘制完成后，点击按钮分享：

```js
    shareImg(){
        const query = wx.createSelectorQuery()    //选择节点
        const dpr = wx.getSystemInfoSync().pixelRatio
        query.select('#canvas')
        .fields({node:true,size:true})
        .exec((res)=>{
            const canvas = res[0].node;
            wx.pro.canvasToTempFilePath({      //生成临时文件
                x:0,
                y:0,
                width:352*dpr,
                height:366*dpr,
                canvas:canvas
            }).then((res)=>{
                this.setData({path:res.tempFilePath})
                wx.pro.showShareImageMenu({       //分享
                    path:this.data.path
                }).then((res)=>{
                    console.log(res)
                })
            })
        })
```



#### 整体代码：

wxml :

```html
<view class="title">生成分享图</view>
<view class="bg">
    <canvas canvas-id="canvas" id="canvas" type="2d" style="height:366px;width:352px" class="cvs"></canvas>
</view>
<button class="ai-button" bind:tap="shareImg">分享</button>
```

scss:

```css
.bg
{
    margin: 5rpx;
    padding-top: 20rpx;
    padding-bottom: 20rpx;

    background-color: rgba(242, 242, 242, 1);

    .cvs
    {
        display: block;

        margin: 20rpx auto;
    }
}

.title
{
    margin: var(--margin-24) var(--margin-16);

    font-size: var(--font-24);
    font-weight: 600;

    color: var(--gray-1);
}

.ai-button
{
    margin-top: 20rpx;
}

```

js:

```js
Page({
    data:{
        labId:53,
        key:'3H8IFT',
        labName:'实验室 N5-101',
        userName:'',
        telephone:'1111111111',
        duration:'0000年00月00日 00时00分',
        deadline:'0000年00月00日 00时00分',
        shareName:'1111111111',
        keyImg:'',
        path:'',
    },
    onLoad:function(option){
        this.setData({
            labId:option.labId,
            key:option.key,
            userName:option.userName,
            telephone:option.telephone,
            duration:option.duration,
            deadline:option.deadline,
            labName:option.labName,
            shareName:option.shareName
        })
        wx.pro.request({
            url:'xxx',
            method:'get',
            header:{
                'content-type':'application/x-www-form-urlencoded',
                'Authorization':wx.getStorageSync('token')
            },
            data:{
                'labId':this.data.labId
            }
        }).then((res)=>{
            this.setData({labName:res.data.data.lab.name})
        }).catch((e)=>{
            console.log(e)
        })
        const scene = `labId=${this.data.labId}&key=${this.data.key}`;
        wx.pro.request({
            url:"xxx",
            method:'post',
            header:{
                'content-type':'application/x-www-form-urlencoded',
                'Authorization':wx.getStorageSync('token')
            },
            data:{
                scene:scene,
                envVersion:'develop'
            }
        }).then((res)=>{
            this.setData({keyImg:res.data.data.generateQRCode})
            this.drawPic()
        }).catch((e)=>{
            console.log(e)
        })
    },
    drawPic(){
        const query = wx.createSelectorQuery()
        query.select('#canvas')
        .fields({node:true,size:true})
        .exec((res)=>{
            const canvas = res[0].node;
            const dpr = wx.getSystemInfoSync().pixelRatio
            canvas.width = res[0].width * dpr
            canvas.height = res[0].height * dpr
            const ctx = canvas.getContext('2d');
            let img = canvas.createImage();
            img.src = "/images/Subtract.svg";
            img.onload = () =>{
                ctx.drawImage(img,1,1,canvas.width,canvas.height);
                ctx.textBaseline = "top";
                ctx.textAlign = "center"
                ctx.font = `600 ${24*dpr}px 黑体`;
                ctx.fillStyle="#333333";
                ctx.fillText(this.data.labName,res[0].width * dpr/2,24*dpr);
                ctx.font = `400 ${14*dpr}px 黑体`;
                ctx.fillStyle="#666666";
                ctx.textAlign = "left"
                ctx.fillText(`管理员：${this.data.userName}`,74*dpr,78*dpr);
                ctx.fillText(`联系电话：${this.data.telephone}`,60*dpr,106*dpr);
                ctx.fillText(`密钥时限：${this.data.duration}`,60*dpr,134*dpr);
                ctx.fillText(`成员时限：${this.data.deadline}`,60*dpr,162*dpr);
                ctx.textAlign = "center"
                ctx.fillStyle="#C74242";
                ctx.fillText("请注意实验室相关事项，注意实验安全",res[0].width * dpr/2,210*dpr);
                ctx.fillText('实验室',307*dpr,319*dpr)
                const buffer = wx.base64ToArrayBuffer(this.data.keyImg),
                filePath = `${wx.env.USER_DATA_PATH}/temp_image.png`;
                wx.getFileSystemManager().writeFile({ 
                    filePath,
                    data: buffer,
                    encoding: 'base64',
                    success:()=>{
                        const code = canvas.createImage();
                        code.src = filePath;
                        code.onload=()=>{
                            ctx.drawImage(code,25*dpr,268*dpr,88*dpr,88*dpr)
                        }
                    }
                })
                ctx.textAlign = "left";
                ctx.fillStyle="#585858";
                ctx.fillText(`${this.data.shareName}已经加入实验室`,130*dpr,291*dpr)
                ctx.fillText('长按小程序码，立即加入',130*dpr,319*dpr)
            }
            ctx.restore();
        })
    },
    shareImg(){
        const query = wx.createSelectorQuery()
        const dpr = wx.getSystemInfoSync().pixelRatio
        query.select('#canvas')
        .fields({node:true,size:true})
        .exec((res)=>{
            const canvas = res[0].node;
            wx.pro.canvasToTempFilePath({
                x:0,
                y:0,
                width:352*dpr,
                height:366*dpr,
                canvas:canvas
            }).then((res)=>{
                this.setData({path:res.tempFilePath})
                wx.pro.showShareImageMenu({
                    path:this.data.path
                }).then((res)=>{
                    console.log(res)
                })
            })
        })
    }
})
```

效果图：

![效果图](/img/生成带base64二维码的分享海报.png)

