## 生成带 base64 二维码的分享海报 Pro

小程序，伤害人你可真有一套！

文档老骗子了，昨天写的海报分享在模拟器上跑起来完全ok，结果真机和体验版都炸了，用回了老接口完美解决，记录一下代码：

- wxml

```html
<view class="list-item" bind:tap="getPic">
    <view class="name">分享实验室密钥</view>
    <view class="value">生成分享图</view>
</view>
<view class="result_paper" wx:if="{{isShow}}" catchtouchmove="moveHandle" bindtap="hidePaper" bind:longtap="shareImg">
    <view class="paper_content" wx:if="{{tempFilePath}}">
        <image src="{{tempFilePath}}" style="height:366px; width:352px;position: relative;z-index: 999;"id="canvas_image" catchtap="hidePaper"/>
    </view>
    <view style="width:0px;height:0px;overflow: hidden;">
        <canvas canvas-id="canvas" class="paper_canvas" id="canvas"></canvas>
    </view>
</view>
```

- scss

```css
.title
{
    margin: var(--margin-24) var(--margin-16);

    font-size: var(--font-24);
    font-weight: 600;

    color: var(--gray-1);
}

.list
{
    margin-top: var(--margin-12);

    .list-item
    {
        display: flex;

        margin: var(--margin-12) var(--margin-16);
        border-radius: var(--radius-8);
        height: 80rpx;

        background-color: var(--blue-background);

        align-items: center;

        .name
        {
            margin-left: var(--margin-16);

            font-size: var(--font-14);
            line-height: var(--font-14);

            color: var(--gray-2);
        }

        .value
        {
            margin-right: var(--margin-16);

            font-size: var(--font-12);
            line-height: var(--font-12);
            text-align: right;

            color: var(--gray-3);

            flex: 1;
        }
    }

    .qrcode
    {
        text-align: center;

        .img
        {
            width: 500rpx;
            height: 500rpx;
        }
    }
}

.ai-button
{
    margin-top: var(--margin-24);

    background-color: var(--red);
}

.result_paper
{
    position: fixed;
    top: 0;
    left: 0;
    z-index: 999999;

    width: 100vw;
    min-height: 100%;

    color: #fff;
    background: rgba(0, 0, 0, .6);

    .paper_content
    {
        overflow: auto;

        position: relative;
        z-index: 98;

        width: 100vw;
        height: 100vh;

        text-align: center;

        #canvas_image
        {
            margin-top: 24%;
        }
    }
}

.paper_canvas
{
    overflow: hidden;

    position: absolute;
    bottom: -5000rpx;
    left: 0;

    width: 704rpx;
    height: 732rpx;

    text-align: center;
}
```

- js

