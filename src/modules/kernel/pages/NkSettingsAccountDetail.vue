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
    <nk-page-layout class="mini" title="账号详情">
        <nk-card title="基本信息">
            <nk-form :col="1" :edit="editModeBase" ref="baseForm">
                <nk-form-item term="账号" :validateFor="user.username" :required="true">
                    {{ user.username }}
                    <a-input size="small" v-model="user.username" slot="edit" style="width:180px"></a-input>
                </nk-form-item>
                <nk-form-item term="姓名" :validateFor="user.realname" :required="true">
                    {{ user.realname }}
                    <a-input size="small" v-model="user.realname" slot="edit" style="width:180px"></a-input>
                </nk-form-item>
                <nk-form-item term="有效期" :validateFor="user.validFrom" :required="true">
                    {{ user.validFrom }}-{{user.validTo}}
                    <a-range-picker size="small" v-model="userValid" slot="edit"></a-range-picker>
                </nk-form-item>
                <nk-form-item term="密码">
                    <a-input slot="edit" size="small" type="password" style="width: 300px" v-model="user.password"></a-input>
                </nk-form-item>
                <nk-form-item term="禁用">
                    {{ user.locked ? '是' : '否'}}
                    <a-switch slot="edit" size="small" v-model="locked"></a-switch>
                </nk-form-item>
                <nk-form-item term="注册时间">
                    {{ user.createdTime | nkDatetime}}
                </nk-form-item>
            </nk-form>
            <div style="text-align: right;margin-right: 40px;" slot="actions">
                <a-button v-if="!editModeBase" size="small" type="primary" @click="editModeBase=true">编辑</a-button>
                <a-button v-else               size="small" type="primary" @click="changeBase">确定</a-button>
                <a-button v-if="!editModeBase" size="small" type="default" @click="clearLoginLock" style="margin-left: 15px;">清除登陆锁定</a-button>
            </div>
        </nk-card>
        <nk-card title="授权">
            <vxe-table
                auto-resize
                size="mini"
                border=inner
                resizable
                highlight-hover-row
                class="me-perms-table"
                header-cell-class-name="headerCellClassName"
                :row-class-name="rowClassName"
                :data="user.authorities">
                <vxe-table-column   title="权限" field="authority"                width="15%"/>
                <vxe-table-column   title="数据限制" field="limitIds"                 width="10%">
                    <template v-slot="{ row }">
                        <span v-if="row.limits">
                            {{row.limits}}
                        </span>
                        <a v-else-if="row.limitIds&&row.limitIds.length" @click="showLimits(row)">
                            {{row.limitIds&&row.limitIds.length}}条限制
                        </a>
                    </template>
                </vxe-table-column>
                <vxe-table-column   title="操作限制" field="subPerm"           width="30%">
                    <template v-slot="{ row }">
                        <pre>{{row.subPerm&&JSON.stringify(row.subPerm,null,2)}}</pre>
                    </template>
                </vxe-table-column>
                <vxe-table-column   title="Level" field="level"                   width="10%">
                    <template v-slot="{ row }">
                        {{row.level.toString(16)}}
                    </template>
                </vxe-table-column>
                <vxe-table-column   title="授权来源">
                    <template v-slot="{ row }">
                        {{row.fromGroupDesc}}-{{row.fromPermissionDesc}}
                    </template>
                </vxe-table-column>
            </vxe-table>
        </nk-card>
        <nk-card title="Secrets">
            <vxe-table
                auto-resize
                size="mini"
                border=inner
                resizable
                highlight-hover-row
                class="me-perms-table"
                header-cell-class-name="headerCellClassName"
                :row-class-name="rowClassName"
                :data="user.secrets">
                <vxe-table-column   title="T"      field="type" width="10%"/>
                <vxe-table-column   title="Key"    field="code" width="30%" />
                <vxe-table-column   title="Secret" field="secret" width="20%" >
                    <template>
                        ********
                    </template>
                </vxe-table-column>
                <vxe-table-column   title="Desc"    field="name" width="15%" />
                <vxe-table-column   title="Updated" field="updatedTime" width="13%" formatter="nkDatetimeFriendly" />
                <vxe-table-column   >
                    <template v-slot="{row}">
                        <a-popconfirm title="删除密钥可能会影响第三方系统的正常运行，确认删除？" placement="topRight" @confirm="removeAppKey(row)">
                            <a-button type="link">删除</a-button>
                        </a-popconfirm>
                    </template>
                </vxe-table-column>
            </vxe-table>
            <a-popconfirm slot="extra" title="创建的密钥将用于访问本系统的系统，确认创建？" placement="topRight" @confirm="createAppKey">
                <a-button size="small" type="link" :loading="createAppKeyLoading">创建AppKey</a-button>
            </a-popconfirm>
        </nk-card>
    </nk-page-layout>
