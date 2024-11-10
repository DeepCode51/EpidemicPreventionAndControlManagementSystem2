<template>
    <div class="main-class">
        <div class="timeNum">
            <p> &nbsp;疫情数据信息更新于
                <span style="font-weight: 500">{{ lastUpdateTime }}</span>
            </p>
        </div>
        <div class="btn-group">
            <el-tag class="tags-view-item" style="cursor: pointer;" size="small" @click="getHistory()">历史疫情数据</el-tag>
            <el-tag class="tags-view-item" style="cursor: pointer;" size="small" @click="getRiskArea()">风险地区查询</el-tag>
            <el-tag class="tags-view-item" style="cursor: pointer;" size="small" @click="getInfiniteNews()">疫情热点信息</el-tag>
            <el-tag class="tags-view-item" style="cursor: pointer;" size="small" @click="getVaccine()">实时疫苗接种</el-tag>
            <el-tag class="tags-view-item" style="cursor: pointer;" size="small" @click="getRumor()">网络信息辟谣</el-tag>
        </div>
        <!--疫情数据-->
        <el-row v-if="this.chinaDayList.length > 0" :gutter="20"
                style="margin-bottom: 11px;text-align: center;">
            <el-col :span="4">
                <div class="local" style="background-color: #fffaf7">
                    <div class="add"> 较上日
                        <span style="color: #e57631">+{{ this.chinaTotal.today.input }}</span>
                    </div>
                    <div class="number" style="color: #e57631;">{{ this.chinaTotal.total.input }}</div>
                    <div class="text">
                        <span>境外输入</span>
                    </div>
                </div>
            </el-col>
            <!-- 类似代码 -->
            <el-col :span="4">
                <div class="local" style="background-color: #fff8f8">
                    <div class="add"> 较上日
                        <span style="color: #e61c1d">+{{ this.chinaTotal.today.storeConfirm }}</span>
                    </div>
                    <div class="number" style="color: #e61c1d">{{ this.chinaDayList[this.chinaDayList.length - 1].total.storeConfirm }}</div>
                    <div class="text">
                        <span>现有确诊</span>
                    </div>
                </div>
            </el-col>
            <el-col :span="4">
                <div class="local" style="background-color: #fff4f4">
                    <div class="add"> 较上日
                        <span style="color: #be2121">+{{ this.chinaTotal.today.confirm }}</span>
                    </div>
                    <div class="number" style="color: #be2121">{{ this.chinaTotal.total.confirm }}</div>
                    <div class="text">
                        <span>累计确诊</span>
                    </div>
                </div>
            </el-col>
            <el-col :span="4">
                <div class="local" style="background-color: #fef7ff">
                    <div class="add"> 较上日
                        <span style="color: #ae3ac6">+{{ this.chinaTotal.extData.incrNoSymptom ? this.chinaTotal.extData.incrNoSymptom : "0" }}</span>
                    </div>
                    <div class="number" style="color: #ae3ac6">{{ this.chinaTotal.extData.noSymptom }}</div>
                    <div class="text">
                        <span>无症状感染者</span>
                    </div>
                </div>
            </el-col>
            <el-col :span="4">
                <div class="local" style="background-color: #f1f5ff">
                    <div class="add"> 较上日
                        <span style="color: #4e8be6">+{{ this.chinaTotal.today.heal }}</span>
                    </div>
                    <div class="number" style="color: #4e8be6">{{ this.chinaTotal.total.heal }}</div>
                    <div class="text">
                        <span>累计治愈</span>
                    </div>
                </div>
            </el-col>
            <el-col :span="4">
                <div class="local" style="background-color: #f3f6f8">
                    <div class="add"> 较上日
                        <span style="color: #4e5a65">+{{ this.chinaTotal.today.dead }}</span>
                    </div>
                    <div class="number" style="color: #4e5a65">{{ this.chinaTotal.total.dead }}</div>
                    <div class="text">
                        <span>累计死亡</span>
                    </div>
                </div>
            </el-col>
        </el-row>
        <!--地图 信息部分-->
        <el-row :gutter="20">
            <el-col :span="10">
                <el-card shadow="hover" :body-style="{padding: '0', height: '430px'}" v-show="news !== undefined">
                    <div style="text-align: center;margin-top: 15px;font-size: 18px;font-weight: 600">最新疫情新闻</div>
                    <el-timeline style="margin-top: 10px">
                        <el-timeline-item
                                v-for="(item, index) in news"
                                :key="index"
                                :timestamp="timestampToTime(item.eventTime)"
                        >
                            <a :href="item.eventUrl" target="_blank">{{ item.eventDescription }}</a>
                        </el-timeline-item>
                    </el-timeline>
                </el-card>
            </el-col>
            <el-col :span="14">
                <el-card shadow="hover" v-show="list.length > 0">
                    <ECharts v-if="list.length > 0" :list="list"/>
                </el-card>
            </el-col>
        </el-row>
        <el-dialog title="全国中高风险地区详情" :visible.sync="riskOpen" width="750px" append-to-body>
            <el-collapse v-model="activeName" accordion>
                <el-collapse-item :title="'高风险地区 ' + high.length + '个'" name="1">
                    <div v-for="item in high" class="text-item">{{ item }}</div>
                </el-collapse-item>
                <el-collapse-item :title="'中风险地区 ' + middle.length + '个'" name="2">
                    <div v-for="item in middle" class="text-item">{{ item }}</div>
                </el-collapse-item>
            </el-collapse>
            <div slot="footer" class="dialog-footer">
                <el-button @click="riskOpen = false">关 闭</el-button>
            </div>
        </el-dialog>
        <el-dialog title="历史疫情数据详情" :visible.sync="historyOpen" width="1111px" append-to-body>

            <ChartThree />

            <div slot="footer" class="dialog-footer">
                <el-button @click="historyOpen = false">关 闭</el-button>
            </div>
        </el-dialog>
        <el-dialog :title="title" :visible.sync="noticeOpen" width="780px" append-to-body>
            <div class="ql-container ql-snow" style="border: 0 !important;">
                <div class="ql-editor" v-html="content">
                </div>
            </div>
        </el-dialog>
        <el-dialog title="疫情热点信息查看" :visible.sync="newsOpen" width="780px" append-to-body>
            <InfiniteNews/>
        </el-dialog>
        <el-dialog title="全球新冠疫苗接种实时追踪" :visible.sync="vaccineOpen" width="780px" append-to-body>
            <Vaccine/>
        </el-dialog>
        <el-dialog title="网络信息辟谣第一线" :visible.sync="rumorOpen" width="700px" append-to-body>
            <Rumor/>
        </el-dialog>
    </div>
