---
title: elementUI 2.0.11各项化表头
dropcap: true
date: 2018-11-15 21:17:30
tags:
categories:
---
```javascript
renderHeader(h,data){
            let column = data.column;
            return h(
                "el-popover",
                {
                    props: {
                        placement: "right",
                        trigger: "hover",
                        popperClass : "popperClassResOut"
                    }
                },
                [
                    h(
                        "div",
                        [`规则1：即当资源池的值达到设定的阈值，触发异常。`]
                    ),
                    h(
                        "div",
                        [`规则2：即当资源池的MAX（主机值）达到设定的阈值，触发异常。`]
                    ),
                    h(
                        "div",
                        [`规则1及规则2的三个值依次代表：高于上限、异常波动、低于下限的阈值`]
                    ),
                    h(
                        "span",
                        {
                            slot: "reference"
                        },
                        [
                            column.label,
                            h("i", {
                                class: "el-icon-question",
                                style: {
                                    marginLeft: "4px",
                                    cursor: "pointer",
                                    color: "#ea9518",
                                }
                            })
                        ]
                    )
                ]
            )
        },
        :render-header="item.keyName == 'thresholds'?renderHeader:null"
```