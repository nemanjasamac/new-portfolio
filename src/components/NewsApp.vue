<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  isOpen: Boolean,
  zIndex: {
    type: Number,
    default: 100
  }
})

const emit = defineEmits(['close', 'focus'])

const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const position = ref({ x: 150, y: 80 })
const isMaximized = ref(false)
const preMaximizePosition = ref({ x: 0, y: 0 })

const sidebarItems = [
  { id: 'top_stories', label: 'Top Stories', icon: 'star' },
  { id: 'spotlight', label: 'Spotlight', icon: 'spotlight' },
  { id: 'projects', label: 'Projects', icon: 'projects' },
  { id: 'blog', label: 'Blog', icon: 'blog' },
]

const selectedItem = ref('top_stories')
const selectedArticle = ref(null)

const newsItems = ref([
  {
    id: 1,
    category: 'Projects',
    title: 'New Portfolio Launched',
    summary: 'Excited to announce the launch of my new portfolio website, built with Vue.js and designed to mimic macOS.',
    content: `I'm thrilled to share my latest project: a personal portfolio website that reimagines the web experience through the lens of a desktop operating system. Built with Vue.js 3 and Vite, this project was a deep dive into component architecture, state management, and complex CSS layouts.

The goal was to create an immersive environment where visitors can explore my work, skills, and background in a familiar yet novel way. Features include a functional Dock, draggable windows, a Launchpad, and fully interactive "apps" for different sections of content.

One of the biggest challenges was managing the z-index stacking context and window focus states to ensure a realistic window management experience. I utilized Vue's reactivity system to handle this seamlessly.

I hope you enjoy exploring this digital space as much as I enjoyed building it!`,
    date: '2h ago',
    source: 'Nemanja Samac',
    image: 'https://images.unsplash.com/photo-1498050108023-c5249f4df085?auto=format&fit=crop&w=800&q=80',
    featured: true
  },
  {
    id: 2,
    category: 'Career',
    title: 'Started a New Position',
    summary: 'I have officially started a new role as a Senior Frontend Developer. Looking forward to the new challenges!',
    content: `I'm happy to announce that I've started a new position as a Senior Frontend Developer! This is a huge milestone in my career, and I'm eager to bring my expertise in Vue.js and modern web technologies to the team.

In this role, I'll be focusing on architectural decisions, performance optimization, and mentoring junior developers. I'm looking forward to tackling complex problems and delivering high-quality user experiences.

Thanks to everyone who has supported me along the way!`,
    date: '1d ago',
    source: 'LinkedIn',
    image: 'https://images.unsplash.com/photo-1486312338219-ce68d2c6f44d?auto=format&fit=crop&w=800&q=80',
    featured: false
  },
  {
    id: 3,
    category: 'Blog',
    title: 'Understanding Vue 3 Composition API',
    summary: 'A deep dive into the Composition API and how it changes the way we write Vue components.',
    content: `The Composition API is a game-changer for Vue.js development. It allows for better logic reuse, more flexible code organization, and improved type inference with TypeScript.

In this article, we'll explore the core concepts: setup(), ref, reactive, computed, and watch. We'll also look at how to extract logic into composable functions, making your components cleaner and more maintainable.

Whether you're migrating from the Options API or starting fresh, understanding the Composition API is essential for modern Vue development.`,
    date: '2d ago',
    source: 'Medium',
    image: null,
    featured: false
  },
  {
    id: 4,
    category: 'Tech',
    title: 'The Future of Web Development',
    summary: 'Thoughts on where the web is heading in 2024 and beyond. AI, WASM, and more.',
    content: `The web development landscape is evolving rapidly. With the rise of AI-assisted coding tools, WebAssembly (WASM) gaining traction, and new frameworks emerging, it's an exciting time to be a developer.

AI is changing how we write code, but it's not replacing us. Instead, it's becoming a powerful assistant that helps us work faster and smarter. WASM is opening up new possibilities for high-performance applications in the browser, blurring the line between web and native apps.

Stay curious and keep learning!`,
    date: '3d ago',
    source: 'Dev.to',
    image: 'https://images.unsplash.com/photo-1504639725590-34d0984388bd?auto=format&fit=crop&w=800&q=80',
    featured: false
  },
  {
    id: 5,
    category: 'Life',
    title: 'Hiking Trip to the Alps',
    summary: 'Took some time off to recharge and explore the beautiful Swiss Alps.',
    content: `Sometimes you just need to disconnect and reconnect with nature. My recent trip to the Swiss Alps was exactly that. The fresh air, stunning views, and physical challenge of hiking were incredibly rejuvenating.

We hiked through lush valleys, climbed rocky peaks, and enjoyed some delicious local cheese and chocolate. It was a reminder of the importance of work-life balance and taking time for yourself.

Highly recommend visiting if you get the chance!`,
    date: '1w ago',
    source: 'Instagram',
    image: 'https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?auto=format&fit=crop&w=800&q=80',
    featured: false
  }
])

