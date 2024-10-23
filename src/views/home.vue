<template>
  <div class="container-home">
    <div class="title">暖欣互联网医院</div>
    <div class="container">
      <div class="imgbox">
        <div
          class="img-wrap"
          :class="{'act': curIndex === imgIndex}"
          v-for="(img, imgIndex) in imgs"
          :key="imgIndex"
          @click="changeImgs(img, imgIndex)">
          <img :src="img" alt="">
        </div>
      </div>
      <div class="measurebox">
        <div class="tools">
          <el-button type="primary" @click="onToggleMeasure">{{createType === 0 ? '测量' : createType === 4 ? '取消测量' : ''}}</el-button>
          <el-button @click="zoom(true)">放大</el-button>
          <el-button @click="zoom(false)">缩小</el-button>
          <el-button @click="fitting">恢复尺寸</el-button>
          <el-button @click="clear">清除</el-button>
        </div>
        <canvas class="canvas"></canvas>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute()
const router = useRouter()
const instance = ref(null)
const createType = ref(0)
const curIndex = ref(0)
const output = ref(null)
const imgs = ref([])

const getImgs = async() => {
  const id = route.query?.id
  const loadingInstance = ElLoading.service({fullscreen: true})
  try {
    setTimeout(() => {
      imgs.value = [
        'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fss2.meipian.me%2Fusers%2F31299500%2F846d9b76bafe42e2b06a4eea8d44f5c3.jpeg%3Fmeipian-raw%2Fbucket%2Fivwen%2Fkey%2FdXNlcnMvMzEyOTk1MDAvODQ2ZDliNzZiYWZlNDJlMmIwNmE0ZWVhOGQ0NGY1YzMuanBlZw%3D%3D%2Fsign%2F603ae0109e37e7ce56ea7aac356909a4.jpg&refer=http%3A%2F%2Fss2.meipian.me&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1732289573&t=896b7b8c10647e6221607feb47b8cd55',
        'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fss2.meipian.me%2Fusers%2F31299500%2Fcc8f2c8753434d6291a48fb0439aefd4.jpeg%3Fmeipian-raw%2Fbucket%2Fivwen%2Fkey%2FdXNlcnMvMzEyOTk1MDAvY2M4ZjJjODc1MzQzNGQ2MjkxYTQ4ZmIwNDM5YWVmZDQuanBlZw%3D%3D%2Fsign%2F55077fa5a0bef10527c377a40dfee6f2.jpg&refer=http%3A%2F%2Fss2.meipian.me&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1732289519&t=fd5f910de2a8768af505dea0884f7813',
        'https://img0.baidu.com/it/u=1637358660,1417913388&fm=253&fmt=auto&app=138&f=JPEG?w=1067&h=800',
        'https://img2.baidu.com/it/u=1391526660,4144272199&fm=253&app=138&f=JPEG?w=904&h=758'
      ]
      init(imgs.value[0])
      loadingInstance.close()
    }, 1000)
  } catch (error) {

  }finally {
    // loadingInstance.close()
  }
}



onMounted(() => {
  getImgs()
})

const init = (img) => {
  instance.value = new CanvasSelect('.canvas')
  instance.value.setImage(img)
  // 0 不创建(默认)，1创建矩形，2创建多边形，3点标注，4折线标注，5圆形标注，6网格标注
  // 图片加载完成
  instance.value.on("load", (src) => {
    console.log("image loaded", src);
  });
  // 添加
  instance.value.on("add", (info) => {
    console.log("add", info);
    window.info = info;
  });
  // 删除
  instance.value.on("delete", (info) => {
    console.log("delete", info);
    window.info = info;
  });
  // 选中
  instance.value.on("select", (shape) => {
    console.log("select", shape);
    window.shape = shape;
  });
  instance.value.on("updated", (result) => {
    result.forEach(single => {
        const coors = single.coor;
        const p1 = {x: coors[0][0], y: coors[0][1]}
        const p2 = {x: coors[coors.length - 1][0], y: coors[coors.length - 1][1]}
        const lebel = Math.sqrt(Math.pow((p1.x - p2.x), 2) + Math.pow((p1.y - p2.y), 2))/10
        single.label = lebel.toFixed(2)+'mm';
      })
      // console.log('标注结果', result)
      const list = [...result];
      list.sort((a, b) => a.index - b.index);
      output.value = JSON.stringify(list, null, 2);
  });

}

const changeImgs = (img, index) => {
  if(curIndex.value === index) return
  curIndex.value = index;
  instance.value.setImage(img);
  clear()
}
const onToggleMeasure = () => {
  createType.value = createType.value === 4 ? 0 : 4
  instance.value.createType = createType.value
}
function zoom(type) {
  instance.value.setScale(type);
}
function fitting() {
  instance.value.fitZoom();
}
function changeData() {
  const data = JSON.parse(output.value);
  instance.value.setData(data);
}
function onFocus() {
  instance.value.setFocusMode(!instance.value.focusMode);
}
const clear = () => {
  createType.value = 0
  instance.value.createType = createType.value
  instance.value.setData([]);
}
</script>
<style lang="less" scoped>
.container-home {
  width: 100%;
  height: 100vh;
  padding: 30px;
  background: #f8f8f8 url(./../assets/imgs/bg_report.png) no-repeat top center;
  background-size: contain;
  .title {
    margin-bottom: 26px;
    width: 100%;
    font-size: 19px;
    line-height: 26px;
    font-weight: bolder;
    color: #fff;
  }
  .container {
    display: flex;
    width: 100%;
    height: calc(100vh - 120px);
    border-radius: 12px;
    background-color:#fff;
    overflow: hidden;
    .imgbox {
      width: 300px;
      height: 100%;
      padding: 20px;
      overflow-y: scroll;
      .img-wrap {
        display: inline-flex;
        margin-bottom: 10px;
        width: 125px;
        height: 125px;
        border: 3px solid transparent;
        &.act {
          border: 3px solid #409eff;
        }
        &:nth-child(2n){
          margin-left: 10px;
        }
        img {
          width: 100%;
          object-fit: cover;
        }
      }
    }
    .measurebox {
      flex: 1;
      padding-top: 10px;
      background-color: #f7fbff;
      text-align: center;
      .tools {
        display: flex;
        justify-content: center;
        margin-bottom: 10px;
        width: 100%;
      }
      .canvas {
        width: 500px;
        height: 500px;
        background-color: black;
        height: calc(100% - 42px);
      }
    }
  }
}
</style>

