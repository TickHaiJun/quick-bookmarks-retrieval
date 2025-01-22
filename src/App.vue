<script setup>
// 导入响应式API
import { ref, computed, onMounted } from 'vue'

// 搜索关键词
const searchQuery = ref('')

// 控制弹出框显示
const isPopupVisible = ref(false)

// 书签数据
const bookmarks = ref([])

// 获取书签数据
const fetchBookmarks = async () => {
  try {
    const bookmarkTree = await chrome.bookmarks.getTree()
    const flattenBookmarks = []

    // 递归遍历书签树
    const traverseBookmarks = (nodes) => {
      for (const node of nodes) {
        if (node.url) {
          flattenBookmarks.push({
            id: node.id,
            title: node.title,
            url: node.url,
            icon: `https://www.google.com/s2/favicons?domain=${new URL(node.url).hostname}&sz=32`
          })
        }
        if (node.children) {
          traverseBookmarks(node.children)
        }
      }
    }

    traverseBookmarks(bookmarkTree)
    bookmarks.value = flattenBookmarks
  } catch (error) {
    console.error('获取书签失败:', error)
  }
}

// 组件挂载时获取书签和注册快捷键
onMounted(() => {
  fetchBookmarks()

  // 注册快捷键
  document.addEventListener('keydown', (e) => {
    if (e.ctrlKey && e.key === 'b') {
      e.preventDefault()
      isPopupVisible.value = !isPopupVisible.value
      if (isPopupVisible.value) {
        setTimeout(() => {
          document.querySelector('.search-input')?.focus()
        }, 0)
      }
    }
  })
})