</template>

<script>
    import ECharts from '../../components/ECharts'
    import ChartThree from "../../components/ChartThree";
    import jsonp from 'jsonp'
    import 'quill/dist/quill.snow.css';
    import 'quill/dist/quill.core.css';
    import 'quill/dist/quill.bubble.css';
    import InfiniteNews from "../../components/InfiniteNews";
    import Vaccine from "../../components/Vaccine";
    import Rumor from "../../components/Rumor";


    export default {
        name: "Index",
        components: {Rumor, Vaccine, InfiniteNews, ECharts, ChartThree},
        data() {
            return {
                userImg: require('../../assets/logo.png'),
                lastUpdateTime: '',
                chinaTotal: {},
                chinaDayList: [],
                list: [],
                news: [],
                riskOpen: false,
                high: [],
                middle: [],
                activeName: '1',
                historyOpen: false,
                noticeOpen: false,
                content: undefined,
                title: undefined,
                newsOpen: false,
                vaccineOpen: false,
                rumorOpen: false
            }
        },
        methods: {
            getUserInfo() {
                this.$axios.get("/userInfo").then(res => {
                    this.$store.commit('SET_USER_INFO', res.data.data.user)
                })
            },
            getNews() {
                this.$axios.get("/news").then(res => {
                    this.news = res.data.data.slice(0, 7)
                })
            },
            timestampToTime(timestamp) {
                let date = new Date(timestamp * 1000);
                let Y = date.getFullYear() + '年';
                let M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '月';
                let D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + '日 ';
                let h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + '时';
                let m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + '分';
                return Y + M + D + h + m;
            },
            getRiskArea(){
                this.$axios.get("/riskarea").then(res => {
                    this.high = []
                    this.middle = []
                    let middle = res.data.data[1]
                    let high = res.data.data[2]
                    for(let key in middle){
                        if (middle.hasOwnProperty(key)){
                            middle[key].forEach(item => {
                                this.middle.push(item.province + " " + item.city + " " + item.addr)
                            })
                        }
                    }
                    for(let key in high){
                        if (high.hasOwnProperty(key)){
                            high[key].forEach(item => {
                                this.high.push(item.province + " " + item.city + " " + item.addr)
                            })
                        }
                    }
                    if (this.high.length === 0){
                        this.activeName = '2'
                    }
                    this.riskOpen = true
                })
            },
            getHistory(){
                this.historyOpen = true
            },
            getNotice(){
                this.$axios.get("/sys/notice").then(res => {
                    let noticeId = localStorage.getItem("noticeId")
                    if (noticeId !== res.data.data.noticeId && res.data.data !== "暂无公告"){
                        localStorage.setItem("noticeId", res.data.data.noticeId)
                        this.title = res.data.data.title
                        this.content = res.data.data.content
                        this.noticeOpen = true
                    }
                })
            },
            getInfiniteNews(){
                this.newsOpen = true
            },
            getVaccine(){
                this.vaccineOpen = true
            },
            getRumor(){
                this.rumorOpen = true
            },
            getData(){
                this.$axios.get("/chinaData").then(data => {
                    let res = data.data.data.data
                    this.list = res.areaTree[2].children.map(item => {
                        return {
                            name: item.name,
                            value: item.today.confirm
                        }
                    })
                    this.lastUpdateTime = res.lastUpdateTime
                    this.chinaTotal = res.chinaTotal
                    this.chinaDayList = res.chinaDayList
                })
            }
        },
        created() {
            this.getUserInfo()
            this.getNews()
            this.getData()
            // jsonp('https://view.inews.qq.com/g2/getOnsInfo?name=disease_h5', {}, (err, data) => {
            //     if (!err) {
            //         console.log(data.data)
            //         let res = JSON.parse(data.data)
            //         this.list = res.areaTree[0].children.map(item => {
            //             return {
            //                 name: item.name,
            //                 value: item.total.nowConfirm
            //             }
            //         })
            //         this.lastUpdateTime = res.lastUpdateTime
            //         this.chinaTotal = res.chinaTotal
            //         this.chinaAdd = res.chinaAdd
            //     }
            // })
            this.getNotice()
        }
    }
