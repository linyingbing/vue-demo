<template>
  <div class="home">
    <div class="left">
      <iframe id="iFrame" :src="src" frameborder="0" style="width: 100%; height: 100%" />
    </div>
    <button @click="handleSubmit">点击获取页面数据排序</button>
  </div>
</template>

<script>
export default {
  name: 'HomeView',
  data () {
    return {
      src: ''
    }
  },
  components: {
  },
  mounted () {
    this.src = '../../page.html'
    // this.src = 'http://127.0.0.1:8848/report-front/intervention-comparison.html?id=1626165191608160257&resize=true'
    // this.src = 'https://stage.report.ricare.online/intervention-comparison.html?id=1626165191608160257&resize=true'
    /* var ratioNum = window.screen.availWidth/2480;
				ratioNum = parseInt(ratioNum * 100) < 100 ? parseInt(ratioNum * 100) : 100;
				$("body").css("zoom", ratioNum + '%'); */
    this.$nextTick(() => {
      const iframe = window.frames['iFrame']
      const handleLoad = () => {
        setTimeout(() => {
          const Do = (iframe.contentWindow || iframe.contentDocument)
          let menus = Do.document.getElementsByClassName('menu_wp')
          for(var i = 0; i < menus.length; i++) {
            menus[i].setAttribute('draggable', true)
            menus[i].setAttribute('sort', i)
          }
          this.drag(Do.document)
        }, 500)
      }
      iframe.addEventListener('load', handleLoad, true)
    })
  },
  methods: {
    drag (doc) {
      // 获取列表dom
      let list = doc.querySelector('.menu_list');
      console.log("list ==>>", list)
      // 创建cruentItem存放将要拖动的元素
      let cruentItem
      list.mousedown = e => {
        e.preventDefault();
      }
      // 当某被拖动的对象在另一对象容器范围内拖动时触发此事件
      list.ondragover = e => {
          e.preventDefault();
      }
      // 拖动开始
      list.ondragstart = e => {
        console.log('拖动开始')
          // 此处使用setTimeout延迟被拖动的原始元素的样式
          setTimeout(() => {
              e.target.classList.add("moving");
          },0)
          // 存储被拖动元素
          cruentItem = e.target;
          // 拖动时默认行为是复制，此处可以改为移动
          e.dataTransfer.effectAllowed = 'move';
      }

      // 拖动中
      list.ondragenter = e => {
        console.log('拖动中')
          // 阻止默认事件，否则元素会先回到拖动开始时的位置，再到拖动结束的位置
          e.preventDefault();
          // 拖动事件期间排除被拖动元素自身，以及事件代理对象ul
          if(e.target == cruentItem || e.target == list) {
              return;
          }
          // list.children获取的是类数组，类数组没有数组的方法，所以要通过Array.from转换为真正的数组
          let itmeList = Array.from(list.children);
          // 获取当前拖动元素位置的下标
          let tiemListIndex = itmeList.indexOf(cruentItem);
          // 获取当前拖动元素所移动到的位置的元素的下标
          let targetIndex = itmeList.indexOf(e.target);
          // 如果当前拖动元素下标小于目标元素下标说明是往下移动，否则网上移动
          if (tiemListIndex < targetIndex) {
              console.log('往下移动');
              // 当前拖动元素插入到目标元素前面，且nextElementSibling目标元素的下一个兄弟元素
              list.insertBefore(cruentItem, e.target.nextElementSibling)
          } else {
              console.log('往上移动');
              list.insertBefore(cruentItem, e.target)
          }
      }

      // 拖动结束
      list.ondragend = e => {
          // 结束后移除虚线样式
          e.target.classList.remove('moving')
      }


    },
    handleSubmit() {
      const iframe = window.frames['iFrame']
      const Do = (iframe.contentWindow || iframe.contentDocument)
      let menus = Do.document.getElementsByClassName('menu_wp')
      let sortArr = []
      for(var i = 0; i < menus.length; i++) {
        const sort = menus[i].getAttribute('sort')
        sortArr.push(sort)
      }

      console.log("排序后的顺序 ==>>", sortArr)
    }
  }
}
</script>
<style scoped>
.left {
  margin: 0 auto;
  width: 50%;
  height: 1000px;
  border: 1px solid #000;
}
.moving {
  background-color: transparent;
  color: transparent;
  border: 1px dashed black;
}
</style>
