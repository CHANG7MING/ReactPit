# Echarts 图表

## 柱状图


x轴上文字倾斜
````js
  xAxis: {
                type: 'category',
                data: xData,
                axisLabel: {
                    rotate:30, //旋转角度
                    interval:0,
                    margin:4,
                    formatter: (value) => {
                            if (value.length > 13) value= value.slice(0, 13) + "..."; //截取文字显示
                            var ret = '\n\n' //拼接加 返回的类目项
                            var maxLength = 8 //每项显示文字个数
                            var valLength = value.length //X轴类目项的文字个数
                            var rowN = Math.ceil(valLength / maxLength) //类目项需要换行的行数
                            if (rowN > 1) {
                              //如果类目项的文字大于10,
                            for (var i = 0; i < rowN; i++) {
                                var temp = '' //每次截取的字符串
                                var start = i * maxLength //开始截取的位置
                                var end = start + maxLength //结束截取的位置
                                temp = value.substring(start, end) + '\n'
                                ret += temp //凭借最终的字符串
                            }
                                return ret
                            } else {
                                return value
                            }
                        },
                },
            },
````

## 饼图

柱状图，柱上的文字
````js
  series: [{
                        data: valData,
                        // type: 'line', //折线图
                        type: 'bar', // 柱状图
                        barWidth: '20',              //---柱形宽度
                        barCategoryGap: '20%',       //---柱形间距
                        label: {
                            show: true, //开启显示数值
                            position: 'top', //数值在上方显示
                            textStyle: {  //数值样式
                              fontSize: 14  //字体大小
                            }},
                    }]
````

![](http://rt3imiukk.hb-bkt.clouddn.com/%E6%88%AA%E5%B1%8F2023-04-23%2015.55.08.png)



饼图百分比设置

饼图百分比

````js
series: [
                        {
                            data: valData,
                            type: "pie", //折线图
                            radius: ["0%", "60%"],
                            label: {
                                normal: {
                                    show: true,
                                    formatter: '{b}({d}%)' //自定义显示格式(b:name, c:value, d:百分比)
                                }
                            },
                        },
                    ],
````

![](http://rt3imiukk.hb-bkt.clouddn.com/%E6%88%AA%E5%B1%8F2023-04-23%2015.55.53.png)

百分比在内侧

````js
series: [{
   name: '产量',
   type: 'pie',
   center: ['22%', '50%'],
   radius: ['0', '82%'],
   avoidLabelOverlap: false,
   label: {
       //echarts饼图内部显示百分比设置
       show: true,
       position: "inside", //outside 外部显示  inside 内部显示
       formatter: `{d}%`,
       color: "#ffffff", //颜色
       fontSize: 12 //字体大小
   },
   data: this.rows
}]
````

![](http://rt3imiukk.hb-bkt.clouddn.com/%E6%88%AA%E5%B1%8F2023-04-23%2016.06.58.png)