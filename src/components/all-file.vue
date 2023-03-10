<template>
    <div>

        <n-h2>全部特殊文件</n-h2>

        <n-space vertical>

            <!-- 搜索按钮 -->
            <n-input-group>
                <n-input v-model:value="searchText" :placeholder="`搜索文件（留空则显示全部）`" clearable></n-input>
                <n-select v-model:value="searchExt" :options="searchExtList" :placeholder="`文件类型（留空则显示全部）`"
                    clearable></n-select>
                <n-button @click="loadFileList" :loading="fileListLoading">{{ (searchText === undefined) ? '查看文件' : '点击搜索'
                }}</n-button>
            </n-input-group>

            <!-- 加载中动画 -->
            <div v-if="fileListLoading">
                <n-skeleton text :repeat="2" />
                <n-skeleton text :style="{ width: '60%' }" />
            </div>

            <!-- 文件列表 -->
            <n-list v-else hoverable>

                <n-list-item v-for="item in fileList">

                    <!-- 列表的主体内容 -->
                    {{ item.fulltext.replace('文件:', '') }}

                    <!-- 列表的前置图标 -->
                    <template #prefix>
                        <n-icon color="#70c0e8" v-if="extList['video'].includes((item.fulltext.split('.').reverse())[0])">
                            <material-symbol> video_file </material-symbol>
                        </n-icon>
                        <n-icon color="#63e2b7"
                            v-else-if="extList['audio'].includes((item.fulltext.split('.').reverse())[0])">
                            <material-symbol> audio_file </material-symbol>
                        </n-icon>
                        <n-icon color="#f2c97d"
                            v-else-if="extList['image'].includes((item.fulltext.split('.').reverse())[0])">
                            <material-symbol> plagiarism </material-symbol>
                        </n-icon>
                        <n-icon color="#e88080" v-else>
                            <material-symbol> draft </material-symbol>
                        </n-icon>
                    </template>

                    <!-- 列表的右侧按钮 -->
                    <template #suffix>
                        <menu-btn :input="item.fulltext"></menu-btn>
                    </template>

                </n-list-item>

            </n-list>

            <!-- 分页 -->
            <n-pagination v-if="totalPage" v-model:page="page" :page-count="totalPage" :disabled="fileListLoading" simple />

        </n-space>

    </div>
</template>

<script lang="ts" setup>
import { ref, watch, computed } from 'vue';
import type { Ref } from 'vue';
import sleep from 'await-sleep';

// 定义类型
interface fileListItem {
    fulltext: string
    fullurl: string
}

// 如果网页链接不是羊羊百科，自动进入测试模式
let isTesting = location.host === 'xyy.huijiwiki.com' ? false : true;

var searchText: Ref<null | string> = ref(null);
var searchExt: Ref<null | 'image' | 'audio' | 'video'> = ref(null);
var fileList: Ref<Array<fileListItem>> = ref([]);
var page: Ref<number | undefined> = ref(undefined);
var totalPage: Ref<number | undefined> = ref(undefined);
var fileListLoading = ref(false);
var extList = ref({
    audio: ['mp3', 'mid', 'wav'],
    video: ['mp4'],
    image: ['webp']
});
var searchExtList = ref([
    { label: '图片', value: 'image' },
    { label: '音频', value: 'audio' },
    { label: '视频', value: 'video' },
]);

// 处理检索
var query = computed(() => {
    let query = '';
    // 搜索内容
    (searchText.value === null) ? query += '' : query += `[[~*${searchText.value}*]]`;
    // 文件类型
    (searchExt.value === null) ? query += '' : query += `[[~*${extList.value[searchExt.value].join('||~*')}]]`;
    return query;
});

