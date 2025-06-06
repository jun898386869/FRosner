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
    <a-modal v-model="visible" :title="modal.title" :width="modal.width" centered :getContainer="getContainer">
        <div slot="footer">
            <a-button @click="visible=false">取消</a-button>
        </div>
        <a-form :label-col="{ span: 6 }" :wrapper-col="{ span: 10 }" @submit="formSubmit">
            <nk-search-box style="margin: 10px 10px 0;">
                <component v-for="(item,index) in modal.searchItems"
                           ref="searchItems"
                           :key="index"
                           :is="item.component"
                           :config="item"
                           :option="item.option || aggs[item.field]"
                           @changed="formItemChanged"
                ></component>
                <nk-search-item>
                    <a-button type="primary" html-type="submit"><a-icon type="search" /></a-button>
                    <slot name="buttons"></slot>
                </nk-search-item>
            </nk-search-box>
        </a-form>
        <vxe-table
            ref="grid"
            auto-resize
            size="mini"
            border=inner
            highlight-hover-row
            highlight-current-row
            header-cell-class-name="headerCellClassName"
            :sort-config="sortConfig"
            @sort-change="vxeSortChanged"
            :data="page.list">
            <vxe-table-column   title="#"        type="seq"               width="8%" ></vxe-table-column>
            <vxe-table-column v-for="column in modal.columns"
                              :key="column.field"
                              :title="column.title"
                              :field="column.field"
                              :width="column.width"
                              :header-align="column.align"
                              :align="column.align"
                              :sortable="column.sortable"
                              :formatter="column.formatter">
            </vxe-table-column>
            <vxe-table-column  title="ACTION" >
                <template v-slot="{ row }">
                    <a @click="selected(row)">选择</a>
                </template>
            </vxe-table-column>
        </vxe-table>
        <vxe-pager
            v-if="page.page"
            perfect
            size="mini"
            :current-page="page.page"
            :page-size="page.rows"
            :total="page.total"
            :layouts="['PrevPage', 'JumpNumber', 'NextPage', 'Total']"
            @page-change="vxePageChanged" />

    </a-modal>
</template>

<script>

export default {
    name: "NkModalSelector",
    props:{
        value:Boolean,
        getContainer:Function,
        modal:{
            type:Object,
            default(){
                return {
                    title:'选择',
                    width:"60%",
                    postCondition:null,
                    index:'document',
                    searchItems:[
                        {
                            name:'搜索',
                            field:'keyword',
                            component:'nk-search-options-text',
                            placeholder:'请输入关键字'
                        }],
                    columns:[{
                        title:'类型',
                        field:'docType',
                        width: '15%'
                    },{
                        title:'名称',
                        field:'docName',
                        width: '25%'
                    },{
                        title:'交易伙伴',
                        field:'partnerName',
                        width: '20%'
                    },{
                        field: 'docStateDesc',
                        title: '状态',
                        width: '20%'
                    }],
                    mappings:[],
                }
            }
        }
    },
    computed:{
        visible:{
            get(){
                return this.value;
            },
            set(value){
                this.$emit("input",value);
            }
        },
        aggs(){
            return this.page.aggs || {}
        },
        sortConfig(){
            return Object.assign({
                trigger: 'cell',
                remote: true,
                //defaultSort: {field: 'age', order: 'desc'},
                orders: ['desc', 'asc', null]
            },this.modal.sortConfig);
        }
    },
    data(){
        return {
            valueOld:undefined,
            page:{},
            params : {
                from : 0,
                rows : 10,
                orderField: '_score',
                order: 'desc'
            }
        }
    },
    created() {
    },
    beforeUpdate(){
        if(this.value && this.valueOld!==this.value){
            this.formSubmit();
        }
        this.valueOld = this.value;
    },
    methods:{
        query(){
            const aggs = this.modal.searchItems.filter(i=>i.agg).map(i=>i.field);
            const postCondition = this.modal.postCondition;
            const url = `/api/doc/list/${this.modal.index||'document'}`;

            this.$http.postJSON(url,Object.assign({aggs,postCondition},this.params))
                .then(response=>{
                    this.page = response.data;
                });
        },
        selected(row) {
            delete row._XID;
            this.visible = false;
            this.$emit("select",row);
        },
        formSubmit(e){
            if(e)e.preventDefault();
            if(this.$refs.grid)this.$refs.grid.clearSort();

            this.params.order = undefined;
            this.params.orderField = undefined;
            if(this.sortConfig && this.sortConfig.remote && this.sortConfig.defaultSort && this.sortConfig.defaultSort.field){
                const column = this.modal.columns.find(item=>item.field===this.sortConfig.defaultSort.field);
                if(column){
                    this.params.orderField = (column.params&&column.params.orderField)||column.field;
                    this.params.order = this.sortConfig.defaultSort.order;
                    if(this.$refs.grid)this.$refs.grid.sort(this.sortConfig.defaultSort.field, this.params.order);
                }
            }

            this.params.from = 0;
            this.query();
        },
        formItemChanged(e){
            if(e.field){
                this.params.conditions = this.params.conditions||{};
                if(e.condition){
                    this.params.conditions[e.field]=e.condition;
                }else{
                    delete this.params.conditions[e.field]
                }

                if(e.trigger){
                    this.params.from = 0;
                    this.query();
                }
            }
        },
        // 排序跳转
        vxeSortChanged({property,order}){
            if(this.sortConfig && this.sortConfig.remote){
                const column = this.modal.columns.find(item=>item.field===property)
                this.params.orderField = order===null?null:((column.params&&column.params.orderField)||property);
                this.params.order = order;
                this.query()
            }
        },
        // 页码跳转
        vxePageChanged(e){
            const from = (e.currentPage-1) * e.pageSize;
            const rows = e.pageSize;

            if(from + rows > 10000){
                this.$message.error("查询记录数不能超过10000条");
                this.$emit("error","查询记录数不能超过10000条");
                this.page.page = this.params.from / this.params.rows + 1;
            }else{
                this.params.from = from;
                this.params.rows = e.pageSize;

                this.query();
            }
        },
    }
}
</script>

<style scoped lang="less">
::v-deep .ant-modal-body{
    padding:0 0;
}
</style>
