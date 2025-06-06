<!--
	This file is part of ELCube.
	ELCube is free software: you can redistribute it and/or modify
	it under the terms of the GNU Affero General Public License as published by
	the Free Software Foundation, either version 3 of the License, or
	(at your option) any later version.
	ELCube is distributed in the hope that it will be useful,
	but WITHOUT ANY WARRANTY; without even the implied warranty of
	MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
	GNU Affero General Public License for more details.
	You should have received a copy of the GNU Affero General Public License
	along with ELCube.  If not, see <https://www.gnu.org/licenses/>.
-->
<template>
    <nk-query-layout
        ref="layout"
        title="脚本组件"
        :search-items-default="searchItemsDefault"
        :dataTableColumns="columns"
        :selectable="false"
        :init-rows="50"
        @change="search"
    >
        <a-button-group slot="action">
            <a-button type="primary" @click="create">新建</a-button>
            <a-button type="default" @click="ide">原生组件</a-button>
        </a-button-group>
    </nk-query-layout>
</template>

<script>
import NkUtil from "@/utils/NkUtil";

const formatVersion = ({cellValue})=>{
  return cellValue.substring(cellValue.length-9,cellValue.length);
}

export default {
    computed:{
        searchItemsDefault(){
            return [
                {
                    name:'类型',
                    field:'type',
                    component:'nk-search-options-single',
                    options:{
                        buckets: [
                            {label: "Card | 卡片",value:"Card"},
                            {label: "Field | 字段",value:"Field"},
                            {label: "Meter | 仪表盘",value:"Meter"},
                            {label: "Service | 服务",value:"Service"},
                        ]
                    }
                },
                {
                    name:'版本',
                    field:'version',
                    component:'nk-search-options-text',
                    placeholder:'请输入版本号'
                },
                {
                    name:'状态',
                    field:'state',
                    component:'nk-search-options-single',
                    options:{
                        buckets: [
                            {label: "Active | 激活的",value:"Active"},
                            {label: "InActive | 未激活",value:"InActive"},
                            {label: "History | 历史",value:"History"},
                        ]
                    }
                },
                {
                    name:'搜索',
                    field:'keyword',
                    component:'nk-search-options-text',
                    placeholder:'请输入关键字'
                },
            ]
        },
        columns(){
            return [
                { type: 'seq',          title: '#',      width: '4%', },
                { field: 'scriptType',  title: '类型',    width: '10%', sortable:true, params:{ orderField: 'SCRIPT_TYPE'}},
                { field: 'scriptName',  title: '组件',    width: '22%', sortable:true, params:{ orderField: 'SCRIPT_NAME'}},
                { field: 'scriptDesc',  title: '描述',    width: '25%', sortable:true, params:{ orderField: 'SCRIPT_DESC'}},
                { field: 'version',     title: '版本',    width: '10%', sortable:true, params:{ orderField: 'VERSION'},formatter: formatVersion},
                { field: 'state',       title: '状态',    width: '10%', sortable:true, params:{ orderField: 'STATE'}},
                { field: 'updatedTime', title: '更新时间', width: '10%', sortable:true, params:{ orderField: 'UPDATED_TIME'}, formatter: 'nkDatetimeFriendly' },
                {                       title: 'ACTION',
                    slots: { default: ({row},h) => {
                            return [h(
                                'router-link',
                                {
                                    props:{to: `/apps/def/component/detail/${row.scriptName}/${row.version}`}
                                },
                                "详情"
                            )]
                        }}
                },
            ];
        }
    },
    methods:{
        search(params){
            this.$http.post("/api/def/script/page",NkUtil.translateParamsToQueryString(params))
                .then((res)=>{
                    this.$emit("setTab","脚本组件");
                    if(this.$refs.layout)
                        this.$refs.layout.setData(res.data)
                });
        },
        create(){
            this.$router.push("/apps/def/component/create")
        },
        ide(){
            this.$router.push("/apps/def/component/ide/list")
        }
    }
}
</script>

<style lang="less">
</style>
