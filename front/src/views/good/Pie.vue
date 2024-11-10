<template>
    <div id="myChart" :style="{width: '100%', height: '250px'}"></div>
</template>

<script>
    import echarts from 'echarts'

    const option = {
        title: {
            text: '物资库存组成',
            left: 'center'
        },
        tooltip: {
            trigger: 'item',
            formatter: '{a} <br/>{b} : {c} ({d}%)'
        },
        legend: {
            orient: 'vertical',
            left: 'left',
            data: ['直接访问', '邮件营销', '联盟广告', '视频广告', '搜索引擎']
        },
        series: [
            {
                name: '库存信息',
                type: 'pie',
                radius: '55%',
                center: ['50%', '60%'],
                zoom: 0.5,
                data: [
                    {value: 335, name: '直接访问'},
                    {value: 310, name: '邮件营销'},
                    {value: 234, name: '联盟广告'},
                    {value: 135, name: '视频广告'},
                    {value: 1548, name: '搜索引擎'}
                ],
                emphasis: {
                    itemStyle: {
                        shadowBlur: 10,
                        shadowOffsetX: 0,
                        shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                }
            }
        ]
    };

    export default {
        name: "Pie",
        props: {
            series: {
                type: Array
            },
            legend: {
                type: Array
            }
        },
        data () {
            return {
            }
        },
        mounted(){
            this.drawLine()
        },
        methods: {
            drawLine(){
                // 基于准备好的dom，初始化echarts实例
                let myChart = echarts.init(document.getElementById('myChart'))
                // 绘制图表
                option.series[0].data = this.series
                option.legend.data = this.legend
                myChart.setOption(option)
            }
        }
    }
</script>