// 过滤书签
const filteredBookmarks = computed(() => {
  return bookmarks.value.filter(bookmark =>
    bookmark.title.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

// 打开书签
const openBookmark = (url) => {
  window.open(url, '_blank')
}
</script>

<template>
  <div class="container" :class="{ 'popup-mode': isPopupVisible }">
    <div v-if="!isPopupVisible" class="background-gradient"></div>
    <!-- 标题区域 -->
    <div class="header">
      <h1>Quick bookmarks retrieval</h1>
      <p class="subtitle">海军制作 Wechat: daxin261</p>
    </div>

    <!-- 搜索框 -->
    <div class="grid"></div>
    <div id="poda">
      <div class="glow"></div>
      <div class="darkBorderBg"></div>
      <div class="darkBorderBg"></div>
      <div class="darkBorderBg"></div>
      <div class="white"></div>
      <div class="border"></div>
      <div id="main">
        <input placeholder="Search..." type="text" v-model="searchQuery" name="text" class="input" />
        <div id="input-mask"></div>
        <div id="pink-mask"></div>
        <div class="filterBorder"></div>
        <div id="filter-icon">
          <svg preserveAspectRatio="none" height="27" width="27" viewBox="4.8 4.56 14.832 15.408" fill="none">
            <path
              d="M8.16 6.65002H15.83C16.47 6.65002 16.99 7.17002 16.99 7.81002V9.09002C16.99 9.56002 16.7 10.14 16.41 10.43L13.91 12.64C13.56 12.93 13.33 13.51 13.33 13.98V16.48C13.33 16.83 13.1 17.29 12.81 17.47L12 17.98C11.24 18.45 10.2 17.92 10.2 16.99V13.91C10.2 13.5 9.97 12.98 9.73 12.69L7.52 10.36C7.23 10.08 7 9.55002 7 9.20002V7.87002C7 7.17002 7.52 6.65002 8.16 6.65002Z"
              stroke="#d6d6e6" stroke-width="1" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round">
            </path>
          </svg>
        </div>
        <div id="search-icon">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" viewBox="0 0 24 24" stroke-width="2"
            stroke-linejoin="round" stroke-linecap="round" height="24" fill="none" class="feather feather-search">
            <circle stroke="url(#search)" r="8" cy="11" cx="11"></circle>
            <line stroke="url(#searchl)" y2="16.65" y1="22" x2="16.65" x1="22"></line>
            <defs>
              <linearGradient gradientTransform="rotate(50)" id="search">
                <stop stop-color="#f8e7f8" offset="0%"></stop>
                <stop stop-color="#b6a9b7" offset="50%"></stop>
              </linearGradient>
              <linearGradient id="searchl">
                <stop stop-color="#b6a9b7" offset="0%"></stop>
                <stop stop-color="#837484" offset="50%"></stop>
              </linearGradient>
            </defs>
          </svg>
        </div>
      </div>
    </div>

    <!-- 书签列表 -->
    <div class="bookmarks-list">
      <div v-for="bookmark in filteredBookmarks" :key="bookmark.id" class="bookmark-item"
        @click="openBookmark(bookmark.url)">
        <img :src="bookmark.icon" alt="icon" class="bookmark-icon" />
        <span class="bookmark-title">{{ bookmark.title }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
#filter-icon {
  position: absolute;
  right: 15px;
  top: 15px;
}

.input {
  width: 100%;
  height: 56px;
  padding: 0 60px;
  font-size: 16px;
  border: none;
  outline: none;
  border-radius: 10px;
  background: linear-gradient(180deg, #161329, black, #1d1b4b);
  border: 1px solid transparent;
  color: #fff;
}

.filterBorder {
  height: 42px;
  width: 40px;
  position: absolute;
  overflow: hidden;
  top: 7px;
  right: 7px;
  border-radius: 10px;
}

.filterBorder::before {
  content: "";
  text-align: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(90deg);
  position: absolute;
  width: 600px;
  height: 600px;
  background-repeat: no-repeat;
  background-position: 0 0;
  filter: brightness(1.35);
  background-image: conic-gradient(rgba(0, 0, 0, 0),
      #3d3a4f,
      rgba(0, 0, 0, 0) 50%,
      rgba(0, 0, 0, 0) 50%,
      #3d3a4f,
      rgba(0, 0, 0, 0) 100%);
  animation: rotate 4s linear infinite;
}

@keyframes rotate {
  from {
    transform: translate(-50%, -50%) rotate(0deg);
  }

  to {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

#main {
  position: relative;
  margin-bottom: 20px;
}

#search-icon {
  position: absolute;
  left: 20px;
  top: 15px;
}

.container {
  width: 700px;
  height: 600px;
  margin: 0 auto;
  padding: 16px;
  color: #333;
  position: relative;
  background: #f8fafc;
  overflow-y: auto;
}

.popup-mode {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 700px;
  height: 600px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  padding: 20px;
}

.background-gradient {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 200px;
  background: linear-gradient(135deg, #6366f1 0%, #a855f7 50%, #ec4899 100%);
  opacity: 0.85;
  z-index: -1;
  border-radius: 0 0 20px 20px;
  animation: gradientFlow 15s ease-in-out infinite;
  background-size: 400% 400%;
  transition: all 0.5s ease;
}

@keyframes gradientFlow {
  0% {
    background-position: 0% 50%;
  }

  50% {
    background-position: 100% 50%;
  }

  100% {
    background-position: 0% 50%;
  }
}

.header {
  text-align: center;
  margin-bottom: 24px;
}

h1 {
  font-size: 2em;
  margin-bottom: 8px;
  background: linear-gradient(135deg, #6366f1 0%, #a855f7 50%, #ec4899 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 600;
}

.subtitle {
  background: linear-gradient(135deg, #6366f1 0%, #a855f7 50%, #ec4899 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-size: 0.9em;
}

.bookmarks-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 12px;
  margin-top: 16px;
}

.bookmark-item {
  display: flex;
  align-items: center;
  padding: 10px 16px;
  background: white;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  border: 1px solid #e2e8f0;
  backdrop-filter: blur(8px);
  background-color: rgba(255, 255, 255, 0.8);
}

.bookmark-item:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.15);
  border-color: #6366f1;
  background-color: rgba(255, 255, 255, 0.95);
}

.bookmark-icon {
  width: 20px;
  height: 20px;
  margin-right: 12px;
  object-fit: cover;
  border-radius: 4px;
}

.bookmark-title {
  font-size: 0.95em;
  color: #334155;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: calc(100% - 32px);
}



/* From Uiverse.io by Lakshay-art */
.grid {
  height: 800px;
  width: 800px;
  background-image: linear-gradient(to right, #0f0f10 1px, transparent 1px),
    linear-gradient(to bottom, #0f0f10 1px, transparent 1px);
  background-size: 1rem 1rem;
  background-position: center center;
  position: absolute;
  z-index: -1;
  filter: blur(1px);
}

.white,
.border,
.darkBorderBg,
.glow {
  max-height: 70px;
  max-width: 314px;
  height: 100%;
  width: 100%;
  position: absolute;
  overflow: hidden;
  z-index: -1;
  /* Border Radius */
  border-radius: 12px;
  filter: blur(3px);
}

.input {
  background-color: #010201;
  border: none;
  /* padding:7px; */
  width: 301px;
  height: 56px;
  border-radius: 10px;
  color: white;
  padding-inline: 59px;
  font-size: 18px;
}

#poda {
  display: flex;
  align-items: center;
  justify-content: center;
}

.input::placeholder {
  color: #c0b9c0;
}

.input:focus {
  outline: none;
}

#main:focus-within>#input-mask {
  display: none;
}

#input-mask {
  pointer-events: none;
  width: 100px;
  height: 20px;
  position: absolute;
  background: linear-gradient(90deg, transparent, black);
  top: 18px;
  left: 70px;
}

#pink-mask {
  pointer-events: none;
  width: 30px;
  height: 20px;
  position: absolute;
  background: #cf30aa;
  top: 10px;
  left: 5px;
  filter: blur(20px);
  opacity: 0.8;
  animation:leftright 4s ease-in infinite;
  transition: all 2s;
}

#main:hover>#pink-mask {
  animation: rotate 4s linear infinite;
  opacity: 0;
}

.white {
  max-height: 63px;
  max-width: 307px;
  border-radius: 10px;
  filter: blur(2px);
}

.white::before {
  content: "";
  z-index: -2;
  text-align: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(83deg);
  position: absolute;
  width: 600px;
  height: 600px;
  background-repeat: no-repeat;
  background-position: 0 0;
  filter: brightness(1.4);
  background-image: conic-gradient(rgba(0, 0, 0, 0) 0%,
      #a099d8,
      rgba(0, 0, 0, 0) 8%,
      rgba(0, 0, 0, 0) 50%,
      #dfa2da,
      rgba(0, 0, 0, 0) 58%);
    animation: rotate 4s linear infinite;
  transition: all 2s;
}

.border {
  max-height: 59px;
  max-width: 303px;
  border-radius: 11px;
  filter: blur(0.5px);
}

.border::before {
  content: "";
  z-index: -2;
  text-align: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(70deg);
  position: absolute;
  width: 600px;
  height: 600px;
  filter: brightness(1.3);
  background-repeat: no-repeat;
  background-position: 0 0;
  background-image: conic-gradient(#1c191c,
      #402fb5 5%,
      #1c191c 14%,
      #1c191c 50%,
      #cf30aa 60%,
      #1c191c 64%);
  animation: rotate 4s 0.1s linear infinite;
  transition: all 2s;
}

.darkBorderBg {
  max-height: 65px;
  max-width: 312px;
}

.darkBorderBg::before {
  content: "";
  z-index: -2;
  text-align: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(82deg);
  position: absolute;
  width: 600px;
  height: 600px;
  background-repeat: no-repeat;
  background-position: 0 0;
  background-image: conic-gradient(rgba(0, 0, 0, 0),
      #18116a,
      rgba(0, 0, 0, 0) 10%,
      rgba(0, 0, 0, 0) 50%,
      #6e1b60,
      rgba(0, 0, 0, 0) 60%);
  transition: all 2s;
}

#poda:hover>.darkBorderBg::before {
  transform: translate(-50%, -50%) rotate(262deg);
}

#poda:hover>.glow::before {
  transform: translate(-50%, -50%) rotate(240deg);
}