// 展示文件列表
async function loadFileList() {

    fileListLoading.value = true;
    totalPage.value = undefined;
    page.value = undefined;

    if (isTesting) {

        // 本地测试（开始）
        await sleep(1000);
        let response = {
            "batchcomplete": "",
            "query": {
                "pages": {
                    "47799": {
                        "pageid": 47799,
                        "ns": 14,
                        "title": "分类:Base64编码的文件",
                        "categoryinfo": {
                            "size": 246,
                            "pages": 0,
                            "files": 246,
                            "subcats": 0
                        }
                    }
                }
            }
        };
        let size = response['query']['pages']['47799']['categoryinfo']['size'];
        totalPage.value = Math.ceil(size / 20);
        page.value = 1;
        // 本地测试（结束）

    } else {

        let response = await fetch(encodeURI(`https://xyy.huijiwiki.com/api/rest_v1/transform/wikitext/to/html?${Date()}`), {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                body_only: true,
                wikitext: `{{#ask:[[分类:Base64编码的文件]]${query.value}|format=count}}`
            })
        });

        let responseText = await response.text();
        let size = (responseText.split(`id="mwAQ">`)[1]).split(`</p>`)[0];
        totalPage.value = Math.ceil(Number(size) / 20);
        page.value = 1;

    }
}

