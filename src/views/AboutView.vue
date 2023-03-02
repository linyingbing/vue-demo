<template>
  <div class="home">
    <div class="left">
      <iframe id="iFrame" ref="iFrame" :src="src" frameborder="0" style="width: 100%; height: 100%" />
    </div>
    <button @click="handleSubmit">点击获取页面数据排序</button>
  </div>
</template>

<script>
export default {
  name: 'AboutView',
  data () {
    return {
      src: '',
      iframe: {},
      ratioNum: 100,
      
      parentClassName: 'drag-box',
      selectedClassName: 'selected'
    }
  },
  components: {
  },
  mounted () {
    this.demo();
    
  },
  methods: {
    demo() {
      this.src = '../../page.html'
      // this.src = '../../drag.html'
      this.$nextTick(() => {
        const iframe = window.frames['iFrame']
        const handleLoad = () => {
          setTimeout(() => {
            const Do = (iframe.contentDocument || iframe.contentWindow.document )
            // 计算缩放比例
            this.getRatio({Do, offsetWidth: (iframe.offsetWidth - 84 / 100)})

            // 在报告页面外包裹一层拖拽层
            this.appendDragBox({Do})

            // 在报告页面添加拖拽属性
            /* this.appendDrag({Do}) */
            
            // 监听拖拽动作
            this.drag({Do})
          }, 500)
        }
        iframe.addEventListener('load', handleLoad, true)
      })
    },

    // 计算缩放比例
    getRatio({Do, offsetWidth}) {
      var ratioNum = offsetWidth / 2480;
      ratioNum = parseInt(ratioNum * 100) < 100 ? parseInt(ratioNum * 100) : 100;
      Do.getElementsByTagName('body')[0].style = `zoom:${ratioNum}%`
      this.ratioNum = ratioNum
    },

    // 在每一页的报告页面外包裹一层拖拽层
    appendDragBox({Do}) {
      let bodyNode = Do.getElementsByTagName('body')[0]
      let divs = bodyNode.children
      let newNode = Do.createElement('div');//添加元素
      newNode.style = `padding: ${21*100 / this.ratioNum}px 0;border: 21px solid #000;box-sizing: border-box; width: auto;`
      newNode.classList.add('drag-box')
      for(var i = 0; i < divs.length; i++) {
        let div = divs[i]
        if(div.className.indexOf('fixpage') > -1 || div.className.indexOf('activepage') > -1) {
          // 判断是否为报告页
          let elem = newNode.cloneNode(true)
          elem.setAttribute('draggable', true)
          elem.setAttribute('sort', i)
          elem.appendChild(div.cloneNode(true))
          
          div.parentNode.replaceChild(elem, div)
        }
      }
    },
    // 在报告页面添加拖拽属性
    appendDrag({Do}) {
      let bodyNode = Do.getElementsByTagName('body')[0]
      let divs = bodyNode.children
      for(var i = 0; i < divs.length; i++) {
        let div = divs[i]
        if(div.className.indexOf('fixpage') > -1 || div.className.indexOf('activepage') > -1) {
          // 判断是否为报告页
          div.setAttribute('draggable', true)
          div.setAttribute('sort', i)
          div.classList.add('drag-box')
        }
      }
    },

    // 单选拖拽
    drag ({Do}) {
      // 获取列表dom
      let list = Do.querySelector('body');
      const scrollHeight = parseInt(list.scrollHeight * (this.ratioNum / 100))
      //用pageY作为位置计算
      // 创建cruentItem存放将要拖动的元素
      let cruentItem

      // 点击选择
      list.onclick = (e) => {
        const offsetHeight = e.target.closest('.drag-box').offsetHeight * (this.ratioNum / 100)
        console.log("点击 ==>> ", e)
        console.log("当前Item的高度 ==>> ", offsetHeight)
        console.log("list的高度 ==>> ", scrollHeight)
        // 此处使用setTimeout延迟被拖动的原始元素的样式
        if(e.target.closest('.drag-box').className.indexOf('selected') > -1) {
          e.target.closest('.drag-box').classList.remove("selected");
        } else {
          e.target.closest('.drag-box').classList.add("selected");
        }
      }
      // 当某被拖动的对象在另一对象容器范围内拖动时触发此事件
      list.ondragover = e => {
          e.preventDefault();
      }
      // 拖动开始
      list.ondragstart = e => {
        console.log('拖动开始', e)
        // 判断当前元素是否被选中，若未被选中，则原来选中的元素(们)清空选中样式
        let dragIsSelect = false;
        const selectedItems = list.getElementsByClassName('selected')
        for(var i = 0; i < selectedItems.length; i++) {
          if(selectedItems[i] === e.target) {
            dragIsSelect = true
          }
        }
        if(!dragIsSelect) {
          setTimeout(() => {
              e.target.closest('.drag-box').classList.add("moving");
          },0)
          // 存储被拖动元素
          cruentItem = e.target;
          // 拖动时默认行为是复制，此处可以改为移动
          e.dataTransfer.effectAllowed = 'move';
          console.log(cruentItem)
          for(var j = 0; j < selectedItems.length; j++) {
            console.log(selectedItems[j])
            selectedItems[j].classList.remove('selected')
          }
        }
        // 此处使用setTimeout延迟被拖动的原始元素的样式
        /* setTimeout(() => {
            e.target.classList.add("moving");
        },0)
        // 存储被拖动元素
        cruentItem = e.target;
        // 拖动时默认行为是复制，此处可以改为移动
        e.dataTransfer.effectAllowed = 'move'; */
      }

      // 拖动中
      list.ondragenter = e => {
        console.log('拖动中', e)
        /* // 阻止默认事件，否则元素会先回到拖动开始时的位置，再到拖动结束的位置
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
        if(e.target.getAttribute('draggable')) {
          if (tiemListIndex < targetIndex) {
            console.log('往下移动');
            // 当前拖动元素插入到目标元素前面，且nextElementSibling目标元素的下一个兄弟元素
            list.insertBefore(cruentItem, e.target.nextElementSibling)
          } else {
            console.log('往上移动');
              list.insertBefore(cruentItem, e.target)
          }
        } */
          
      }

      // 拖动结束
      list.ondragend = e => {
          // 结束后移除虚线样式
          e.target.classList.remove('moving')
      }


    },
    handleSubmit() {
      const iframe = window.frames['iFrame']
      const Do = (iframe.contentDocument || iframe.contentWindow.document )
      let bodyNode = Do.getElementsByTagName('body')[0]
      let divs = bodyNode.children
      let sortArr = []
      for(var i = 0; i < divs.length; i++) {
        let div = divs[i]
        if(div.getAttribute('draggable')) {
          // 判断是否为报告页
          const sort = div.getAttribute('sort')
          sortArr.push(sort)
        }
      }
      
      console.log("排序后的顺序 ==>>", sortArr)
    }
  }
}
</script>
<style scoped>
.left {
  float: left;
  box-sizing: border-box;
  padding: 10px;
  width: 200px;
  height: calc(100vh - 78px);
  border: 1px solid #000;
  
}
.right {
  float: right;
}
.left iframe {
  /* 隐藏滚动条 */
  scrollbar-width: none; /* firefox */
  -ms-overflow-style: none; /* IE 10+ */
}
.left iframe::-webkit-scrollbar {
    display: none; /* Chrome Safari */
}
.moving {
  background-color: transparent;
  color: transparent;
  border: 1px dashed black;
}
</style>