</script>

<style lang="scss">
    .el-collapse-item__header{
        font-size: 16px;
        color: rgb(190, 33, 33);
    }

    .text-item{
        font-size: 17px;
        padding: 10px;
        text-indent: 15px;
    }

    .el-timeline-item {
        padding-bottom: 17px;
    }

    .el-timeline {
        font-size: 15px;

        a {
            color: inherit;
            text-decoration: none;
            display: block;
            border: none;
            -webkit-tap-highlight-color: rgba(255, 255, 255, 0);
            -webkit-user-select: none;
            -moz-user-select: none;
        }

        a:hover {
            color: #409eff;
        }

        a:active, a:visited, a:link, a:focus {
            -webkit-tap-highlight-color: transparent;
            outline: none;
            background: none;
            text-decoration: none;
        }

        ::selection {
            background: #FFF;
            color: #333;
        }

        ::-moz-selection {
            background: #FFF;
            color: #333;
        }
    }

    .local {
        border-radius: 5px;
        padding: 18px;
    }

    .add {
        padding-top: 5px;
        font-size: 18px;
        line-height: 8px;
        font-weight: 400;
        color: #7c7c7c;
    }

    .number {
        font-size: 30px;
        line-height: 22px;
        font-weight: 700;
        padding-top: 22px;
    }

    .text {
        font-size: 18px;
        line-height: 13px;
        color: #222;
        font-weight: 500;
        margin-top: 18px;
    }

    .timeNum {
        color: #737373;
        font-size: 15px;
        padding: 0;
        margin-top: -12px;
        line-height: 8px;
        border-radius: 5px;
    }

    .btn-group{
        float: right;
        margin-top: -30px;
        .el-tag + .el-tag {
            margin-left: 6px;
        }
    }
</style>
