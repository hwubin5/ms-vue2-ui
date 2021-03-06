<!--
Created by Bane.Shi.
Copyright MoenSun
User: Bane.Shi
Date: 2017/1/13
Time: 09:30-->
<template>
  <div class="ms-grid" :style="[gridStyleCompute]">
    <div class="ms-grid-box" :style="[boxHeightCompute]">
      <ms-grid-scope v-if="hasLeftCompute"
                     :grid-container="gridContainer"
                     :tree-structure="treeStructure"
                     position="left"
                     :data="dataData "
                     :origin-columns="columns"
                     :columns="leftColumnsData"
                     :bordered="bordered"
                     :max-column-level="maxColumnLevel"
                     :max-head-height="maxHeadHeight"
                     :max-head-offset-height="maxHeadOffsetHeight"
                     :height="heightCompute"
                     :width="leftWidthData"
                     :scrollWidth="scrollWidth"
                     :scrollX="scrollX"
                     :scrollY="scrollY"
                     :need-scroll-x-space="needScrollXSpaceData"
                     :ms-grid-id="msGridId"/>
      <ms-grid-scope ref="ms_grid_scope_center"
                     :grid-container="gridContainer"
                     :tree-structure="treeStructure"
                     :data="dataData "
                     :origin-columns="columns"
                     :columns="centerColumnsData"
                     :bordered="bordered"
                     :max-column-level="maxColumnLevel"
                     :max-head-height="maxHeadHeight"
                     :max-head-offset-height="maxHeadOffsetHeight"
                     :height="heightCompute"
                     :width="centerWidthCompute"
                     :left="centerLeft"
                     :has-left="hasLeftCompute"
                     :scrollWidth="scrollWidth"
                     :scrollX="scrollX"
                     :scrollY="scrollY"
                     :need-scroll-x-space="needScrollXSpaceData"
                     :ms-grid-id="msGridId" />
      <ms-grid-scope v-if="hasRightCompute"
                     :grid-container="gridContainer"
                     :tree-structure="treeStructure"
                     position="right"
                     :data="dataData "
                     :origin-columns="columns"
                     :columns="rightColumnsData"
                     :bordered="bordered"
                     :max-column-level="maxColumnLevel"
                     :max-head-height="maxHeadHeight"
                     :max-head-offset-height="maxHeadOffsetHeight"
                     :height="heightCompute"
                     :width="rightWidthData"
                     :scrollWidth="scrollWidth"
                     :scrollX="scrollX"
                     :scrollY="scrollY"
                     :need-scroll-x-space="needScrollXSpaceData"
                     :ms-grid-id="msGridId"/>

    </div>
  </div>
