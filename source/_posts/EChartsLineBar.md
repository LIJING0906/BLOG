---
title: EChartsLineBar
dropcap: true
date: 2018-11-29 14:49:11
tags: eCharts
categories: eCharts
---
```javascript
option.tooltip.formatter: function(params) {
    let res = `${params[0].axisValue}：<br>`
    for (let i in params) {
        res += `${params[i].marker}${params[i].seriesName}：${params[i].value}个<br>`
    }
    return res;
}
```
![](./EChartsLineBar/tooltips0.png)