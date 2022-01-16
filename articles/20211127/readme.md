# 一次内存问题修复的流水账记录

先简单的介绍一下项目的情况，后台管理项目，layui + jQuery。接手的时候已经基本完成，主要做一些支持性、维护性的工作。

月初的时候，根据需求修改了导出 excel 功能，从：
```javascript
async function exportExcel() {
    const ids = await api.getRecordIDs(userIDs)
    const sheet = []
    for (let i = 0;i < ids.length; i++) {
        const data = await api.getRecord(ids[i])
        sheet.push(format(data))
    }
    excel.export(sheet)
}
```

改成了：
```javascript
async function exportExcel() {
    const records = await api.getRecords(userIDs)
    const sheet = []
    for (let i = 0;i < records.length; i++) {
        sheet.push(format(records[i]))
    }
    excel.export(sheet)
}
```

简单测试之后，部署到了线上。

一周后，用户报告导出功能无法使用，具体表现为：导出即将完成时页面崩溃，同时CPU、内存都被占满。

一开始，怀疑是用户的电脑硬件配置低、与导出插件冲突，就把插件从 1.6.2 升级到了 1.7.2 ，但是没有用，问题依然存在。于是在插件的演示地址尝试导出20万行、9列的模拟数据，同时观察系统资源使用情况。
![](./img/插件demo导出20w行9列数据的占用.png)
从上图看，占用并不高，排除了插件的嫌疑。

导出100条数据，并使用开发者工具录制性能分析数据：
![](./img/bug.100条.内存.png)
![bug.100条.性能](./img/bug.100条.性能.png)
![bug.100条.性能剖析](./img/bug.100条.性能剖析.png)
![bug.100条.占用](./img/bug.100条.占用.png)

确定嫌疑代码：

![旧循环](./img/旧循环.png)

把`layui.data`的调用移到最外面试试：
![新循环.1](./img/新循环.1.png)
![新循环.1.性能](./img/新循环.1.性能.png)
![新循环.1.占用](./img/新循环.1.占用.png)  
依然存在问题。

弃用`layui.data`改用原生`lcoalStorage`：
![弃用layui.data](./img/弃用layui.data.png)
![弃用layui.data.性能](./img/弃用layui.data.性能.png)
![弃用layui.data.占用](./img/弃用layui.data.占用.png)

看着好像没问题了，模拟个4000行的数据试试：
![4000条.占用](./img/4000条.占用.png)
![4000条.崩溃](./img/4000条.崩溃.png)

把`localStorage`的调用移动到页面里，同时在页面里复制一个专用的`getNameByID`方法，模拟导出4000条，占用：
![最终占用](./img/最终占用.png)
耗时：  
![最终耗时](./img/最终耗时.png)

一番测试下来，最终版本的极限在28000条，数量再大就会报 *STATUS_BREAKPOINT* 错误。
![28k极限](./img/28k极限.png)

一番折腾下来，深切感受到自己的鶸，对于边界情况的预计不够敏锐、完善。