</template>
<script>
    import "ms-jquery-resize";
    import Vue from "vue";
    import MethodsMixin from "./mixins/MethodsMixin";

    import MsGridScope from "./grid-scope.vue";

    import bus from "./GridEvents";
    export default {
      name:'ms-grid',
      mixins:[MethodsMixin],
      props:{
        layout:{
          type:String
        },
        treeStructure:{
            type:Boolean,
            default:function() {
                return false;
            }
        },
        data:{
          type:Array,
          default:function(){
            return [];
          }
        },
        columns:{
          type:Array,
          default:function() {
            return [];
          }
        },
        width:{
          type:Number
        },
        height:{
          type:Number
        },
        scrollX:{
          type:Boolean,
          default(){
            return false;
          }
        },
        scrollY:{
          type:Boolean,
          default(){
            return false;
          }
        },
        columnMinWidth:{
          type:Number,
          default(){
            return 100;
          }
        },
        bordered:{
          type:Boolean,
          default(){
            return false;
          }
        }
      },
      data(){
        let me = this;
        return {
          msGridId:_.uniqueId("ms_grid_"),
          gridContainer:me.$parent,
          componentReady:false,
          maxColumnLevel:1,
          maxHeadHeight:0,
          maxHeadOffsetHeight:0,
          dataData:[],
          leftColumnsData:[],
          centerColumnsData:[],
          rightColumnsData:[],
          leftWidthData:0,
          centerWidthData:0,
          rightWidthData:0,
          centerLeft:0,
          hasLeftData:false,
          gridWidth:me.width,
          gridHeight:this.height,
          boxHeight:0,
          needScrollXSpaceData:false,
          scrollWidth:0
        }
      },
      computed:{
        gridStyleCompute:function(){
          let me = this;
          if(me.componentReady){
            let style = {};
             if(me.gridHeight){
              Object.assign(style,{height:me.gridHeight+'px'});
             }
            return style;
          }
        },
        dataCompute:function() {
          let me = this;
          if(me.treeStructure){
            return me.dataFormat(me.data);
          }
          return me.data;
        },
        heightCompute:function(){
          let me = this;
          if(me.scrollY){
            return me.gridHeight;
          }
        },
        boxHeightCompute:function(){
          let me = this;
          return {height:me.boxHeight+"px"};
        },
        centerWidthCompute:function(){
          let me = this;
          if(me.componentReady){
              return me.gridWidth-me.leftWidthData-me.rightWidthData;
          }
        },
        hasLeftCompute:function(){
          let me = this;
          if(me.leftColumnsData && me.leftColumnsData.length>0){
            return true;
          }
          return false;
        },
        hasRightCompute:function(){
          let me = this;
          if(me.rightColumnsData && me.rightColumnsData.length>0){
            return true;
          }
          return false;
        }
      },
      watch:{
        width:{
          handler:function(newVal,oldVal){
            let me = this;
            if(newVal!=undefined && newVal!=oldVal && me.layout!="fit"){
              me.gridWidth = newVal;
            }
          },
          immediate: true
        },
        height:{
          handler:function(newVal,oldVal){
            let me = this;
            if(newVal!=undefined && newVal!=oldVal && me.layout!="fit"){
              me.gridHeight = newVal;
            }
          },
          immediate: true
        }
      },
      beforeCreate(){
        let me = this;
        //me.columnsSplit();
      },
      created(){
          let me = this;
          me.dataData = _.cloneDeep(me.dataFormat(me.data));
          me.columnsSplit();
          bus.$on('ms-grid-max-level',function(level){
            if(me.maxColumnLevel<level){
              me.maxColumnLevel = level;
            }
          });

          bus.$on('ms-grid-head-table-height',function(gridId,height,offsetHeight){
            if(me.msGridId == gridId){
              height = Math.floor(height);
              if(me.maxHeadHeight < height){
                me.maxHeadHeight = height;
              }
              offsetHeight = Math.floor(offsetHeight);
              if(me.maxHeadOffsetHeight < offsetHeight){
                me.maxHeadOffsetHeight = offsetHeight;
              }
            }
          });
          bus.$on('ms-grid-scope-height',function(gridId,height){
            if(me.msGridId == gridId){
              if(me.boxHeight != height){
                me.boxHeight = height;
              }
            }
          });
          bus.$on('ms-grid-scroll-space-x',(gridId,result)=>{
              if(me.msGridId == gridId ){
                if(this.needScrollXSpaceData != result){
                  this.needScrollXSpaceData = result;
                }
              }
          });
          bus.$on('ms-grid-scope-width-compute',function(gridId,position,gridScopeWidth){
            if(me.msGridId == gridId){
              if(position == "left"){
                if(me.leftWidthData != gridScopeWidth){
                  me.leftWidthData = gridScopeWidth;
                }
                if(me.centerLeft != gridScopeWidth){
                  me.centerLeft = gridScopeWidth;
                }
              }
              if(position == "right"){
                if(me.rightWidthData != gridScopeWidth){
                  me.rightWidthData = gridScopeWidth;
                }
              }
            }
          });
          bus.$on('ms-children-expand-toggle',function(rowIndex){
            let record = me.dataData[rowIndex];
            record._expanded = !record._expanded;
          });
          bus.$on('show-children',function(gridId,rowIndex,show){
            if(me.dataData[rowIndex]['_show'] != undefined || me.dataData[rowIndex]['_show']!=show){
              Vue.set(me.dataData[rowIndex],'_show',show);
            }
          });
          bus.$on('ms-grid-scroll-width',function(id,width){
            me.scrollWidth = width;
          });
      },
      mounted(){
        let me = this;
        if(me.componentReady != undefined){
            me.componentReady = true;
        }
        me.$nextTick(me.initSize());
      },
      updated(){
        let me = this;
      },
      methods:{
        initSize(){
          let me = this;
          if(me.layout == "fit"){
            if(me.$el.parentElement){
               me.gridWidth = $(me.$el.parentElement).width();
               me.gridHeight = $(me.$el.parentElement).height();
               $(me.$el.parentElement).resize(function(){
                  me.gridWidth = $(me.$el.parentElement).width();
                  me.gridHeight = $(me.$el.parentElement).height();
               });
            }
          }
          if(!me.gridWidth){
            me.gridWidth = $(me.$el).width();
            $(me.$el).resize(()=>{
              me.gridWidth = $(me.$el).width();
            });
          }
        },
        columnsSplit:function(){
          let me = this;
          let leftColumns = [];
          let centerColumns = [];
          let rightColumns = [];
          _.forEach(_.cloneDeep(me.columns),function(column){
             if(column.lockable){
                if(column.lockPosition=="right"){
                  rightColumns.push(column);
                }else{
                  leftColumns.push(column);
                }
             }else {
                centerColumns.push(column);
             }
          });

          me.leftColumnsData = leftColumns;
          me.rightColumnsData = rightColumns;
          me.centerColumnsData = centerColumns;
          if(me.leftColumnsData && me.leftColumnsData.length>0){
            me.hasLeftData = true;
          }
        },
      },
      components: {
        MsGridScope
      }
    }
</script>
<style>

</style>
