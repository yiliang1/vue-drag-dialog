<template>
  <div id="tlw-drag-dialog" v-show="visible" 
   :style="{height: height + 'px', width: width + 'px'}">
    <div class="title">
        <h2>{{title}}</h2>
        <div>
            <a v-show="showMin" class="min" href="javascript:;" title="最小化"></a>
            <a v-show="showMax"  class="max" href="javascript:;" title="最大化"></a>
            <a class="revert" href="javascript:;" title="还原"></a>
            <a v-show="showClose"  class="close" href="javascript:;" title="关闭"></a>
        </div>
    </div>
    <div class="resizeL"></div>
    <div class="resizeT"></div>
    <div class="resizeR"></div>
    <div class="resizeB"></div>
    <div class="resizeLT"></div>
    <div class="resizeTR"></div>
    <div class="resizeBR"></div>
    <div class="resizeLB"></div>
    <div class="content">
      <slot></slot>
    </div>    
  </div>
</template>

<script>
export default {
  name: 'tlwDragDialog',
  data() {
    return {
      showRevert: false,
    };
  },
  props: {
     /* 控制可见性 */
    visible: {
      type: Boolean,
      default: true
    },
     /* 标题 */
    title:  {
      type: String,
      default: '标题'
    },
     /* 宽度 */
    width: {
      type: Number,
      default: 300
    },
     /* 高度 */
    height: {
      type: Number,
      default: 160
    },
     /* 最小宽度 */
    dragMinWidth: {
      type: Number,
      default: 250
    },
    /* 最小高度 */
    dragMinHeight: {
      type: Number,
      default: 124
    },
     /* 是否展示最小化按钮 */
    showMin: {
      type: Boolean,
      default: true
    },
     /* 是否展示最大化按钮 */
    showMax: {
      type: Boolean,
      default: true
    },
     /* 是否展示关闭按钮 */
    showClose: {
      type: Boolean,
      default: true
    }
  },
  methods: {
    /*-------------------------- +
    获取id, class, tagName
    +-------------------------- */
    getById: function(id) {
      return typeof id === "string" ? document.getElementById(id) : id
    },
    getByClass: function(sClass, oParent) {
      var aClass = [];
      var reClass = new RegExp("(^| )" + sClass + "( |$)");
      var aElem = this.getByTagName("*", oParent);
      for (var i = 0; i < aElem.length; i++) reClass.test(aElem[i].className) && aClass.push(aElem[i]);
      return aClass
    },
    getByTagName: function(elem, obj) {
      return (obj || document).getElementsByTagName(elem)
    },
    /*-------------------------- +
    拖拽函数
    +-------------------------- */
    drag(oDrag, handle)
    {
      var disX = 0
      let disY = 0
      var oMin = this.getByClass("min", oDrag)[0];
      var oMax = this.getByClass("max", oDrag)[0];
      var oRevert = this.getByClass("revert", oDrag)[0];
      var oClose = this.getByClass("close", oDrag)[0];
      handle = handle || oDrag;
      handle.style.cursor = "move";
      handle.onmousedown = event=> {
        if (!event) {
          event =window.event
        }
        disX = event.clientX - oDrag.offsetLeft;
        disY = event.clientY - oDrag.offsetTop;
        
        document.onmousemove = event => {
          if (!event) {
            event =window.event
          }
          var iL = event.clientX - disX;
          var iT = event.clientY - disY;
          var maxL = document.documentElement.clientWidth - oDrag.offsetWidth;
          var maxT = document.documentElement.clientHeight - oDrag.offsetHeight;
          
          iL <= 0 && (iL = 0);
          iT <= 0 && (iT = 0);
          iL >= maxL && (iL = maxL);
          iT >= maxT && (iT = maxT);
          
          oDrag.style.left = iL + "px";
          oDrag.style.top = iT + "px";
          
          return false
        };
        
        document.onmouseup = function ()
        {
          document.onmousemove = null;
          document.onmouseup = null;
          this.releaseCapture && this.releaseCapture()
        };
        this.setCapture && this.setCapture();
        return false
      };	
      //最大化按钮
      oMax.onclick = () => {
        oDrag.style.top = oDrag.style.left = 0;
        oDrag.style.width = document.documentElement.clientWidth - 2 + "px";
        oDrag.style.height = document.documentElement.clientHeight - 2 + "px";
        oMax.style.display = "none";
        oRevert.style.display = "block";
      };
      //还原按钮
      oRevert.onclick = () => {		
        oDrag.style.width = this.dragMinWidth + "px";
        oDrag.style.height = this.dragMinHeight + "px";
        oDrag.style.left = (document.documentElement.clientWidth - oDrag.offsetWidth) / 2 + "px";
        oDrag.style.top = (document.documentElement.clientHeight - oDrag.offsetHeight) / 2 + "px";
        oRevert.style.display = "none";
        oMax.style.display = "block";
      };
      //最小化按钮
      oMin.onclick = () => {
        oDrag.style.display = "none";
        var oA = document.createElement("a");
        oA.className = "tlw-drag-dialog-open";
        oA.href = "javascript:;";
        oA.title = "还原";
        document.body.appendChild(oA);
        oA.onclick = function () {
          oDrag.style.display = "block";
          document.body.removeChild(oA);
          oA.onclick = null;
        };
      };
      //关闭按钮
      oClose.onclick = () => {
       this.$emit('update:visible', false)
      };
      //阻止冒泡
      oMin.onmousedown = oMax.onmousedown = oClose.onmousedown = function (event)
      {
        this.onfocus = function () {this.blur()};
        (event || window.event).cancelBubble = true	
      };
    },
    /*-------------------------- +
     改变大小函数
    +-------------------------- */
    resize(oParent, handle, isLeft, isTop, lockX, lockY){
      handle.onmousedown =  event => {
        if (!event) {
          event =window.event
        }
        var disX = event.clientX - handle.offsetLeft;
        var disY = event.clientY - handle.offsetTop;	
        var iParentTop = oParent.offsetTop;
        var iParentLeft = oParent.offsetLeft;
        var iParentWidth = oParent.offsetWidth;
        var iParentHeight = oParent.offsetHeight;
        const position = {
          top: oParent.offsetTop,
          left: oParent.offsetLeft,
          width: oParent.offsetWidth,
          height: oParent.offsetHeight
        }

        this.$emit('beforeResizeChange', position)

        document.onmousemove = event => {
          if (!event) {
            event =window.event
          }
          
          var iL = event.clientX - disX;
          var iT = event.clientY - disY;
          var maxW = document.documentElement.clientWidth - oParent.offsetLeft - 2;
          var maxH = document.documentElement.clientHeight - oParent.offsetTop - 2;			
          var iW = isLeft ? iParentWidth - iL : handle.offsetWidth + iL;
          var iH = isTop ? iParentHeight - iT : handle.offsetHeight + iT;
          
          isLeft && (oParent.style.left = iParentLeft + iL + "px");
          isTop && (oParent.style.top = iParentTop + iT + "px");
          
          iW < this.dragMinWidth && (iW = this.dragMinWidth);
          iW > maxW && (iW = maxW);
          lockX || (oParent.style.width = iW + "px");
          
          iH < this.dragMinHeight && (iH = this.dragMinHeight);
          iH > maxH && (iH = maxH);
          lockY || (oParent.style.height = iH + "px");
          
          if((isLeft && iW == this.dragMinWidth) || (isTop && iH == this.dragMinHeight)) document.onmousemove = null;
          
          return false;	
        };
        document.onmouseup =  () => {
          const position = {
            top: oParent.offsetTop,
            left: oParent.offsetLeft,
            width: oParent.offsetWidth,
            height: oParent.offsetHeight
          }
          this.$emit('afterResizeChange', position)
          document.onmousemove = null;
          document.onmouseup = null;
        };
        return false;
      }
    }
  },
  mounted() {
    var oDrag = document.getElementById("tlw-drag-dialog");
    var oTitle = this.getByClass("title", oDrag)[0];
    var oL = this.getByClass("resizeL", oDrag)[0];
    var oT = this.getByClass("resizeT", oDrag)[0];
    var oR = this.getByClass("resizeR", oDrag)[0];
    var oB = this.getByClass("resizeB", oDrag)[0];
    var oLT = this.getByClass("resizeLT", oDrag)[0];
    var oTR = this.getByClass("resizeTR", oDrag)[0];
    var oBR = this.getByClass("resizeBR", oDrag)[0];
    var oLB = this.getByClass("resizeLB", oDrag)[0];

    this.drag(oDrag, oTitle);
    //四角
    this.resize(oDrag, oLT, true, true, false, false);
    this.resize(oDrag, oTR, false, true, false, false);
    this.resize(oDrag, oBR, false, false, false, false);
    this.resize(oDrag, oLB, true, false, false, false);
    //四边
    this.resize(oDrag, oL, true, false, false, true);
    this.resize(oDrag, oT, false, true, true, false);
    this.resize(oDrag, oR, false, false, false, true);
    this.resize(oDrag, oB, false, false, true, false);
    oDrag.style.left = (document.documentElement.clientWidth - this.width) / 2 + "px";
    oDrag.style.top = (document.documentElement.clientHeight - this.height) / 2 + "px";
  }
}
</script>