```js
Page({
    data:{
        labId:'',
        key:'',
        keyList:[],
        tempFilePath: '',
		isShow: false,
        keyImage:'',
        labName:'',
        shareName:'',
        background:'/images/Subtract.png'
    },
    onLoad:function(option){
        this.setData({labId:option.id,key:option.key})
    },
    onShow:function(){
        
    },
    preventTouchMove() {},
    getPic(){
        this.setData({isShow:true})
        if(this.data.tempFilePath){
            this.shareImg();
            return; 
        }else{
            wx.pro.showLoading({title:'加载中'});
        }

        wx.pro.request({
            url:' ',
            method:'get',
            header:{
                'content-type':'application/x-www-form-urlencoded',
                'Authorization':wx.getStorageSync('token')
            },
            data:{
                'labId':this.data.labId
            }
        }).then((res)=>{
            //console.log('获取实验室名称',res.data.data.lab.name)
            this.setData({labName:res.data.data.lab.name})
            return wx.pro.request({
                url:" ",
                method:'get',
                header:{
                  'content-type':'application/x-www-form-urlencoded',
                  'Authorization':wx.getStorageSync('token')
                },
            })
        }).then((res)=>{
            //console.log('获取分享者姓名',res.data.data.userInfo.name)
            this.setData({shareName:res.data.data.userInfo.name})
            const scene = `labId=${this.data.labId}&key=${this.data.key}`;
            const info = wx.pro.getAccountInfoSync();//版本类型
            return wx.pro.request({
                url:" ",
                method:'post',
                header:{
                    'content-type':'application/x-www-form-urlencoded',
                    'Authorization':wx.getStorageSync('token')
                },
                data:{
                    scene:scene,
                    envVersion:info.miniProgram.envVersion
                }
            })
        }).then((res)=>{
            //console.log('获取qrcode',res)
            this.setData({keyImg:res.data.data.generateQRCode})
            const buffer = wx.base64ToArrayBuffer(this.data.keyImg),
            filePath = `${wx.env.USER_DATA_PATH}/temp_image.png`;
            return wx.getFileSystemManager().writeFile({ 
                filePath,
                data: buffer,
                encoding: 'base64',
                success:()=>{
                    this.setData({keyImage:filePath});
                    this.drawPic();
                },
            })
        }).catch(() => {
            this.setData({isShow: false});
            wx.hideLoading()
            wx.showToast({
                title: '加载失败',
                duration: 2000,
                icon: 'none'
            })
        })
    },
    drawPic(){
        const ctx = wx.createCanvasContext('canvas');
        const dpr = wx.getSystemInfoSync.pixelRatio;
        ctx.fillStyle="#F2F2F2";
        //ctx.scale(dpr,dpr)
        ctx.fillRect(0,0,352,366)
        ctx.drawImage(this.data.background,1,1,350,364);
        ctx.drawImage(this.data.keyImage,25,268,88,88);
        ctx.setTextBaseline("top");
        ctx.setTextAlign("center")
        ctx.setFontSize(24)
        ctx.setFillStyle("#333333");
        ctx.fillText(this.data.labName,352/2,12);
        ctx.setFontSize(14)
        ctx.setFillStyle("#666666");
        ctx.setTextAlign("left")
        ctx.fillText(`管理员：${this.data.keyList.userName}`,74,78);
        ctx.fillText(`联系电话：${this.data.keyList.telephone}`,60,106);
        ctx.fillText(`密钥时限：${this.data.keyList.duration}`,60,134);
        ctx.fillText(`成员时限：${this.data.keyList.deadline}`,60,162);
        ctx.setTextAlign("center")
        ctx.setFillStyle("#C74242");
        ctx.fillText("请注意实验室相关事项，注意实验安全",352/2,210);
        ctx.fillText('实验室',307,319);
        ctx.setTextAlign("left");
        ctx.setFillStyle("#585858");
        ctx.fillText(`${this.data.shareName}已经加入实验室`,130,291)
        ctx.fillText('长按小程序码，立即加入',130,319);
        ctx.draw(false,()=>{
            setTimeout(()=>{
                wx.canvasToTempFilePath({
                    x: 0,
                    y: 0,
                    width: 352,
                    height: 366,
                    destWidth: 352*dpr,
                    destHeight: 366*dpr,
                    canvasId: 'canvas',
                    quality: 1,
                    success: (res) => {
                        console.log('success',res)
                        var tempFilePath = res.tempFilePath;
                        this.setData({
                            show:false,
                            tempFilePath: tempFilePath
                        })
                        wx.hideLoading();
                        this.shareImg();
                    },
                    fail: res => {
                        console.log('fail',res)
                        wx.hideLoading()
                        wx.showToast({
                            title: '海报加载失败',
                            duration: 2000,
                            icon: 'none'
                        })
                    }
                })
            },100)
        })
    },
    shareImg(){
        wx.pro.showShareImageMenu({
            path:this.data.tempFilePath
        }).then((res)=>{
            console.log(res)
        }).catch((e)=>{
            console.log(e)
        })
    },
    hidePaper: function () {
        this.setData({isShow: false});
        wx.hideLoading()
    },
})
```