const filteredNews = computed(() => {
  if (selectedItem.value === 'top_stories') return newsItems.value
  if (selectedItem.value === 'spotlight') return newsItems.value.filter(i => i.featured)
  // Simple mapping for other categories
  const categoryMap = {
    'projects': 'Projects',
    'blog': 'Blog'
  }
  const category = categoryMap[selectedItem.value]
  if (category) {
    return newsItems.value.filter(i => i.category === category)
  }
  return newsItems.value
})

const handleMouseDown = (e) => {
  emit('focus')
  if (e.target.closest('.window-controls') || e.target.closest('.sidebar') || e.target.closest('.news-content')) return
  
  isDragging.value = true
  dragOffset.value = {
    x: e.clientX - position.value.x,
    y: e.clientY - position.value.y
  }
  
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('mouseup', handleMouseUp)
}

const handleMouseMove = (e) => {
  if (!isDragging.value) return
  if (isMaximized.value) return
  position.value = {
    x: e.clientX - dragOffset.value.x,
    y: e.clientY - dragOffset.value.y
  }
}

const handleMouseUp = () => {
  isDragging.value = false
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('mouseup', handleMouseUp)
}

const toggleMaximize = () => {
  if (isMaximized.value) {
    position.value = { ...preMaximizePosition.value }
    isMaximized.value = false
  } else {
    preMaximizePosition.value = { ...position.value }
    position.value = { x: 0, y: 0 }
    isMaximized.value = true
  }
}

const openArticle = (article) => {
  selectedArticle.value = article
}

const closeArticle = () => {
  selectedArticle.value = null
}
</script>

