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
    <div class="border"
         @dragstart="$emit('dragstart',$event)"
         @drag="$emit('drag',$event)"
         @dragover="$emit('dragover',$event)"
         @dragend="$emit('dragend',$event)"
         @click="$emit('click',$event)"
         :draggable="draggable">
        <div class="nk-form-divider">
            <label v-if="term||title" class="l">
                <b>{{term||title}}</b>
            </label>
            <slot></slot>
        </div>
    </div>
</template>

<script>
export default {
    name: "NkFormDivider",
    props: {
        draggable:String,
        options:{},
        term: {
            type: String,
            required: false
        },
        title: {
            type: String,
            required: false
        },
    }
}
</script>

<style scoped lang="less">
div.border{
    width: 100%;
}
div.nk-form-divider{
    position:relative;height:1px;background-color: rgba(0, 0, 0, 0.2);margin:20px 30px 20px 30px;

    label.l{
        position: absolute;top:-10px;left: 80px;background-color: #fff;padding: 0 10px;font-size: 12px;
    }
}

</style>
