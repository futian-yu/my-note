**前端**

​	前端都有个前中后，可以不了解前端框架；但是有三个阶段一定要了解清楚，这样子简单的调试就基本ok；

​	调用后端前：处理数据，设置参数，传递至后端

​	调用后端的具体语句代码（查询）：一般封装好就一句代码；

​	调用后端并返回数据后：有特殊情况的时候可以处理返回后的值（显示界面）；

遇到的问题：

​	![1597334949731](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\1597334949731.png)



如图，多选框点击查询后，入账状态(accountingStatus)栏消失；

原因：多选框字段原本的字段格式被我们改变(数组：并且多个字段之间可能是以某种格式区分，可能是逗号，可能是空格，除非看到组件源码，否则只是猜)，改变是以逗号拼接后传至后端；此时后端不报错，dto字段类型为String；但是前端多选框该字段是数组类型，默认array[0] = '...',不报错,但是点击查询后界面显示为空，因为娶不到值;

解决方法：传值前在前端设置一个临时变量，存储多选框accountingStatus本身的面貌；传值时把拼接好的想要的格式传给后端，传值后设置 model.value.accountingStatus = 临时变量；相当于在后端处理完返回数据给前端后，accountingStatus 原本的样式又回来了；   这里注意，model的值可以有临时变量，dto不需要对应加。

代码：

```js
queryResource: function (e) {    if (viewModel.model.accountingStatus != null && !((typeof viewModel.model.accountingStatus=='string')&&viewModel.model.accountingStatus.constructor==String)) {        var accountingStatus = viewModel.model.accountingStatus.join(",");        var accountingStatusTemp = viewModel.model.accountingStatus;        viewModel.model.set("accountingStatus", accountingStatus);    }    $('#Grid').data('kendoGrid').dataSource.page(1);    viewModel.model.set("accountingStatus", accountingStatusTemp);},
```