<style scoped>
#tlw-drag-dialog{position: fixed;z-index: 1000;top:100px;left:100px;box-sizing:border-box; background:#e9e9e9;border:1px solid #444;border-radius:5px;box-shadow:0 1px 3px 2px #666;}
#tlw-drag-dialog .title{position:relative;height:27px;margin:5px;}
#tlw-drag-dialog .title h2{text-align: initial;font-size:14px;height:27px;line-height:24px;border-bottom:1px solid #A1B4B0;}
#tlw-drag-dialog .title div{position:absolute;height:19px;top:2px;right:0;}
#tlw-drag-dialog .title a,a.open{float:left;width:21px;height:19px;display:block;margin-left:5px;background:url(~@/assets/img/tool.png) no-repeat;}
a.open{position:absolute;top:10px;left:50%;margin-left:-10px;background-position:0 0;}
a.open:hover{background-position:0 -29px;}
#tlw-drag-dialog .title a.min{background-position:-29px 0;}
#tlw-drag-dialog .title a.min:hover{background-position:-29px -29px;}
#tlw-drag-dialog .title a.max{background-position:-60px 0;}
#tlw-drag-dialog .title a.max:hover{background-position:-60px -29px;}
#tlw-drag-dialog .title a.revert{background-position:-149px 0;display:none;}
#tlw-drag-dialog .title a.revert:hover{background-position:-149px -29px;}
#tlw-drag-dialog .title a.close{background-position:-89px 0;}
#tlw-drag-dialog .title a.close:hover{background-position:-89px -29px;}
#tlw-drag-dialog .content{overflow:auto;margin:0 5px;}
#tlw-drag-dialog .resizeBR{position:absolute;width:14px;height:14px;right:0;bottom:0;overflow:hidden;cursor:nw-resize;} /*background:url(img/resize.png) no-repeat;*/
#tlw-drag-dialog .resizeL,#tlw-drag-dialog .resizeT,#tlw-drag-dialog .resizeR,#tlw-drag-dialog .resizeB,#tlw-drag-dialog .resizeLT,#tlw-drag-dialog .resizeTR,#tlw-drag-dialog .resizeLB{position:absolute;background:#000;overflow:hidden;opacity:0;filter:alpha(opacity=0);}
#tlw-drag-dialog .resizeL,#tlw-drag-dialog .resizeR{top:0;width:5px;height:100%;cursor:w-resize;}
#tlw-drag-dialog .resizeR{right:0;}
#tlw-drag-dialog .resizeT,#tlw-drag-dialog .resizeB{width:100%;height:5px;cursor:n-resize;}
#tlw-drag-dialog .resizeT{top:0;}
#tlw-drag-dialog .resizeB{bottom:0;}
#tlw-drag-dialog .resizeLT,#tlw-drag-dialog .resizeTR,#tlw-drag-dialog .resizeLB{width:8px;height:8px;background:#FF0;}
#tlw-drag-dialog .resizeLT{top:0;left:0;cursor:nw-resize;}
#tlw-drag-dialog .resizeTR{top:0;right:0;cursor:ne-resize;}
</style>
