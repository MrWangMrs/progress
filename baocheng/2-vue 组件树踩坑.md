# vue 组件树踩坑笔记 ==>  vue-treeselect.js

## vue组件树官网： https://vue-treeselect.js.org/

### 标准格式:
    export default {
      data: () => ({
        value: [],
        options: [ {
          id: 'fruits',
          label: 'Fruits',
          children: [ {
            id: 'apple',
            label: 'Apple 🍎',
            isNew: true,
          }, {
            id: 'grapes',
            label: 'Grapes 🍇',
          }, {
            id: 'pear',
            label: 'Pear 🍐',
          }, {
            id: 'strawberry',
            label: 'Strawberry 🍓',
          }, {
            id: 'watermelon',
            label: 'Watermelon 🍉',
          } ],
        }]
      }),
    }


### 注意事项：
    1. 数据 options 必须是数组， 每一项数据必须有id 和 label ,下一级数据必须是children,后台返回的id 名称不是 id的可以用正则转化 ，label,children同理


    2.options 必须在export default 的data 里面定义 ，axios 获取返回值后 赋值 this.options= 返回的转化后的数组

    示例
      axios(url).then(res=>{
          let treeDates=res.data;
          将参数转化为标准格式里面的参数
          treeDates = treeDates.replace(/weaponTypeId/g, 'id');
          treeDates = treeDates.replace(/weaponTypeName/g, 'label');
          treeDates = treeDates.replace(/weaponInfos/g, "children");
          转化后赋值
          this.options=treeDates;
      })

  
