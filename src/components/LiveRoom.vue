<script setup>
import { nextTick, onMounted, ref } from "vue";
import BScroll from "better-scroll";
let timer = null;
let barrageList = ref([]);
let newBarrageList = ref([]); //新留言
const isBottom = ref(true);
const isTouch = ref(false);
const i = ref(0);
const message = ref("");
let scroll = null;
onMounted(() => {
  initSwiper();
  setBarrage();
});
const barrage = ref(null);
const barrage_wrap = ref(null);
const initSwiper = () => {
  scroll = new BScroll(barrage.value, {
    scrollY: true,
    click: true,
    probeType: 3,
    mouseWheel: true,
    bounce: {
      top: true,
      bottom: false,
    },
    scrollbar: {
      fade: true,
    },
  });
  //用户轻抚
  scroll.on("flick", () => {
    let barAllH = barrage_wrap.value.offsetHeight;
    let wrapH = barrage.value.offsetHeight;
    if (barAllH > wrapH) {
      isTouch.value = true;
    }
  });
  //滚动中
  scroll.on("scroll", (obj) => {
    let barAllH = barrage_wrap.value?.offsetHeight;
    // console.log(barAllH, Math.abs(obj.y));
    if (barAllH - 429 <= Math.abs(obj.y)) {
      isBottom.value = true;
      isTouch.value = false;
      if (newBarrageList.value.length) {
        getNewBarrage();
      }
    } else if (barAllH - 429 > 0) {
      isBottom.value = false;
    }
  });
};
const setBarrage = () => {
  timer = setTimeout(() => {
    // 从弹幕池中获取的新的上屏弹幕，可以固定每次最多n条上屏
    // 注意弹幕上屏的时间要大于过渡动画以及删除弹幕的时间，否则会引起闪屏
    const upBarrage = [
      { name: "万里无云" + i.value, text: "主播666", id: i.value++ },
      { name: "万里无云" + i.value, text: "主播666", id: i.value++ },
    ];
    dataPush(upBarrage);
    setBarrage();
  }, 2000);
};
const sendBarrage = () => {
  if (message.value) {
    const upBarrage = [
      { name: "万里无云", text: message.value, id: i.value++ },
    ];
    dataPush(upBarrage);
    message.value = "";
  }
};
//设置滚动条滚动至底
const setScrollTop = () => {
  let Lis = barrage_wrap.value?.children; // 查找所有bar_block元素
  const h = barrage_wrap.value.offsetHeight;
  console.log("容器高度：", h, "元素个数：", Lis.length);
  scroll.refresh();
  scroll.scrollToElement(Lis[Lis.length - 1], 400); // 滑动到指定元素,设置动画400ms
  const len = barrageList.value.length;

  // 限制弹幕超过200条时，固定150条，优化性能
  // 删除多余的弹幕
  if (len > 200) {
    setTimeout(() => {
      barrageList.value.splice(0, len - 150);
    }, 500);
  }
};
//数据获取
const dataPush = (data) => {
  console.log(isBottom.value, isTouch.value);
  if (isBottom.value && !isTouch.value) {
    barrageList.value = [...barrageList.value, ...data];
    nextTick(() => {
      setScrollTop();
    });
  } else {
    newBarrageList.value = newBarrageList.value.concat(data);
    // this.$set(this.newBarrageList, this.newBarrageList.length, ...data);
  }
};
//读取新消息
const getNewBarrage = () => {
  clearTimeout(timer); // 拼接滚动到展开的新弹幕时，停止读取上屏弹幕
  barrageList.value = [...barrageList.value, ...newBarrageList.value];
  newBarrageList.value = [];
  isBottom.value = true;
  nextTick(() => {
    setScrollTop();
    setBarrage(); // 拼接滚动完成后，继续上屏弹幕
  });
};
</script>

<template>
  <div>
    <div
      style="
        position: fixed;
        top: 0;
        right: 0;
        color: #f4da92;
        margin: 20px;
        font-size: 36px;
        background: rgba(0, 0, 0, 0.4);
        width: 100px;
        height: 36px;
        display: flex;
        justify-content: space-around;
        border-radius: 36px;
        line-height: 36px;
      "
    >
      <span>-</span>
      <span style="transform: rotate(45deg); display: inline-block">+</span>
    </div>
    <div class="wrap">
      <div class="barrage" ref="barrage">
        <div id="barrage_wrap" class="barrage_wrap" ref="barrage_wrap">
          <div
            class="bar_block"
            v-for="(item, index) in barrageList"
            :key="item.id"
          >
            <div class="barragediv">
              <span class="barrage_name">{{ item.name }}：</span>
              {{ item.text }}-{{ item.id }}
            </div>
          </div>
        </div>
      </div>
      <div
        class="newBarrage"
        v-if="newBarrageList.length > 0"
        @click="getNewBarrage"
      >
        {{ newBarrageList.length }}条新消息
      </div>
    </div>
    <div
      style="
        position: fixed;
        bottom: 0;
        padding: 20px;
        width: 100%;
        display: flex;
        justify-content: space-between;
        box-sizing: border-box;
      "
    >
      <input
        type="text"
        name="barrage_input"
        id="barrage_input"
        @keyup.enter="sendBarrage"
        v-model="message"
      />
      <button id="barrage_send" @click="sendBarrage">发送</button>
    </div>
  </div>
</template>

<style scoped>
.wrap {
  width: 100%;
  height: 150px;
  left: 50%;
  bottom: 20px;
  position: fixed;
  overflow: hidden;
  transform: translate(-50%, -50%);
}

.barrage {
  width: 100%;
  font-size: 12px;
  height: 150px;
  padding: 20px;
  z-index: 99;
  border-radius: 5px;
  bottom: 0;
  transition: all 0.4s;
  overflow-y: hidden;
  -ms-overflow-style: none;
  overflow: -moz-scrollbars-none;
}

.barrage::-webkit-scrollbar {
  width: 0 !important;
}

.barrage .scrollstyle {
  width: 50% !important;
  left: 50% !important;
  margin-left: -25% !important;
}

.barrage .bar_block {
  padding-bottom: 10px;
  max-width: 300px;
}

.barrage_wrap {
  position: absolute;
}

.barrage .barragediv {
  word-wrap: break-word;
  /*在长单词或URL地址内部进行换行，防止过长导致自动换行 */
  word-break: break-all;
  /*允许再单词内换行*/
  display: inline-block;
  max-width: 400px;
  color: #fff;
  background: rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  line-height: 24px;
  padding: 0 10px;
}

.barrage .barragediv .barrage_name {
  color: #ffbb7e;
}

.newBarrage {
  display: inline-block;
  color: #f85b0f;
  position: absolute;
  font-size: 12px;
  padding: 0 19px;
  border-radius: 10px;
  bottom: 0;
  height: 24px;
  z-index: 99;
  line-height: 24px;
  background: #fff;
  left: 20px;
  letter-spacing: 1px;
}
#barrage_input {
  background: #fff;
  border: none;
  outline: none;
  padding: 8px;
  margin: 0;
  font-size: 12px; /* 根据需要调整字体大小 */
  color: currentColor; /* 使用当前文本颜色 */
  border-radius: 16px;
  width: 70%;
}
#barrage_input:focus {
  outline: none;
}
#barrage_send {
  width: 20%;
  padding: 8px;
  border: none;
  background-color: #f4da92;
  color: #000;
  border-radius: 18px;
}
</style>
