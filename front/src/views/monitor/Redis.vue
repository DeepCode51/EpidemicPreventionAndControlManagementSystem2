<template>
    <div class="main-class">
        <el-row :gutter="12">
            <el-col :span="24" v-if="info">
                <el-card shadow="hover">
                    <el-descriptions title="缓存基本信息" :column="4" border>
                        <el-descriptions-item label="Redis版本">{{ info.redis_version }}</el-descriptions-item>
                        <el-descriptions-item label="运行模式">{{ info.redis_mode === "standalone" ? "单机" : "集群" }}</el-descriptions-item>
                        <el-descriptions-item label="端口">{{ info.tcp_port }}</el-descriptions-item>
                        <el-descriptions-item label="客户端数">{{ info.connected_clients }}</el-descriptions-item>
                        <el-descriptions-item label="运行时间(天)">{{ info.uptime_in_days }}</el-descriptions-item>
                        <el-descriptions-item label="使用内存">{{ info.used_memory_human }}</el-descriptions-item>
                        <el-descriptions-item label="使用CPU">{{ info.used_cpu_user_children }}</el-descriptions-item>
                        <el-descriptions-item label="内存配置">{{ info.maxmemory_human }}</el-descriptions-item>
                        <el-descriptions-item label="AOF是否开启">
                            {{ info.aof_enabled === "0" ? "否" : "是" }}
                        </el-descriptions-item>
                        <el-descriptions-item label="RDB是否成功">{{ info.rdb_last_bgsave_status }}</el-descriptions-item>
                        <el-descriptions-item label="Key数量">{{ dbSize }}</el-descriptions-item>
                        <el-descriptions-item label="网络入口/出口">{{ info.instantaneous_input_kbps }}kps/{{info.instantaneous_output_kbps}}kps</el-descriptions-item>
                    </el-descriptions>
                </el-card>
            </el-col>
            <el-col :span="12" style="margin-top: 10px">
                <el-card shadow="hover">
                    <div ref="commandstats" style="height: 335px" />
                </el-card>
            </el-col>
            <el-col :span="12" style="margin-top: 10px">
                <el-card shadow="hover">
                    <div ref="usedmemory" style="height: 335px" />
                </el-card>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    import echarts from "echarts";

    export default {
        name: "Redis",
        data(){
            return{
                commandStats: undefined,
                info: undefined,
                dbSize: undefined,
                // 统计命令信息
                commandstats: null,
                // 使用内存
                usedmemory: null,
            }
        },
        methods: {
            getCache(){
                this.$axios.get("/monitor/redis").then(res => {
                    this.commandStats = res.data.data.commandStats
                    this.info = res.data.data.info
                    this.dbSize = res.data.data.dbSize

                    this.commandstats = echarts.init(this.$refs.commandstats, "macarons")
                    this.commandstats.setOption({
                        title: {
                            text: '缓存命令统计'
                        },
                        tooltip: {
                            trigger: "item",
                            formatter: "{a} <br/>{b} : {c} ({d}%)",
                        },
                        series: [
                            {
                                name: "命令",
                                type: "pie",
                                roseType: "radius",
                                radius: "70%",
                                center: ["50%", "63%"],
                                data: this.commandStats,
                                animationEasing: "cubicInOut",
                                animationDuration: 1000,
                            },
                        ],
                    })

                    this.usedmemory = echarts.init(this.$refs.usedmemory, "macarons")
                    this.usedmemory.setOption({
                        title: {
                            text: '缓存内存信息'
                        },
                        tooltip: {
                            formatter: "{b} <br/>{a} : " + this.info.used_memory_human,
                        },
                        series: [
                            {
                                name: "峰值",
                                type: "gauge",
                                min: 0,
                                max: 1000,
                                radius: "90%",
                                center: ["50%", "63%"],
                                detail: {
                                    formatter: this.info.used_memory_human,
                                },
                                data: [
                                    {
                                        value: parseFloat(this.info.used_memory_human),
                                        name: "内存消耗",
                                    },
                                ],
                            },
                        ],
                    })
                })
            }
        },
        mounted() {
            this.getCache()
        }
    }
</script>

<style lang="scss">
    .el-descriptions__body .el-descriptions__table .el-descriptions-item__cell{
        text-align: center;
    }
</style>
