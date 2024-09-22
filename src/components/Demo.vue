<template>
  <div class="demo-box">
      <div class="header">物料板ApiDemo</div>
      <div class="main">
        <div class="left">
          <div class="tools">
            <div class="tools-item" :class="{active:activeIndex===0}" @click="activeIndex=0" >接口调用</div>
            <div class="tools-item" :class="{active:activeIndex===1}" @click="activeIndex=1" >拖拽演示</div>
          </div>
          <div class="board-content">
            <!-- 接口方法 -->
            <div v-if="activeIndex===0" class="api-box">
              <button @click="getData()">获取物料板数据</button>
            </div>
            <!-- 拖拽演示 -->
            <div v-if="activeIndex===1" class="api-drag-box">
              <img v-for="(item,index) in imgArr"
                   draggable="true"
                   @dragstart="onDrag"
                   @dragend="onDragEnd"
                   :src="item.src"
                   :k="index"
              />
            </div>

          </div>
        </div>
        <div class="right">
          <!-- src="http://localhost:8081/" -->
          <iframe id="boardFrame"
                  height="100%"
                  width="100%"
                  frameborder="0"
                  scrolling="no"
                  src="../../static/board/index.html"></iframe>
          <div v-if="overLayerShow" @drop="drop" @dragover.prevent class="overLayer"></div>
        </div>
      </div>
  </div>
</template>

<script>
export default {
  name: 'Demo',
  data () {
    return {
      activeIndex:0,
      imgArr:[{src:'http://pic.soutu123.com/element_origin_min_pic/16/12/17/52eb47ea119560b6ba6822f2f78ffbcc.jpg!/fw/700/quality/90/unsharp/true/compress/true'}
        ,{src:'http://img2.jc001.cn/img/488/1501488/1208/125024881294a30.jpg'}
        ,{src:'http://img2.jc001.cn/img/488/1501488/1208/125024acfad88b1.jpg'},
        {src:'http://pic.soutu123.com/element_origin_min_pic/16/12/17/52eb47ea119560b6ba6822f2f78ffbcc.jpg!/fw/700/quality/90/unsharp/true/compress/true'}
        ,{src:'http://img2.jc001.cn/img/488/1501488/1208/125024881294a30.jpg'}
        ,{src:'http://img2.jc001.cn/img/488/1501488/1208/125024acfad88b1.jpg'}
      ],
      overLayerShow:false,
    }
  },
  methods: {
    //获取物料板JSON数据
    getData() {
      this.sendMessage('getData');
    },
    //拖拽开始
    onDrag(e){
      this.overLayerShow = true;
      const dataTransfer = e.dataTransfer;
      const dom = e.target;
      if(dom&&dataTransfer){
        let index = dom.getAttribute("k") || 0;
        dataTransfer.setData('item', JSON.stringify(this.imgArr[index]))
      }
    },
    //拖拽结束
    onDragEnd(e){
      this.overLayerShow = false;
      const dataTransfer = e.dataTransfer;
      dataTransfer&&dataTransfer.clearData();
    },
    //放下
    drop(e){
      const dataTransfer = e.dataTransfer;
      const item = dataTransfer&&dataTransfer.getData('item');
      if(item){
        const obj = JSON.parse(item);
        /**
         * 参数说明：
         * @src 地址(必填)
         * @width 宽度（可选）
         * @height 高度（可选）
         */
        this.sendMessage('inertImage',{src:obj.src});
      }
    },
    sendMessage(method,data){
      const dataObj = {method:method,data:data};
      document.getElementById("boardFrame").contentWindow.postMessage(dataObj,"*");
    },
    receiveMessage(e){
      if(typeof e.data === 'string'){
        const obj = JSON.parse(e.data);
        if(!obj.method){
          return;
        }
        const result = obj.result;
        if(result&&result.code!=0){
          alert(result.desc);
          return;
        }
        switch (obj.method) {
          //读取JSON接口
          case 'getData':
            console.log(JSON.stringify(result.data));
            break;
        }
      }
    },
  },
  mounted() {
    window.addEventListener("message", this.receiveMessage, false);
  },
  beforeUnmount(){
    window.removeEventListener("message", this.receiveMessage, false);
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .demo-box{
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }
  .header{
    display: flex;
    height: 60px;
    background-color: #3D372C;
    align-items: center;
    color: #ffffff;
    justify-content: center;
  }
  .main{
    display: flex;
    height: calc(100vh - 60px);
  }
  .left{
    width: 25%;
    height: 100%;
    display: flex;
  }
  .tools{
    width: 80px;
    background-color: #3D372C;
    height: 100%;
  }
  .board-content{
    flex: 1;
    background-color: #4B4945;
    height: 100%;
    overflow-y: auto;
    padding: 20px;
  }
  .right{
    flex: 1;
    height:100%;
    position: relative;
  }
  .right .overLayer{
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
  }
  .api-box{
    margin: 10px;

  }

  .tools-item{
    width: 80px;
    height: 80px;
    color: #c3c3c3;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
  }
  .tools-item.active{
    background-color: #4B4945;
    color: #ffffff;
  }

  .api-drag-box{
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;
  }
  .api-drag-box img{
    width: 100px;
    height: 100px;
    margin-left: 20px;
    margin-bottom: 20px;
  }

</style>