<template>
  <div 
    v-if="isOpen"
    class="news-window"
    :class="{ maximized: isMaximized }"
    :style="{ 
      top: position.y + 'px', 
      left: position.x + 'px',
      zIndex: zIndex
    }"
    @mousedown="handleMouseDown"
  >
    <div class="sidebar">
      <div class="window-controls">
        <button class="control-btn close" @click="$emit('close')"></button>
        <button class="control-btn minimize"></button>
        <button class="control-btn maximize" @click="toggleMaximize"></button>
      </div>
      
      <div class="sidebar-section">
        <div class="section-title">samacOS News</div>
        <div 
          v-for="item in sidebarItems" 
          :key="item.id"
          class="sidebar-item"
          :class="{ active: selectedItem === item.id }"
          @click="selectedItem = item.id"
        >
          <span class="item-icon">
            <svg v-if="item.icon === 'star'" viewBox="0 0 24 24" fill="currentColor"><path d="M12 17.27L18.18 21l-1.64-7.03L22 9.24l-7.19-.61L12 2 9.19 8.63 2 9.24l5.46 4.73L5.82 21z"/></svg>
            <svg v-if="item.icon === 'spotlight'" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-6h2v6zm0-8h-2V7h2v2z"/></svg>
            <svg v-if="item.icon === 'projects'" viewBox="0 0 24 24" fill="currentColor"><path d="M20 6h-8l-2-2H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2zm0 12H4V8h16v10z"/></svg>
            <svg v-if="item.icon === 'blog'" viewBox="0 0 24 24" fill="currentColor"><path d="M14 2H6c-1.1 0-1.99.9-1.99 2L4 20c0 1.1.89 2 1.99 2H18c1.1 0 2-.9 2-2V8l-6-6zm2 16H8v-2h8v2zm0-4H8v-2h8v2zm-3-5V3.5L18.5 9H13z"/></svg>
          </span>
          {{ item.label }}
        </div>
      </div>
    </div>

    <div class="main-content">
      <div class="toolbar">
        <div class="toolbar-left" v-if="selectedArticle">
          <button class="back-btn" @click="closeArticle">
            <svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor"><path d="M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z"/></svg>
            <span class="back-text">Back</span>
          </button>
        </div>
        <div class="toolbar-title">{{ selectedArticle ? '' : sidebarItems.find(i => i.id === selectedItem)?.label }}</div>
      </div>

      <div v-if="selectedArticle" class="article-view">
        <div class="article-view-content">
          <div class="article-header-image" v-if="selectedArticle.image">
            <img :src="selectedArticle.image" :alt="selectedArticle.title" />
          </div>
          <div class="article-body">
            <div class="article-meta-full">
              <span class="source-name-full">{{ selectedArticle.source }}</span>
              <span class="article-date-full">{{ selectedArticle.date }}</span>
            </div>
            <h1 class="article-title-full">{{ selectedArticle.title }}</h1>
            <div class="article-text">
              <p v-for="(paragraph, index) in selectedArticle.content.split('\n\n')" :key="index">
                {{ paragraph }}
              </p>
            </div>
          </div>
        </div>
      </div>

      <div v-else class="news-feed">
        <div class="feed-header">
          <h1>{{ sidebarItems.find(i => i.id === selectedItem)?.label }}</h1>
          <p class="date">{{ new Date().toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric' }) }}</p>
        </div>

        <div class="articles-grid">
          <div 
            v-for="article in filteredNews" 
            :key="article.id" 
            class="article-card"
            :class="{ 'featured': article.featured }"
            @click="openArticle(article)"
          >
            <div class="article-image" v-if="article.image">
              <img :src="article.image" :alt="article.title" />
            </div>
            <div class="article-content">
              <div class="article-meta">
                <span class="source-icon">
                    <img src="../assets/Icons/apple-logo.svg" width="12" height="12" v-if="article.source === 'Nemanja Samac'" />
                    <span v-else class="generic-source-icon">{{ article.source[0] }}</span>
                </span>
                <span class="source-name">{{ article.source }}</span>
              </div>
              <h3 class="article-title">{{ article.title }}</h3>
              <p class="article-summary">{{ article.summary }}</p>
              <div class="article-footer">
                <span class="article-time">{{ article.date }}</span>
                <button class="more-btn">
                    <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><circle cx="5" cy="12" r="2"/><circle cx="12" cy="12" r="2"/><circle cx="19" cy="12" r="2"/></svg>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.news-window {
  position: absolute;
  width: 950px;
  height: 650px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
  display: flex;
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 0.1);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.news-window.maximized {
  top: 0 !important;
  left: 0 !important;
  width: 100% !important;
  height: 100% !important;
  border-radius: 0;
}

.sidebar {
  width: 240px;
  background: #f2f2f7;
  border-right: 1px solid rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  padding-top: 10px;
}

.window-controls {
  display: flex;
  gap: 8px;
  padding: 10px 15px 20px;
}

.control-btn {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: none;
  cursor: pointer;
  padding: 0;
}

.close { background: #ff5f56; border: 1px solid #e0443e; }
.minimize { background: #ffbd2e; border: 1px solid #dea123; }
.maximize { background: #27c93f; border: 1px solid #1aab29; }

.sidebar-section {
  margin-bottom: 20px;
}

.section-title {
  padding: 0 15px 10px;
  font-size: 22px;
  font-weight: 700;
  color: #000;
  display: flex;
  align-items: center;
}

.sidebar-item {
  display: flex;
  align-items: center;
  padding: 8px 15px;
  font-size: 15px;
  color: #000;
  cursor: pointer;
  transition: background 0.2s;
  margin: 0 10px;
  border-radius: 6px;
}

.sidebar-item:hover {
  background: rgba(0, 0, 0, 0.05);
}

.sidebar-item.active {
  background: #fa2d48;
  color: white;
}

.item-icon {
  margin-right: 12px;
  display: flex;
  align-items: center;
  color: inherit;
}

.item-icon svg {
  width: 20px;
  height: 20px;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: #fff;
  overflow: hidden;
}

.toolbar {
  height: 50px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 20px;
  background: rgba(255,255,255,0.8);
  backdrop-filter: blur(10px);
  position: absolute;
  top: 0;
  left: 240px;
  right: 0;
  z-index: 10;
}

.toolbar-title {
  font-size: 15px;
  font-weight: 600;
  color: #333;
  opacity: 0; /* Hidden initially, could show on scroll */
}

.news-feed {
  flex: 1;
  padding: 70px 40px 40px;
  overflow-y: auto;
}

.feed-header {
  margin-bottom: 30px;
  border-bottom: 1px solid #eee;
  padding-bottom: 15px;
}

.feed-header h1 {
  font-size: 34px;
  font-weight: 800;
  margin: 0 0 5px 0;
  color: #111;
}

.feed-header .date {
  font-size: 13px;
  color: #888;
  font-weight: 600;
  text-transform: uppercase;
  margin: 0;
}

.articles-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 25px;
}

.article-card {
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
  border: 1px solid rgba(0,0,0,0.05);
  transition: transform 0.2s, box-shadow 0.2s;
  display: flex;
  flex-direction: column;
  cursor: pointer;
}

.article-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
}

.article-card.featured {
  grid-column: 1 / -1;
  flex-direction: row;
  min-height: 300px;
}

.article-image {
  height: 200px;
  overflow: hidden;
}

.article-card.featured .article-image {
  height: auto;
  width: 60%;
  flex-shrink: 0;
}

.article-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.article-content {
  padding: 20px;
  display: flex;
  flex-direction: column;
  flex: 1;
}

.article-meta {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.source-icon {
  width: 20px;
  height: 20px;
  border-radius: 4px;
  background: #eee;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 8px;
  overflow: hidden;
}

.generic-source-icon {
    font-size: 10px;
    font-weight: bold;
    color: #555;
}

.source-name {
  font-size: 11px;
  font-weight: 700;
  color: #fa2d48;
  text-transform: uppercase;
}

.article-title {
  font-size: 18px;
  font-weight: 700;
  line-height: 1.3;
  margin: 0 0 10px 0;
  color: #111;
}

.article-card.featured .article-title {
  font-size: 28px;
}

.article-summary {
  font-size: 14px;
  line-height: 1.5;
  color: #666;
  margin: 0 0 15px 0;
  flex: 1;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.article-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: auto;
}

.article-time {
  font-size: 12px;
  color: #999;
}

.more-btn {
  background: none;
  border: none;
  color: #999;
  cursor: pointer;
  padding: 4px;
}

.more-btn:hover {
  color: #333;
}

@media (max-width: 768px) {
  .article-card.featured {
    flex-direction: column;
  }
  .article-card.featured .article-image {
    width: 100%;
    height: 250px;
  }
}

.toolbar-left {
  position: absolute;
  left: 20px;
  display: flex;
  align-items: center;
}

.back-btn {
  background: none;
  border: none;
  cursor: pointer;
  color: #fa2d48;
  padding: 5px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  font-weight: 500;
}

.back-btn:hover {
  opacity: 0.8;
}

.back-text {
  margin-left: 4px;
}

.article-view {
  flex: 1;
  overflow-y: auto;
  background: #fff;
}

.article-view-content {
  max-width: 800px;
  margin: 0 auto;
  padding-bottom: 60px;
}

.article-header-image {
  width: 100%;
  height: 400px;
  overflow: hidden;
}

.article-header-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.article-body {
  padding: 40px;
}

.article-meta-full {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 15px;
}

.source-name-full {
  font-weight: 700;
  color: #fa2d48;
  text-transform: uppercase;
  font-size: 14px;
}

.article-date-full {
  color: #888;
  font-size: 14px;
}

.article-title-full {
  font-size: 42px;
  font-weight: 800;
  line-height: 1.1;
  margin: 0 0 30px 0;
  color: #111;
  font-family: "New York", "Times New Roman", serif;
}

.article-text {
  font-size: 18px;
  line-height: 1.6;
  color: #333;
  font-family: "New York", "Times New Roman", serif;
}

.article-text p {
  margin-bottom: 20px;
}
</style>