#poda:hover>.white::before {
  transform: translate(-50%, -50%) rotate(263deg);
}

#poda:hover>.border::before {
  transform: translate(-50%, -50%) rotate(250deg);
}

#poda:hover>.darkBorderBg::before {
  transform: translate(-50%, -50%) rotate(-98deg);
}

#poda:hover>.glow::before {
  transform: translate(-50%, -50%) rotate(-120deg);
}

#poda:hover>.white::before {
  transform: translate(-50%, -50%) rotate(-97deg);
}

#poda:hover>.border::before {
  transform: translate(-50%, -50%) rotate(-110deg);
}

#poda:focus-within>.darkBorderBg::before {
  transform: translate(-50%, -50%) rotate(442deg);
  transition: all 4s;
}

#poda:focus-within>.glow::before {
  transform: translate(-50%, -50%) rotate(420deg);
  transition: all 4s;
}

#poda:focus-within>.white::before {
  transform: translate(-50%, -50%) rotate(443deg);
  transition: all 4s;
}

#poda:focus-within>.border::before {
  transform: translate(-50%, -50%) rotate(430deg);
  transition: all 4s;
}

.glow {
  overflow: hidden;
  filter: blur(30px);
  opacity: 0.4;
  max-height: 130px;
  max-width: 354px;
}

