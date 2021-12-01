## upData 与 setData

**一定要按需取用！**

两者都可以触发时更新 UI，类似于 React 的 useState，但也有比较坑人的地方：

- setData 是全部修改，直接覆盖：

        ```js
        data:{
            list:{
                name:'xiaoming',
                age:18,
            }
        }
        let newList = { name:'xiaohong' };
        this.setData({list:neList});
        console.log(this.data.list);   //{ name:'xiaohong' }
        ```

- upData 是只修改或增加，不覆盖：

```js
data:{
    list:{
        name:'xiaoming',
        age:18,
    }
}
let newList = { name:'xiaohong' };
this.upData({list:neList});
console.log(this.data.list);   //{ name:"xiaohong", age:18 }
```

所以在进行列表项删除或者新数据少于旧数据时，要使用 setData；

在部分修改时，使用 upData；

在进行列表项增加或者新数据多余旧数据时，二者均可。