// 监听页数变化并加载
watch(page, async (page) => {

    // 如果page是undefined，说明正在刷新，无视他
    if (page === undefined) { return; }

    // 使状态变为“加载中”
    fileListLoading.value = true;

    // 强制等待0.5秒，欣赏加载动画，防止加载速度过快导致“闪屏”
    await sleep(500);

    let offset = (page - 1) * 20;
    console.log(offset);
    if (isTesting) {
        // 本地测试（开始）
        await sleep(1000);
        let response = {
            "query-continue-offset": 20,
            "query": {
                "printrequests": [
                    {
                        "label": "",
                        "key": "",
                        "redi": "",
                        "typeid": "_wpg",
                        "mode": 2
                    }
                ],
                "results": [
                    {
                        "文件:2016年02月03日 天地壹号广告 杯子舞篇.mp4": {
                            "printouts": [],
                            "fulltext": "文件:2016年02月03日 天地壹号广告 杯子舞篇.mp4",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:2016%E5%B9%B402%E6%9C%8803%E6%97%A5_%E5%A4%A9%E5%9C%B0%E5%A3%B9%E5%8F%B7%E5%B9%BF%E5%91%8A_%E6%9D%AF%E5%AD%90%E8%88%9E%E7%AF%87.mp4",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:2020 新年快乐.mp4": {
                            "printouts": [],
                            "fulltext": "文件:2020 新年快乐.mp4",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:2020_%E6%96%B0%E5%B9%B4%E5%BF%AB%E4%B9%90.mp4",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:2021年05月20日 京东玩具乐器超级品类日.mp4": {
                            "printouts": [],
                            "fulltext": "文件:2021年05月20日 京东玩具乐器超级品类日.mp4",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:2021%E5%B9%B405%E6%9C%8820%E6%97%A5_%E4%BA%AC%E4%B8%9C%E7%8E%A9%E5%85%B7%E4%B9%90%E5%99%A8%E8%B6%85%E7%BA%A7%E5%93%81%E7%B1%BB%E6%97%A5.mp4",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:2023年01月08日 灰太狼的PLUS超能力.mp4": {
                            "printouts": [],
                            "fulltext": "文件:2023年01月08日 灰太狼的PLUS超能力.mp4",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:2023%E5%B9%B401%E6%9C%8808%E6%97%A5_%E7%81%B0%E5%A4%AA%E7%8B%BC%E7%9A%84PLUS%E8%B6%85%E8%83%BD%E5%8A%9B.mp4",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 alert.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 alert.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_alert.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 bag.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 bag.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_bag.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 bus.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 bus.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_bus.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 button.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 button.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_button.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 camera.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 camera.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_camera.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 default.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 default.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_default.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 film.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 film.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_film.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 fly.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 fly.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_fly.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 message.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 message.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_message.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 remote.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 remote.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_remote.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》十五周年 xyy.mp3": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》十五周年 xyy.mp3",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%8D%81%E4%BA%94%E5%91%A8%E5%B9%B4_xyy.mp3",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:《喜羊羊与灰太狼》圣诞流沙杯垫.mp4": {
                            "printouts": [],
                            "fulltext": "文件:《喜羊羊与灰太狼》圣诞流沙杯垫.mp4",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E3%80%8A%E5%96%9C%E7%BE%8A%E7%BE%8A%E4%B8%8E%E7%81%B0%E5%A4%AA%E7%8B%BC%E3%80%8B%E5%9C%A3%E8%AF%9E%E6%B5%81%E6%B2%99%E6%9D%AF%E5%9E%AB.mp4",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:勇闯四季城 优酷宣传横幅1.webp": {
                            "printouts": [],
                            "fulltext": "文件:勇闯四季城 优酷宣传横幅1.webp",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E5%8B%87%E9%97%AF%E5%9B%9B%E5%AD%A3%E5%9F%8E_%E4%BC%98%E9%85%B7%E5%AE%A3%E4%BC%A0%E6%A8%AA%E5%B9%851.webp",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:勇闯四季城 优酷宣传横幅2.webp": {
                            "printouts": [],
                            "fulltext": "文件:勇闯四季城 优酷宣传横幅2.webp",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E5%8B%87%E9%97%AF%E5%9B%9B%E5%AD%A3%E5%9F%8E_%E4%BC%98%E9%85%B7%E5%AE%A3%E4%BC%A0%E6%A8%AA%E5%B9%852.webp",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:勇闯四季城 优酷宣传横幅3.webp": {
                            "printouts": [],
                            "fulltext": "文件:勇闯四季城 优酷宣传横幅3.webp",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E5%8B%87%E9%97%AF%E5%9B%9B%E5%AD%A3%E5%9F%8E_%E4%BC%98%E9%85%B7%E5%AE%A3%E4%BC%A0%E6%A8%AA%E5%B9%853.webp",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    },
                    {
                        "文件:勇闯四季城 优酷宣传横幅4.webp": {
                            "printouts": [],
                            "fulltext": "文件:勇闯四季城 优酷宣传横幅4.webp",
                            "fullurl": "https://xyy.huijiwiki.com/wiki/%E6%96%87%E4%BB%B6:%E5%8B%87%E9%97%AF%E5%9B%9B%E5%AD%A3%E5%9F%8E_%E4%BC%98%E9%85%B7%E5%AE%A3%E4%BC%A0%E6%A8%AA%E5%B9%854.webp",
                            "namespace": 6,
                            "exists": "1",
                            "displaytitle": ""
                        }
                    }
                ],
                "serializer": "SMW\\Serializers\\QueryResultSerializer",
                "version": 3,
                "meta": {
                    "hash": "d2c28332dddfc1d30d16a61648acf1e1",
                    "count": 20,
                    "offset": 0,
                    "source": "",
                    "time": "0.101089"
                }
            }
        };
        let a = response['query']['results'];
        let b: Array<fileListItem> = [];
        a.forEach((element: { [x: string]: any; }, index: string | number) => {
            b[Number(index)] = element[Object.keys(element)[0]];
        });
        console.log(b);
        fileList.value = b;
        fileListLoading.value = false;
        // 本地测试（结束）
    } else {

        let response = await fetch(`https://xyy.huijiwiki.com/api.php?action=ask&format=json&query=[[分类:Base64编码的文件]]${query.value}|limit=20|offset=${offset}&api_version=3&${Date()}`);
        let responseJSON = await response.json();

        let a = responseJSON['query']['results'];
        let b: Array<fileListItem> = [];
        a.forEach((element: { [x: string]: any; }, index: string | number) => {
            b[Number(index)] = element[Object.keys(element)[0]];
        });
        console.log(b);
        fileList.value = b;
        fileListLoading.value = false;
    }
})

</script>

<style>
#wiki-body li.n-list-item {
    margin-top: 0;
    margin-bottom: 0;
    line-height: inherit;
}

.mw-content-ltr ul,
.mw-content-rtl .mw-content-ltr ul {
    margin: inherit;
    padding: inherit;
}
</style>