.glow:before {
  content: "";
  z-index: -2;
  text-align: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(60deg);
  position: absolute;
  width: 999px;
  height: 999px;
  background-repeat: no-repeat;
  background-position: 0 0;
  /*border color, change middle color*/
  background-image: conic-gradient(#000,
      #402fb5 5%,
      #000 38%,
      #000 50%,
      #cf30aa 60%,
      #000 87%);

  animation: rotate 4s 0.3s linear infinite;
  transition: all 2s;
}

@keyframes rotate {
  100% {
    transform: translate(-50%, -50%) rotate(450deg);
  }
}

@keyframes leftright {
  0% {
    transform: translate(0px, 0px);
    opacity: 1;
  }

  49% {
    transform: translate(250px, 0px);
    opacity: 0;
  }

  80% {
    transform: translate(-40px, 0px);
    opacity: 0;
  }

  100% {
    transform: translate(0px, 0px);
    opacity: 1;
  }
}

#filter-icon {
  position: absolute;
  top: 8px;
  right: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
  max-height: 40px;
  max-width: 38px;
  height: 100%;
  width: 100%;

  isolation: isolate;
  overflow: hidden;
  /* Border Radius */
  border-radius: 10px;
  background: linear-gradient(180deg, #161329, black, #1d1b4b);
  border: 1px solid transparent;
}

.filterBorder {
  height: 42px;
  width: 40px;
  position: absolute;
  overflow: hidden;
  top: 7px;
  right: 7px;
  border-radius: 10px;
}

.filterBorder::before {
  content: "";

  text-align: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) rotate(90deg);
  position: absolute;
  width: 600px;
  height: 600px;
  background-repeat: no-repeat;
  background-position: 0 0;
  filter: brightness(1.35);
  background-image: conic-gradient(rgba(0, 0, 0, 0),
      #3d3a4f,
      rgba(0, 0, 0, 0) 50%,
      rgba(0, 0, 0, 0) 50%,
      #3d3a4f,
      rgba(0, 0, 0, 0) 100%);
  animation: rotate 4s linear infinite;
}

#main {
  position: relative;
}

#search-icon {
  position: absolute;
  left: 20px;
  top: 15px;
}
</style>