</template>

<script>
import moment from "moment";
import { v4 as uuidv4 } from 'uuid';
import { encode } from 'js-base64';
export default {
    name: "NkMe",
    data(){
        return {
            user: {},
            editModeBase:false,
            password:undefined,
            newPassword:undefined,
            confirmPassword:undefined,
            limits:undefined,
            createAppKeyLoading:false
        }
    },
    computed:{
        locked:{
            get(){
                return !!this.user.locked
            },
            set(value){
                this.$set(this.user,'locked',value ? 1 : 0);
            }
        },
        userValid:{
            get(){
                let from = this.user.validFrom === '00000000' ? '20200101': this.user.validFrom;
                let to   = this.user.validTo   === '00000000' ? '20200101': this.user.validTo;
                return [
                    from&&moment(from,"YYYYMMDD"),
                    to  &&moment(to  ,"YYYYMMDD"),
                ]
            },
            set(value){
                this.$set(this.user,'validFrom',value&&value[0]&&value[0].format('YYYYMMDD'));
                this.$set(this.user,'validTo',  value&&value[1]&&value[1].format('YYYYMMDD'));
            }
        }
    },
    created() {
        if(this.$route.params.mode==='detail'){
            this.$http.get('/api/settings/auth/accounts/detail?username='+this.$route.params.username)
                .then(res=>{
                    this.user = res.data;
                    this.$emit("setTab","账号详情:"+this.user.username);
                });
        }else{
            this.editModeBase = true;
            this.$emit("setTab","新建账号");
        }
    },
    methods:{
        clearLoginLock(){
            this.$http.postJSON("/api/settings/auth/accounts/clearLoginLock",this.user).then(()=>{
                this.$message.info("已清除")
            }).finally(()=>{
            })
        },
        changeBase(){
            if(!this.$refs.baseForm.hasError()){
                this.$http.postJSON("/api/settings/auth/accounts/update",this.user).then((res)=>{
                    if(this.$route.params.mode==='detail'){
                        this.user = res.data;
                        this.editModeBase = false;
                    }else{
                        this.$emit("replace",'/apps/settings/auth/accounts/detail/'+res.data.username);
                    }

                }).finally(()=>{
                })
            }
        },
        showLimits(row){
            this.$http.postJSON("/api/authentication/info/limits",row.limitIds)
                .then(res=>{
                    this.$set(row,'limits',res.data.map(x=>x.limitDesc).join(','));
                })
        },
        rowClassName({ row }){
            if(row.disabled){
                return "disabled"
            }
        },
        removeAppKey(row){
            this.$http.postJSON('/api/settings/auth/accounts/secret/remove',row)
                .then(()=>{
                    this.user.secrets.splice(this.user.secrets.indexOf(row),1);
                });
        },
        createAppKey(){

            this.createAppKeyLoading = true;

            const key    = uuidv4().replaceAll('-','').substring(0, 16).toUpperCase();
            const secret = encode(uuidv4().replaceAll('-','').toUpperCase());

            this.$http.postJSON('/api/settings/auth/accounts/secret',{code: key,secret,accountId:this.user.id,name:'AppKey'})
                .then(()=>{
                    const h = this.$createElement;
                    this.$info({
                        title: '密钥已创建，请记录',
                        width: 580,
                        centered:true,
                        content: h('nk-form', {style:{'margin-top':'30px'},props:{col:1}}, [
                            h('nk-form-item', {props:{title:'KEY',width:80}}, key),
                            h('nk-form-item', {props:{title:'SECRET',width:80}}, secret),
                        ]),
                        onOk() {
                        },
                    });
                    this.createAppKeyLoading = false;
                    this.$http.get('/api/settings/auth/accounts/detail?username='+this.$route.params.username)
                        .then(res=>{
                            this.user = res.data;
                        });
                });
        }
    }
}
</script>

<style scoped>
    ::v-deep.me-perms-table tr.disabled {
        background-color: #eee;
        text-decoration:line-through;
    }
</style>
