<template>
    <div id="myChart2" :style="{width: '100%', height: '250px'}"></div>
</template>

<script>
    import echarts from 'echarts'
    const option = {
        title: {
            text: '物资库存数量',
            left: 'center'
        },
        color: ['#3398DB'],
        tooltip: {
            trigger: 'axis',
            axisPointer: {            // 坐标轴指示器，坐标轴触发有效
                type: 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
            }
        },
        grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
        },
        xAxis: [
            {
                type: 'category',
                data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
                axisTick: {
                    alignWithLabel: true
                }
            }
        ],
        yAxis: [
            {
                type: 'value'
            }
        ],
        series: [
            {
                name: '库存余量',
                type: 'bar',
                barWidth: '60%',
                data: [10, 52, 200, 334, 390, 330, 220]
            }
        ]
    };
    export default {
        name: "Bar",
        props: {
            legend: {
                type: Array
            },
            total: {
                type: Array
            }
        },
        mounted(){
            this.drawLine()
        },
        methods: {
            drawLine(){
                // 基于准备好的dom，初始化echarts实例
                let myChart = echarts.init(document.getElementById('myChart2'))
                // 绘制图表
                option.series[0].data = this.total
                option.xAxis[0].data = this.legend
                myChart.setOption(option)
            }
        }
    }
</script>

<style scoped>

</style>
