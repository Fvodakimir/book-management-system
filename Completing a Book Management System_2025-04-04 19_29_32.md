### Completing a Book Management System

用户:
请帮我补全这个不完整的项目，像文件里的Claude一样做。

Claude:
I'll help you complete the unfinished project. Based on the files provided, this appears to be a Java and MySQL-based book management system with a Vue.js frontend. Let me continue where the documentation left off.

Looking at the provided files, I see that we have:
1. The entity classes for the backend
2. Service implementations for various functionalities
3. Controller implementations for RESTful APIs
4. Started on the Vue.js frontend with store modules
5. The last component shown is the Pagination component, which is incomplete

Let me first complete the Pagination component and then continue with the remaining necessary components and implementations.

## Completing the Pagination component (src/components/Pagination.vue)

```vue
<template>
  <div class="pagination-container">
    <el-pagination
      background
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      :page-size="pageSize"
      :current-page="currentPage"
      :page-sizes="[10, 20, 30, 50]"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'Pagination',
  props: {
    total: {
      type: Number,
      required: true
    },
    pageSize: {
      type: Number,
      default: 10
    },
    currentPage: {
      type: Number,
      default: 1
    }
  },
  emits: ['update:pageSize', 'update:currentPage', 'pagination'],
  setup(props, { emit }) {
    const handleSizeChange = (val) => {
      emit('update:pageSize', val);
      emit('pagination', { page: props.currentPage - 1, size: val });
    };

    const handleCurrentChange = (val) => {
      emit('update:currentPage', val);
      emit('pagination', { page: val - 1, size: props.pageSize });
    };

    return {
      handleSizeChange,
      handleCurrentChange
    };
  }
});
</script>

<style scoped>
.pagination-container {
  margin-top: 20px;
  text-align: center;
}
</style>
```

## Creating Main App Component (src/App.vue)

```vue
<template>
  <div id="app">
    <AppHeader />
    <main class="main-content">
      <router-view />
    </main>
    <AppFooter />
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import AppHeader from '@/components/AppHeader.vue';
import AppFooter from '@/components/AppFooter.vue';

export default defineComponent({
  name: 'App',
  components: {
    AppHeader,
    AppFooter
  }
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

html, body {
  margin: 0;
  padding: 0;
  height: 100%;
}
</style>
```

## Updating main.js (src/main.js)

```javascript
import { createApp } from 'vue';
import ElementPlus from 'element-plus';
import 'element-plus/dist/index.css';
import * as ElementPlusIconsVue from '@element-plus/icons-vue';
import App from './App.vue';
import router from './router';
import store from './store';

const app = createApp(App);

// Register all Element Plus icons
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component);
}

app.use(store)
   .use(router)
   .use(ElementPlus, { size: 'medium' })
   .mount('#app');
```

## Creating Home Page (src/views/Home.vue)

```vue
<template>
  <div class="home">
    <el-carousel height="400px" class="carousel">
      <el-carousel-item v-for="book in popularBooks" :key="book.id">
        <div class="carousel-item" @click="navigateToBook(book.id)">
          <img :src="book.coverImage || defaultCover" :alt="book.title" class="carousel-image">
          <div class="carousel-caption">
            <h2>{{ book.title }}</h2>
            <p>{{ book.author }}</p>
          </div>
        </div>
      </el-carousel-item>
    </el-carousel>

    <el-divider>
      <el-icon><star-filled /></el-icon>
      <span>推荐图书</span>
    </el-divider>

    <el-row :gutter="20">
      <el-col :xs="24" :sm="12" :md="8" :lg="6" v-for="book in recommendedBooks" :key="book.id">
        <BookCard :book="book" />
      </el-col>
    </el-row>

    <el-divider>
      <el-icon><reading /></el-icon>
      <span>最近浏览</span>
    </el-divider>

    <el-row :gutter="20">
      <el-col :xs="24" :sm="12" :md="8" :lg="6" v-for="book in recentlyViewed" :key="book.id">
        <BookCard :book="book" />
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { useRouter } from 'vue-router';
import BookCard from '@/components/BookCard.vue';
import defaultCover from '@/assets/default-book-cover.jpg';

export default defineComponent({
  name: 'Home',
  components: {
    BookCard
  },
  setup() {
    const store = useStore();
    const router = useRouter();
    
    const defaultCover = ref(defaultCover);
    
    const popularBooks = computed(() => store.getters['book/popularBooks']);
    const recommendedBooks = computed(() => store.getters['book/recommendedBooks']);
    const recentlyViewed = computed(() => store.getters['history/recentlyViewed']);
    
    onMounted(() => {
      store.dispatch('book/fetchPopularBooks', 5);
      store.dispatch('book/fetchRecommendedBooks', 8);
      
      if (store.getters['auth/isLoggedIn']) {
        store.dispatch('history/fetchRecentlyViewedBooks', 8);
      }
    });
    
    const navigateToBook = (bookId) => {
      router.push({ name: 'BookDetail', params: { id: bookId } });
    };
    
    return {
      popularBooks,
      recommendedBooks,
      recentlyViewed,
      defaultCover,
      navigateToBook
    };
  }
});
</script>

<style scoped>
.home {
  margin-bottom: 40px;
}

.carousel {
  margin-bottom: 40px;
  border-radius: 8px;
  overflow: hidden;
}

.carousel-item {
  height: 100%;
  width: 100%;
  position: relative;
  cursor: pointer;
}

.carousel-image {
  height: 100%;
  width: 100%;
  object-fit: cover;
}

.carousel-caption {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  padding: 20px;
}

.carousel-caption h2 {
  margin: 0;
  font-size: 24px;
}

.carousel-caption p {
  margin: 5px 0 0;
  font-size: 16px;
}

.el-divider {
  margin: 40px 0;
}
</style>
```

## Creating Login Page (src/views/auth/Login.vue)

```vue
<template>
  <div class="login-container">
    <el-card class="login-card">
      <template #header>
        <div class="card-header">
          <h2>登录</h2>
        </div>
      </template>
      
      <el-form 
        ref="loginForm" 
        :model="loginForm" 
        :rules="rules" 
        label-position="top"
      >
        <el-form-item label="用户名" prop="username">
          <el-input 
            v-model="loginForm.username" 
            prefix-icon="User"
            placeholder="请输入用户名"
          />
        </el-form-item>
        
        <el-form-item label="密码" prop="password">
          <el-input 
            v-model="loginForm.password" 
            type="password" 
            prefix-icon="Lock"
            placeholder="请输入密码" 
            show-password
          />
        </el-form-item>
        
        <el-form-item>
          <el-button 
            type="primary" 
            :loading="loading" 
            @click="handleLogin" 
            class="login-button"
          >
            登录
          </el-button>
        </el-form-item>
      </el-form>
      
      <div class="login-links">
        <p>
          还没有账号？
          <router-link to="/register">立即注册</router-link>
        </p>
      </div>
    </el-card>
  </div>
</template>

<script>
import { defineComponent, reactive, ref, computed } from 'vue';
import { useStore } from 'vuex';
import { useRouter, useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';

export default defineComponent({
  name: 'Login',
  setup() {
    const store = useStore();
    const router = useRouter();
    const route = useRoute();
    
    const loginForm = ref(null);
    const loading = ref(false);
    
    const formData = reactive({
      username: '',
      password: ''
    });
    
    const rules = {
      username: [
        { required: true, message: '请输入用户名', trigger: 'blur' }
      ],
      password: [
        { required: true, message: '请输入密码', trigger: 'blur' }
      ]
    };
    
    const redirectPath = computed(() => route.query.redirect || '/');
    
    const handleLogin = async () => {
      await loginForm.value.validate(async (valid) => {
        if (valid) {
          try {
            loading.value = true;
            await store.dispatch('auth/login', {
              username: formData.username,
              password: formData.password
            });
            
            ElMessage.success('登录成功');
            router.push(redirectPath.value);
          } catch (error) {
            let errorMessage = '登录失败，请检查用户名和密码';
            
            if (error.response && error.response.data && error.response.data.message) {
              errorMessage = error.response.data.message;
            }
            
            ElMessage.error(errorMessage);
          } finally {
            loading.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    return {
      loginForm,
      loginForm: formData,
      rules,
      loading,
      handleLogin
    };
  }
});
</script>

<style scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 80vh;
}

.login-card {
  width: 400px;
  max-width: 100%;
}

.card-header {
  text-align: center;
}

.login-button {
  width: 100%;
}

.login-links {
  text-align: center;
  margin-top: 20px;
}
</style>
```

## Creating Register Page (src/views/auth/Register.vue)

```vue
<template>
  <div class="register-container">
    <el-card class="register-card">
      <template #header>
        <div class="card-header">
          <h2>注册</h2>
        </div>
      </template>
      
      <el-form 
        ref="registerForm" 
        :model="registerForm" 
        :rules="rules" 
        label-position="top"
      >
        <el-form-item label="用户名" prop="username">
          <el-input 
            v-model="registerForm.username" 
            prefix-icon="User"
            placeholder="请输入用户名"
          />
        </el-form-item>
        
        <el-form-item label="邮箱" prop="email">
          <el-input 
            v-model="registerForm.email" 
            prefix-icon="Message"
            placeholder="请输入邮箱"
          />
        </el-form-item>
        
        <el-form-item label="密码" prop="password">
          <el-input 
            v-model="registerForm.password" 
            type="password" 
            prefix-icon="Lock"
            placeholder="请输入密码" 
            show-password
          />
        </el-form-item>
        
        <el-form-item label="确认密码" prop="confirmPassword">
          <el-input 
            v-model="registerForm.confirmPassword" 
            type="password" 
            prefix-icon="Lock"
            placeholder="请再次输入密码" 
            show-password
          />
        </el-form-item>
        
        <el-form-item>
          <el-button 
            type="primary" 
            :loading="loading" 
            @click="handleRegister" 
            class="register-button"
          >
            注册
          </el-button>
        </el-form-item>
      </el-form>
      
      <div class="register-links">
        <p>
          已有账号？
          <router-link to="/login">立即登录</router-link>
        </p>
      </div>
    </el-card>
  </div>
</template>

<script>
import { defineComponent, reactive, ref } from 'vue';
import { useStore } from 'vuex';
import { useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';

export default defineComponent({
  name: 'Register',
  setup() {
    const store = useStore();
    const router = useRouter();
    
    const registerForm = ref(null);
    const loading = ref(false);
    
    const formData = reactive({
      username: '',
      email: '',
      password: '',
      confirmPassword: ''
    });
    
    const validatePassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'));
      } else if (value.length < 8) {
        callback(new Error('密码长度不能小于8位'));
      } else if (!/^(?=.*[a-zA-Z])(?=.*\d).+$/.test(value)) {
        callback(new Error('密码必须包含字母和数字'));
      } else {
        callback();
      }
    };
    
    const validateConfirmPassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'));
      } else if (value !== formData.password) {
        callback(new Error('两次输入密码不一致'));
      } else {
        callback();
      }
    };
    
    const rules = {
      username: [
        { required: true, message: '请输入用户名', trigger: 'blur' },
        { min: 3, max: 20, message: '用户名长度在3到20个字符之间', trigger: 'blur' }
      ],
      email: [
        { required: true, message: '请输入邮箱', trigger: 'blur' },
        { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
      ],
      password: [
        { required: true, trigger: 'blur', validator: validatePassword }
      ],
      confirmPassword: [
        { required: true, trigger: 'blur', validator: validateConfirmPassword }
      ]
    };
    
    const handleRegister = async () => {
      await registerForm.value.validate(async (valid) => {
        if (valid) {
          try {
            loading.value = true;
            await store.dispatch('auth/register', {
              username: formData.username,
              email: formData.email,
              password: formData.password
            });
            
            ElMessage.success('注册成功，请登录');
            router.push('/login');
          } catch (error) {
            let errorMessage = '注册失败，请稍后重试';
            
            if (error.response && error.response.data && error.response.data.message) {
              errorMessage = error.response.data.message;
            }
            
            ElMessage.error(errorMessage);
          } finally {
            loading.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    return {
      registerForm,
      registerForm: formData,
      rules,
      loading,
      handleRegister
    };
  }
});
</script>

<style scoped>
.register-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 80vh;
}

.register-card {
  width: 400px;
  max-width: 100%;
}

.card-header {
  text-align: center;
}

.register-button {
  width: 100%;
}

.register-links {
  text-align: center;
  margin-top: 20px;
}
</style>
```

## Creating Book List Page (src/views/book/BookList.vue)

```vue
<template>
  <div class="book-list">
    <h1 class="page-title">图书列表</h1>
    
    <div class="filter-container">
      <el-select 
        v-model="sortBy" 
        placeholder="排序方式" 
        @change="handleSortChange"
      >
        <el-option label="ID" value="id" />
        <el-option label="书名" value="title" />
        <el-option label="作者" value="author" />
        <el-option label="出版日期" value="publishDate" />
      </el-select>
      
      <el-select 
        v-model="sortDir" 
        placeholder="排序方向" 
        @change="handleSortChange"
      >
        <el-option label="升序" value="asc" />
        <el-option label="降序" value="desc" />
      </el-select>
    </div>
    
    <el-row :gutter="20">
      <el-col 
        :xs="24" 
        :sm="12" 
        :md="8" 
        :lg="6" 
        v-for="book in books" 
        :key="book.id"
      >
        <BookCard :book="book" />
      </el-col>
    </el-row>
    
    <el-empty v-if="books.length === 0" description="暂无数据" />
    
    <Pagination
      v-if="books.length > 0"
      :total="total"
      :page-size.sync="pageSize"
      :current-page.sync="currentPage"
      @pagination="handlePagination"
    />
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import BookCard from '@/components/BookCard.vue';
import Pagination from '@/components/Pagination.vue';

export default defineComponent({
  name: 'BookList',
  components: {
    BookCard,
    Pagination
  },
  setup() {
    const store = useStore();
    
    const currentPage = ref(1);
    const pageSize = ref(12);
    const sortBy = ref('id');
    const sortDir = ref('asc');
    
    const books = computed(() => store.getters['book/allBooks']);
    const pagination = computed(() => store.getters['book/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['book/loading']);
    
    onMounted(() => {
      fetchBooks();
    });
    
    const fetchBooks = () => {
      store.dispatch('book/fetchAllBooks', {
        page: currentPage.value - 1,
        size: pageSize.value,
        sortBy: sortBy.value,
        sortDir: sortDir.value
      });
    };
    
    const handlePagination = ({ page, size }) => {
      currentPage.value = page + 1;
      pageSize.value = size;
      fetchBooks();
    };
    
    const handleSortChange = () => {
      currentPage.value = 1;
      fetchBooks();
    };
    
    return {
      books,
      total,
      loading,
      currentPage,
      pageSize,
      sortBy,
      sortDir,
      handlePagination,
      handleSortChange
    };
  }
});
</script>

<style scoped>
.book-list {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.filter-container {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20px;
  gap: 10px;
}
</style>
```

## Creating Book Detail Page (src/views/book/BookDetail.vue)

```vue
<template>
  <div class="book-detail" v-loading="loading">
    <div v-if="book" class="book-container">
      <div class="book-cover">
        <img :src="coverUrl" :alt="book.title" />
        
        <div class="book-actions">
          <el-button 
            type="primary" 
            :icon="book.favorite ? 'StarFilled' : 'Star'"
            @click="toggleFavorite"
          >
            {{ book.favorite ? '已收藏' : '收藏' }}
          </el-button>
        </div>
      </div>
      
      <div class="book-info">
        <h1 class="book-title">{{ book.title }}</h1>
        
        <p class="book-meta">
          <span class="label">作者：</span>{{ book.author }}
        </p>
        
        <p class="book-meta">
          <span class="label">ISBN：</span>{{ book.isbn }}
        </p>
        
        <p class="book-meta">
          <span class="label">出版社：</span>{{ book.publisher }}
        </p>
        
        <p class="book-meta">
          <span class="label">出版日期：</span>{{ formatDate(book.publishDate) }}
        </p>
        
        <p class="book-meta">
          <span class="label">分类：</span>
          <el-tag size="small">{{ book.categoryName }}</el-tag>
        </p>
        
        <div class="book-description">
          <h3>图书简介</h3>
          <p>{{ book.description || '暂无简介' }}</p>
        </div>
      </div>
    </div>
    
    <el-empty v-if="!loading && !book" description="图书不存在" />
    
    <div v-if="book" class="book-recommendations">
      <h2>相关推荐</h2>
      
      <el-row :gutter="20">
        <el-col 
          :xs="24" 
          :sm="12" 
          :md="8" 
          :lg="6" 
          v-for="book in recommendedBooks" 
          :key="book.id"
        >
          <BookCard :book="book" />
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { useRoute, useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';
import BookCard from '@/components/BookCard.vue';
import defaultCover from '@/assets/default-book-cover.jpg';

export default defineComponent({
  name: 'BookDetail',
  components: {
    BookCard
  },
  setup() {
    const store = useStore();
    const route = useRoute();
    const router = useRouter();
    
    const bookId = computed(() => route.params.id);
    const book = computed(() => store.getters['book/currentBook']);
    const recommendedBooks = computed(() => store.getters['book/recommendedBooks']);
    const loading = computed(() => store.getters['book/loading']);
    
    const coverUrl = computed(() => {
      return book.value?.coverImage || defaultCover;
    });
    
    onMounted(async () => {
      try {
        await store.dispatch('book/fetchBookById', bookId.value);
        
        // 获取推荐图书
        store.dispatch('book/fetchRecommendedBooks', 4);
      } catch (error) {
        ElMessage.error('获取图书信息失败');
      }
    });
    
    const formatDate = (dateString) => {
      if (!dateString) return '';
      
      const date = new Date(dateString);
      return date.toLocaleDateString();
    };
    
    const toggleFavorite = async () => {
      try {
        const isLoggedIn = store.getters['auth/isLoggedIn'];
        
        if (!isLoggedIn) {
          ElMessage.warning('请先登录');
          router.push({ 
            name: 'Login', 
            query: { redirect: `/books/${bookId.value}` } 
          });
          return;
        }
        
        const isFavorite = await store.dispatch('book/toggleFavorite', bookId.value);
        ElMessage.success(isFavorite ? '添加收藏成功' : '取消收藏成功');
      } catch (error) {
        ElMessage.error('操作失败，请重试');
      }
    };
    
    return {
      book,
      recommendedBooks,
      loading,
      coverUrl,
      formatDate,
      toggleFavorite
    };
  }
});
</script>

<style scoped>
.book-detail {
  margin-bottom: 40px;
}

.book-container {
  display: flex;
  gap: 30px;
  margin-bottom: 40px;
}

@media (max-width: 768px) {
  .book-container {
    flex-direction: column;
  }
}

.book-cover {
  flex: 0 0 300px;
  display: flex;
  flex-direction: column;
}

.book-cover img {
  width: 100%;
  height: auto;
  max-height: 400px;
  object-fit: contain;
  margin-bottom: 20px;
}

.book-actions {
  display: flex;
  justify-content: center;
}

.book-info {
  flex: 1;
}

.book-title {
  margin-top: 0;
  margin-bottom: 20px;
  font-size: 24px;
}

.book-meta {
  margin: 10px 0;
  font-size: 16px;
}

.label {
  font-weight: bold;
  margin-right: 5px;
}

.book-description {
  margin-top: 20px;
}

.book-description h3 {
  margin-bottom: 10px;
}

.book-recommendations {
  margin-top: 40px;
}

.book-recommendations h2 {
  margin-bottom: 20px;
}
</style>
```

## Creating Book Search Page (src/views/book/BookSearch.vue)

```vue
<template>
  <div class="book-search">
    <h1 class="page-title">搜索图书</h1>
    
    <div class="search-form">
      <el-form :model="searchForm" :inline="true">
        <el-form-item label="关键词">
          <el-input
            v-model="searchForm.keyword"
            placeholder="书名关键词"
            clearable
            @keyup.enter="handleSearch"
          />
        </el-form-item>
        
        <el-form-item label="作者">
          <el-input
            v-model="searchForm.author"
            placeholder="作者名"
            clearable
            @keyup.enter="handleSearch"
          />
        </el-form-item>
        
        <el-form-item label="ISBN">
          <el-input
            v-model="searchForm.isbn"
            placeholder="ISBN编码"
            clearable
            @keyup.enter="handleSearch"
          />
        </el-form-item>
        
        <el-form-item label="分类">
          <el-select
            v-model="searchForm.categoryId"
            placeholder="选择分类"
            clearable
          >
            <el-option
              v-for="category in categories"
              :key="category.id"
              :label="category.name"
              :value="category.id"
            />
          </el-select>
        </el-form-item>
        
        <el-form-item>
          <el-button type="primary" @click="handleSearch" :loading="loading">
            <el-icon><search /></el-icon>
            搜索
          </el-button>
          <el-button @click="resetForm">
            <el-icon><refresh-right /></el-icon>
            重置
          </el-button>
        </el-form-item>
      </el-form>
    </div>
    
    <div class="search-results">
      <h2 v-if="hasSearched">搜索结果 ({{ total }})</h2>
      
      <el-row :gutter="20">
        <el-col
          :xs="24"
          :sm="12"
          :md="8"
          :lg="6"
          v-for="book in books"
          :key="book.id"
        >
          <BookCard :book="book" />
        </el-col>
      </el-row>
      
      <el-empty
        v-if="hasSearched && books.length === 0"
        description="未找到


用户:
continue

Claude:
Let me continue with the Book Search Page (src/views/book/BookSearch.vue) implementation:

```vue
<template>
  <div class="book-search">
    <h1 class="page-title">搜索图书</h1>
    
    <div class="search-form">
      <el-form :model="searchForm" :inline="true">
        <el-form-item label="关键词">
          <el-input
            v-model="searchForm.keyword"
            placeholder="书名关键词"
            clearable
            @keyup.enter="handleSearch"
          />
        </el-form-item>
        
        <el-form-item label="作者">
          <el-input
            v-model="searchForm.author"
            placeholder="作者名"
            clearable
            @keyup.enter="handleSearch"
          />
        </el-form-item>
        
        <el-form-item label="ISBN">
          <el-input
            v-model="searchForm.isbn"
            placeholder="ISBN编码"
            clearable
            @keyup.enter="handleSearch"
          />
        </el-form-item>
        
        <el-form-item label="分类">
          <el-select
            v-model="searchForm.categoryId"
            placeholder="选择分类"
            clearable
          >
            <el-option
              v-for="category in categories"
              :key="category.id"
              :label="category.name"
              :value="category.id"
            />
          </el-select>
        </el-form-item>
        
        <el-form-item>
          <el-button type="primary" @click="handleSearch" :loading="loading">
            <el-icon><search /></el-icon>
            搜索
          </el-button>
          <el-button @click="resetForm">
            <el-icon><refresh-right /></el-icon>
            重置
          </el-button>
        </el-form-item>
      </el-form>
    </div>
    
    <div class="search-results">
      <h2 v-if="hasSearched">搜索结果 ({{ total }})</h2>
      
      <el-row :gutter="20">
        <el-col
          :xs="24"
          :sm="12"
          :md="8"
          :lg="6"
          v-for="book in books"
          :key="book.id"
        >
          <BookCard :book="book" />
        </el-col>
      </el-row>
      
      <el-empty
        v-if="hasSearched && books.length === 0"
        description="未找到匹配的图书"
      />
      
      <Pagination
        v-if="hasSearched && books.length > 0"
        :total="total"
        :page-size.sync="pageSize"
        :current-page.sync="currentPage"
        @pagination="handlePagination"
      />
    </div>
  </div>
</template>

<script>
import { defineComponent, ref, computed, reactive, onMounted } from 'vue';
import { useStore } from 'vuex';
import { useRoute, useRouter } from 'vue-router';
import BookCard from '@/components/BookCard.vue';
import Pagination from '@/components/Pagination.vue';

export default defineComponent({
  name: 'BookSearch',
  components: {
    BookCard,
    Pagination
  },
  setup() {
    const store = useStore();
    const route = useRoute();
    const router = useRouter();
    
    const currentPage = ref(1);
    const pageSize = ref(12);
    const hasSearched = ref(false);
    
    const searchForm = reactive({
      keyword: route.query.keyword || '',
      author: route.query.author || '',
      isbn: route.query.isbn || '',
      categoryId: route.query.categoryId ? Number(route.query.categoryId) : null,
      sortBy: 'title',
      sortDir: 'asc'
    });
    
    const books = computed(() => store.getters['book/allBooks']);
    const pagination = computed(() => store.getters['book/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['book/loading']);
    const categories = computed(() => store.getters['category/allCategories']);
    
    onMounted(() => {
      // 加载分类列表
      store.dispatch('category/fetchAllCategories');
      
      // 如果URL中包含查询参数，执行搜索
      if (route.query.keyword || route.query.author || route.query.isbn || route.query.categoryId) {
        handleSearch();
      }
    });
    
    const handleSearch = () => {
      hasSearched.value = true;
      currentPage.value = 1;
      performSearch();
      
      // 更新URL中的查询参数
      updateQueryParams();
    };
    
    const handlePagination = ({ page, size }) => {
      currentPage.value = page + 1;
      pageSize.value = size;
      performSearch();
    };
    
    const performSearch = () => {
      store.dispatch('book/searchBooks', {
        keyword: searchForm.keyword,
        author: searchForm.author,
        isbn: searchForm.isbn,
        categoryId: searchForm.categoryId,
        page: currentPage.value - 1,
        size: pageSize.value,
        sortBy: searchForm.sortBy,
        sortDir: searchForm.sortDir
      });
    };
    
    const resetForm = () => {
      Object.keys(searchForm).forEach(key => {
        if (key !== 'sortBy' && key !== 'sortDir') {
          searchForm[key] = '';
        }
      });
      searchForm.categoryId = null;
      
      // 清除URL中的查询参数
      router.push({ path: '/search' });
      
      // 重置搜索状态
      hasSearched.value = false;
    };
    
    const updateQueryParams = () => {
      const query = {};
      if (searchForm.keyword) query.keyword = searchForm.keyword;
      if (searchForm.author) query.author = searchForm.author;
      if (searchForm.isbn) query.isbn = searchForm.isbn;
      if (searchForm.categoryId) query.categoryId = searchForm.categoryId;
      
      router.push({ path: '/search', query });
    };
    
    return {
      books,
      total,
      loading,
      currentPage,
      pageSize,
      searchForm,
      categories,
      hasSearched,
      handleSearch,
      handlePagination,
      resetForm
    };
  }
});
</script>

<style scoped>
.book-search {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.search-form {
  margin-bottom: 30px;
  padding: 20px;
  background-color: #f5f7fa;
  border-radius: 8px;
}

.search-results h2 {
  margin-bottom: 20px;
}
</style>
```

Now, let me create the Category Books Page (src/views/book/CategoryBooks.vue):

```vue
<template>
  <div class="category-books">
    <div v-if="category" class="category-header">
      <h1 class="page-title">{{ category.name }}</h1>
      <p class="category-description" v-if="category.description">{{ category.description }}</p>
    </div>
    
    <div class="filter-container">
      <el-select 
        v-model="sortBy" 
        placeholder="排序方式" 
        @change="handleSortChange"
      >
        <el-option label="ID" value="id" />
        <el-option label="书名" value="title" />
        <el-option label="作者" value="author" />
        <el-option label="出版日期" value="publishDate" />
      </el-select>
      
      <el-select 
        v-model="sortDir" 
        placeholder="排序方向" 
        @change="handleSortChange"
      >
        <el-option label="升序" value="asc" />
        <el-option label="降序" value="desc" />
      </el-select>
    </div>
    
    <el-row :gutter="20">
      <el-col 
        :xs="24" 
        :sm="12" 
        :md="8" 
        :lg="6" 
        v-for="book in books" 
        :key="book.id"
      >
        <BookCard :book="book" />
      </el-col>
    </el-row>
    
    <el-empty v-if="books.length === 0" description="暂无图书" />
    
    <Pagination
      v-if="books.length > 0"
      :total="total"
      :page-size.sync="pageSize"
      :current-page.sync="currentPage"
      @pagination="handlePagination"
    />
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted, watch } from 'vue';
import { useStore } from 'vuex';
import { useRoute } from 'vue-router';
import BookCard from '@/components/BookCard.vue';
import Pagination from '@/components/Pagination.vue';

export default defineComponent({
  name: 'CategoryBooks',
  components: {
    BookCard,
    Pagination
  },
  props: {
    id: {
      type: [String, Number],
      required: true
    }
  },
  setup(props) {
    const store = useStore();
    const route = useRoute();
    
    const currentPage = ref(1);
    const pageSize = ref(12);
    const sortBy = ref('id');
    const sortDir = ref('asc');
    
    const categoryId = computed(() => Number(props.id));
    const category = computed(() => store.getters['category/currentCategory']);
    const books = computed(() => store.getters['book/allBooks']);
    const pagination = computed(() => store.getters['book/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['book/loading']);
    
    // 监听分类ID变化，重新加载数据
    watch(() => categoryId.value, () => {
      currentPage.value = 1;
      fetchCategoryAndBooks();
    });
    
    onMounted(() => {
      fetchCategoryAndBooks();
    });
    
    const fetchCategoryAndBooks = async () => {
      try {
        // 获取分类信息
        await store.dispatch('category/fetchCategoryById', categoryId.value);
        
        // 获取分类下的图书
        fetchBooks();
      } catch (error) {
        console.error('Failed to fetch category or books:', error);
      }
    };
    
    const fetchBooks = () => {
      store.dispatch('book/fetchBooksByCategory', {
        categoryId: categoryId.value,
        page: currentPage.value - 1,
        size: pageSize.value,
        sortBy: sortBy.value,
        sortDir: sortDir.value
      });
    };
    
    const handlePagination = ({ page, size }) => {
      currentPage.value = page + 1;
      pageSize.value = size;
      fetchBooks();
    };
    
    const handleSortChange = () => {
      currentPage.value = 1;
      fetchBooks();
    };
    
    return {
      category,
      books,
      total,
      loading,
      currentPage,
      pageSize,
      sortBy,
      sortDir,
      handlePagination,
      handleSortChange
    };
  }
});
</script>

<style scoped>
.category-books {
  margin-bottom: 40px;
}

.category-header {
  margin-bottom: 20px;
}

.page-title {
  margin-bottom: 10px;
}

.category-description {
  color: #606266;
  margin-bottom: 20px;
}

.filter-container {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20px;
  gap: 10px;
}
</style>
```

Next, let's create the User Profile Page (src/views/user/UserProfile.vue):

```vue
<template>
  <div class="user-profile">
    <h1 class="page-title">个人中心</h1>
    
    <el-card class="profile-card">
      <div class="profile-header">
        <div class="avatar-container">
          <el-avatar :size="100" :src="userAvatar"></el-avatar>
          <el-button
            type="primary"
            size="small"
            class="upload-button"
            @click="triggerAvatarUpload"
          >
            更换头像
          </el-button>
          <input
            type="file"
            ref="avatarInput"
            accept="image/*"
            style="display: none"
            @change="handleAvatarChange"
          />
        </div>
        
        <div class="user-info">
          <h2>{{ user.username }}</h2>
          <p>{{ user.email }}</p>
          <p>用户角色: {{ userRole }}</p>
        </div>
      </div>
      
      <el-divider content-position="left">个人资料</el-divider>
      
      <el-form
        ref="profileForm"
        :model="profileForm"
        :rules="profileRules"
        label-width="100px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="profileForm.username" />
        </el-form-item>
        
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="profileForm.email" />
        </el-form-item>
        
        <el-form-item>
          <el-button type="primary" @click="updateProfile" :loading="loading">
            保存修改
          </el-button>
        </el-form-item>
      </el-form>
      
      <el-divider content-position="left">修改密码</el-divider>
      
      <el-form
        ref="passwordForm"
        :model="passwordForm"
        :rules="passwordRules"
        label-width="100px"
      >
        <el-form-item label="当前密码" prop="currentPassword">
          <el-input
            v-model="passwordForm.currentPassword"
            type="password"
            show-password
          />
        </el-form-item>
        
        <el-form-item label="新密码" prop="newPassword">
          <el-input
            v-model="passwordForm.newPassword"
            type="password"
            show-password
          />
        </el-form-item>
        
        <el-form-item label="确认新密码" prop="confirmPassword">
          <el-input
            v-model="passwordForm.confirmPassword"
            type="password"
            show-password
          />
        </el-form-item>
        
        <el-form-item>
          <el-button type="primary" @click="changePassword" :loading="loading">
            修改密码
          </el-button>
        </el-form-item>
      </el-form>
      
      <el-divider content-position="left">账户操作</el-divider>
      
      <div class="account-actions">
        <el-popconfirm
          title="确定要注销您的账户吗？此操作不可恢复！"
          @confirm="deleteAccount"
        >
          <template #reference>
            <el-button type="danger">注销账户</el-button>
          </template>
        </el-popconfirm>
      </div>
    </el-card>
  </div>
</template>

<script>
import { defineComponent, ref, reactive, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';
import defaultAvatar from '@/assets/default-avatar.png';

export default defineComponent({
  name: 'UserProfile',
  setup() {
    const store = useStore();
    const router = useRouter();
    
    const profileForm = ref(null);
    const passwordForm = ref(null);
    const avatarInput = ref(null);
    const loading = ref(false);
    
    const user = computed(() => store.getters['auth/currentUser'] || {});
    
    const userAvatar = computed(() => {
      return user.value.avatar || defaultAvatar;
    });
    
    const userRole = computed(() => {
      if (user.value.role === 'ROLE_ADMIN') {
        return '管理员';
      }
      return '普通用户';
    });
    
    const profileFormData = reactive({
      username: '',
      email: ''
    });
    
    const passwordFormData = reactive({
      currentPassword: '',
      newPassword: '',
      confirmPassword: ''
    });
    
    // 验证密码复杂度
    const validatePassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'));
      } else if (value.length < 8) {
        callback(new Error('密码长度不能小于8位'));
      } else if (!/^(?=.*[a-zA-Z])(?=.*\d).+$/.test(value)) {
        callback(new Error('密码必须包含字母和数字'));
      } else {
        callback();
      }
    };
    
    // 验证确认密码
    const validateConfirmPassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'));
      } else if (value !== passwordFormData.newPassword) {
        callback(new Error('两次输入密码不一致'));
      } else {
        callback();
      }
    };
    
    const profileRules = {
      username: [
        { required: true, message: '请输入用户名', trigger: 'blur' },
        { min: 3, max: 20, message: '用户名长度在3到20个字符之间', trigger: 'blur' }
      ],
      email: [
        { required: true, message: '请输入邮箱', trigger: 'blur' },
        { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
      ]
    };
    
    const passwordRules = {
      currentPassword: [
        { required: true, message: '请输入当前密码', trigger: 'blur' }
      ],
      newPassword: [
        { required: true, trigger: 'blur', validator: validatePassword }
      ],
      confirmPassword: [
        { required: true, trigger: 'blur', validator: validateConfirmPassword }
      ]
    };
    
    onMounted(() => {
      // 初始化表单数据
      profileFormData.username = user.value.username || '';
      profileFormData.email = user.value.email || '';
    });
    
    // 触发头像上传
    const triggerAvatarUpload = () => {
      avatarInput.value.click();
    };
    
    // 处理头像变更
    const handleAvatarChange = async (e) => {
      const file = e.target.files[0];
      if (!file) return;
      
      // 验证文件类型
      const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
      if (!allowedTypes.includes(file.type)) {
        ElMessage.error('只支持 JPG、PNG、GIF 格式的图片');
        return;
      }
      
      // 验证文件大小（最大5MB）
      if (file.size > 5 * 1024 * 1024) {
        ElMessage.error('图片大小不能超过5MB');
        return;
      }
      
      try {
        loading.value = true;
        
        // 创建FormData
        const formData = new FormData();
        formData.append('avatar', file);
        
        // 更新头像
        await store.dispatch('auth/updateAvatar', formData);
        
        ElMessage.success('头像更新成功');
      } catch (error) {
        console.error('Failed to update avatar:', error);
        ElMessage.error('头像更新失败，请重试');
      } finally {
        loading.value = false;
        // 清空文件输入
        e.target.value = '';
      }
    };
    
    // 更新个人资料
    const updateProfile = async () => {
      await profileForm.value.validate(async (valid) => {
        if (valid) {
          try {
            loading.value = true;
            
            await store.dispatch('auth/updateUser', {
              username: profileFormData.username,
              email: profileFormData.email
            });
            
            ElMessage.success('个人资料更新成功');
          } catch (error) {
            console.error('Failed to update profile:', error);
            ElMessage.error('个人资料更新失败，请重试');
          } finally {
            loading.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    // 修改密码
    const changePassword = async () => {
      await passwordForm.value.validate(async (valid) => {
        if (valid) {
          try {
            loading.value = true;
            
            await store.dispatch('auth/changePassword', {
              currentPassword: passwordFormData.currentPassword,
              newPassword: passwordFormData.newPassword
            });
            
            ElMessage.success('密码修改成功，请重新登录');
            
            // 清空表单
            passwordForm.value.resetFields();
            
            // 登出，回到登录页
            store.dispatch('auth/logout');
            router.push('/login');
          } catch (error) {
            console.error('Failed to change password:', error);
            ElMessage.error('密码修改失败，请重试');
          } finally {
            loading.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    // 注销账户
    const deleteAccount = async () => {
      try {
        loading.value = true;
        
        await store.dispatch('auth/deleteAccount');
        
        ElMessage.success('账户已注销');
        
        // 登出，回到首页
        store.dispatch('auth/logout');
        router.push('/');
      } catch (error) {
        console.error('Failed to delete account:', error);
        ElMessage.error('账户注销失败，请重试');
      } finally {
        loading.value = false;
      }
    };
    
    return {
      user,
      userAvatar,
      userRole,
      profileForm,
      profileForm: profileFormData,
      profileRules,
      passwordForm,
      passwordForm: passwordFormData,
      passwordRules,
      avatarInput,
      loading,
      triggerAvatarUpload,
      handleAvatarChange,
      updateProfile,
      changePassword,
      deleteAccount
    };
  }
});
</script>

<style scoped>
.user-profile {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.profile-card {
  max-width: 800px;
  margin: 0 auto;
}

.profile-header {
  display: flex;
  margin-bottom: 20px;
}

.avatar-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-right: 30px;
}

.upload-button {
  margin-top: 10px;
}

.user-info {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.user-info h2 {
  margin: 0 0 10px 0;
}

.user-info p {
  margin: 5px 0;
  color: #606266;
}

.account-actions {
  margin-top: 20px;
}
</style>
```

Let's continue with the User Favorites Page (src/views/user/UserFavorites.vue):

```vue
<template>
  <div class="user-favorites">
    <h1 class="page-title">我的收藏</h1>
    
    <el-empty v-if="favorites.length === 0" description="暂无收藏图书" />
    
    <el-row :gutter="20" v-else>
      <el-col :xs="24" :sm="12" :md="8" :lg="6" v-for="favorite in favorites" :key="favorite.id">
        <el-card class="favorite-card">
          <div class="favorite-cover" @click="navigateToBook(favorite.bookId)">
            <img :src="favorite.bookCoverImage || defaultCover" :alt="favorite.bookTitle" />
          </div>
          
          <div class="favorite-info">
            <h3 class="favorite-title" @click="navigateToBook(favorite.bookId)">
              {{ favorite.bookTitle }}
            </h3>
            
            <div class="favorite-actions">
              <el-button 
                type="danger" 
                size="small" 
                icon="Delete"
                @click="removeFavorite(favorite.bookId)"
              >
                取消收藏
              </el-button>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>
    
    <Pagination
      v-if="favorites.length > 0"
      :total="total"
      :page-size.sync="pageSize"
      :current-page.sync="currentPage"
      @pagination="handlePagination"
    />
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { useRouter } from 'vue-router';
import { ElMessage, ElMessageBox } from 'element-plus';
import Pagination from '@/components/Pagination.vue';
import defaultCover from '@/assets/default-book-cover.jpg';

export default defineComponent({
  name: 'UserFavorites',
  components: {
    Pagination
  },
  setup() {
    const store = useStore();
    const router = useRouter();
    
    const currentPage = ref(1);
    const pageSize = ref(12);
    const defaultCover = ref(defaultCover);
    
    const favorites = computed(() => store.getters['favorite/allFavorites']);
    const pagination = computed(() => store.getters['favorite/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['favorite/loading']);
    
    onMounted(() => {
      fetchFavorites();
    });
    
    const fetchFavorites = () => {
      store.dispatch('favorite/fetchUserFavorites', {
        page: currentPage.value - 1,
        size: pageSize.value
      });
    };
    
    const handlePagination = ({ page, size }) => {
      currentPage.value = page + 1;
      pageSize.value = size;
      fetchFavorites();
    };
    
    const navigateToBook = (bookId) => {
      router.push({
        name: 'BookDetail',
        params: { id: bookId }
      });
    };
    
    const removeFavorite = (bookId) => {
      ElMessageBox.confirm(
        '确定要取消收藏这本书吗？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(async () => {
        try {
          await store.dispatch('favorite/removeFromFavorites', bookId);
          ElMessage.success('已取消收藏');
          fetchFavorites();
        } catch (error) {
          console.error('Failed to remove favorite:', error);
          ElMessage.error('操作失败，请重试');
        }
      }).catch(() => {
        // 用户取消操作
      });
    };
    
    return {
      favorites,
      total,
      loading,
      currentPage,
      pageSize,
      defaultCover,
      handlePagination,
      navigateToBook,
      removeFavorite
    };
  }
});
</script>

<style scoped>
.user-favorites {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.favorite-card {
  margin-bottom: 20px;
  transition: transform 0.3s;
}

.favorite-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.favorite-cover {
  height: 200px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  background-color: #f5f5f5;
  margin-bottom: 10px;
  cursor: pointer;
}

.favorite-cover img {
  width: auto;
  height: 100%;
  object-fit: cover;
}

.favorite-title {
  margin: 0 0 10px 0;
  font-size: 16px;
  cursor: pointer;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.favorite-title:hover {
  color: #409EFF;
}

.favorite-actions {
  display: flex;
  justify-content: center;
}
</style>
```

Now let's create the User Browsing History Page (src/views/user/UserHistory.vue):

```vue
<template>
  <div class="user-history">
    <div class="page-header">
      <h1 class="page-title">浏览历史</h1>
      <el-button 
        type="danger" 
        @click="clearHistory" 
        :disabled="browsingHistory.length === 0"
        :loading="loading"
      >
        清空历史记录
      </el-button>
    </div>
    
    <el-empty v-if="browsingHistory.length === 0" description="暂无浏览历史" />
    
    <el-row :gutter="20" v-else>
      <el-col 
        :xs="24" 
        :sm="12" 
        :md="8" 
        :lg="6" 
        v-for="book in browsingHistory" 
        :key="book.id"
      >
        <BookCard :book="book" />
      </el-col>
    </el-row>
    
    <Pagination
      v-if="browsingHistory.length > 0"
      :total="total"
      :page-size.sync="pageSize"
      :current


用户:
继续

Claude:
I'll continue with the User Browsing History Page (src/views/user/UserHistory.vue):

```vue
<template>
  <div class="user-history">
    <div class="page-header">
      <h1 class="page-title">浏览历史</h1>
      <el-button 
        type="danger" 
        @click="clearHistory" 
        :disabled="browsingHistory.length === 0"
        :loading="loading"
      >
        清空历史记录
      </el-button>
    </div>
    
    <el-empty v-if="browsingHistory.length === 0" description="暂无浏览历史" />
    
    <el-row :gutter="20" v-else>
      <el-col 
        :xs="24" 
        :sm="12" 
        :md="8" 
        :lg="6" 
        v-for="book in browsingHistory" 
        :key="book.id"
      >
        <BookCard :book="book" />
      </el-col>
    </el-row>
    
    <Pagination
      v-if="browsingHistory.length > 0"
      :total="total"
      :page-size.sync="pageSize"
      :current-page.sync="currentPage"
      @pagination="handlePagination"
    />
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage, ElMessageBox } from 'element-plus';
import BookCard from '@/components/BookCard.vue';
import Pagination from '@/components/Pagination.vue';

export default defineComponent({
  name: 'UserHistory',
  components: {
    BookCard,
    Pagination
  },
  setup() {
    const store = useStore();
    
    const currentPage = ref(1);
    const pageSize = ref(12);
    
    const browsingHistory = computed(() => store.getters['history/browsingHistory']);
    const pagination = computed(() => store.getters['history/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['history/loading']);
    
    onMounted(() => {
      fetchHistory();
    });
    
    const fetchHistory = () => {
      store.dispatch('history/fetchBrowsingHistory', {
        page: currentPage.value - 1,
        size: pageSize.value
      });
    };
    
    const handlePagination = ({ page, size }) => {
      currentPage.value = page + 1;
      pageSize.value = size;
      fetchHistory();
    };
    
    const clearHistory = () => {
      ElMessageBox.confirm(
        '确定要清空所有浏览历史记录吗？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(async () => {
        try {
          await store.dispatch('history/clearBrowsingHistory');
          ElMessage.success('浏览历史已清空');
        } catch (error) {
          console.error('Failed to clear browsing history:', error);
          ElMessage.error('操作失败，请重试');
        }
      }).catch(() => {
        // 用户取消操作
      });
    };
    
    return {
      browsingHistory,
      total,
      loading,
      currentPage,
      pageSize,
      handlePagination,
      clearHistory
    };
  }
});
</script>

<style scoped>
.user-history {
  margin-bottom: 40px;
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.page-title {
  margin: 0;
}
</style>
```

Now, let's create the User Feedback Page (src/views/user/UserFeedback.vue):

```vue
<template>
  <div class="user-feedback">
    <h1 class="page-title">用户反馈</h1>
    
    <el-card class="feedback-form-card">
      <template #header>
        <div class="card-header">
          <h3>提交反馈</h3>
        </div>
      </template>
      
      <el-form ref="feedbackForm" :model="feedbackForm" :rules="rules">
        <el-form-item prop="content">
          <el-input
            v-model="feedbackForm.content"
            type="textarea"
            :rows="5"
            placeholder="请输入您的反馈内容..."
            maxlength="2000"
            show-word-limit
          />
        </el-form-item>
        
        <el-form-item>
          <el-button type="primary" @click="submitFeedback" :loading="submitting">
            提交反馈
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>
    
    <div class="feedback-history" v-if="feedbacks.length > 0">
      <h2>我的反馈历史</h2>
      
      <el-timeline>
        <el-timeline-item
          v-for="feedback in feedbacks"
          :key="feedback.id"
          :timestamp="formatDate(feedback.createdAt)"
          :type="feedbackTypeMap[feedback.status].type"
          :color="feedbackTypeMap[feedback.status].color"
        >
          <el-card class="feedback-card">
            <div class="feedback-header">
              <span class="feedback-status">
                {{ feedbackTypeMap[feedback.status].label }}
              </span>
              
              <el-popconfirm
                title="确定要删除这条反馈吗？"
                @confirm="deleteFeedback(feedback.id)"
              >
                <template #reference>
                  <el-button type="danger" size="small" icon="Delete" circle />
                </template>
              </el-popconfirm>
            </div>
            
            <div class="feedback-content">
              {{ feedback.content }}
            </div>
          </el-card>
        </el-timeline-item>
      </el-timeline>
      
      <Pagination
        v-if="total > pageSize"
        :total="total"
        :page-size.sync="pageSize"
        :current-page.sync="currentPage"
        @pagination="handlePagination"
      />
    </div>
    
    <el-empty 
      v-else-if="!loading" 
      description="暂无反馈历史"
    />
  </div>
</template>

<script>
import { defineComponent, ref, reactive, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage } from 'element-plus';
import Pagination from '@/components/Pagination.vue';

export default defineComponent({
  name: 'UserFeedback',
  components: {
    Pagination
  },
  setup() {
    const store = useStore();
    
    const feedbackForm = ref(null);
    const currentPage = ref(1);
    const pageSize = ref(10);
    const submitting = ref(false);
    
    const formData = reactive({
      content: ''
    });
    
    const rules = {
      content: [
        { required: true, message: '请输入反馈内容', trigger: 'blur' },
        { min: 5, max: 2000, message: '反馈内容长度在5到2000个字符之间', trigger: 'blur' }
      ]
    };
    
    const feedbackTypeMap = {
      PENDING: { label: '待处理', type: 'warning', color: '#E6A23C' },
      PROCESSED: { label: '已处理', type: 'success', color: '#67C23A' },
      REJECTED: { label: '已拒绝', type: 'danger', color: '#F56C6C' }
    };
    
    const feedbacks = computed(() => store.getters['feedback/userFeedbacks']);
    const pagination = computed(() => store.getters['feedback/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['feedback/loading']);
    
    onMounted(() => {
      fetchFeedbacks();
    });
    
    const fetchFeedbacks = () => {
      store.dispatch('feedback/fetchUserFeedback', {
        page: currentPage.value - 1,
        size: pageSize.value
      });
    };
    
    const handlePagination = ({ page, size }) => {
      currentPage.value = page + 1;
      pageSize.value = size;
      fetchFeedbacks();
    };
    
    const submitFeedback = async () => {
      await feedbackForm.value.validate(async (valid) => {
        if (valid) {
          try {
            submitting.value = true;
            
            await store.dispatch('feedback/createFeedback', formData.content);
            
            ElMessage.success('反馈提交成功');
            
            // 清空表单
            formData.content = '';
            feedbackForm.value.resetFields();
            
            // 刷新反馈列表
            fetchFeedbacks();
          } catch (error) {
            console.error('Failed to submit feedback:', error);
            ElMessage.error('反馈提交失败，请重试');
          } finally {
            submitting.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    const deleteFeedback = async (id) => {
      try {
        await store.dispatch('feedback/deleteFeedback', id);
        ElMessage.success('反馈删除成功');
        fetchFeedbacks();
      } catch (error) {
        console.error('Failed to delete feedback:', error);
        ElMessage.error('反馈删除失败，请重试');
      }
    };
    
    const formatDate = (dateString) => {
      if (!dateString) return '';
      
      const date = new Date(dateString);
      return date.toLocaleString();
    };
    
    return {
      feedbackForm,
      feedbackForm: formData,
      rules,
      feedbacks,
      total,
      loading,
      currentPage,
      pageSize,
      submitting,
      feedbackTypeMap,
      handlePagination,
      submitFeedback,
      deleteFeedback,
      formatDate
    };
  }
});
</script>

<style scoped>
.user-feedback {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.feedback-form-card {
  margin-bottom: 30px;
}

.feedback-history {
  margin-top: 40px;
}

.feedback-history h2 {
  margin-bottom: 20px;
}

.feedback-card {
  margin-bottom: 10px;
}

.feedback-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.feedback-status {
  font-weight: bold;
}

.feedback-content {
  white-space: pre-wrap;
  word-break: break-word;
}
</style>
```

Now, let's create the Admin Dashboard Page (src/views/admin/AdminDashboard.vue):

```vue
<template>
  <div class="admin-dashboard">
    <h1 class="page-title">管理后台</h1>
    
    <el-row :gutter="20">
      <el-col :xs="24" :sm="12" :md="6">
        <el-card class="stat-card">
          <div class="stat-card-header">
            <el-icon class="stat-icon"><Collection /></el-icon>
            <div class="stat-info">
              <div class="stat-title">图书总数</div>
              <div class="stat-value">{{ stats.bookCount }}</div>
            </div>
          </div>
          <div class="stat-action">
            <router-link to="/admin/books">查看图书</router-link>
          </div>
        </el-card>
      </el-col>
      
      <el-col :xs="24" :sm="12" :md="6">
        <el-card class="stat-card">
          <div class="stat-card-header">
            <el-icon class="stat-icon"><Files /></el-icon>
            <div class="stat-info">
              <div class="stat-title">分类总数</div>
              <div class="stat-value">{{ stats.categoryCount }}</div>
            </div>
          </div>
          <div class="stat-action">
            <router-link to="/admin/categories">查看分类</router-link>
          </div>
        </el-card>
      </el-col>
      
      <el-col :xs="24" :sm="12" :md="6">
        <el-card class="stat-card">
          <div class="stat-card-header">
            <el-icon class="stat-icon"><User /></el-icon>
            <div class="stat-info">
              <div class="stat-title">用户总数</div>
              <div class="stat-value">{{ stats.userCount }}</div>
            </div>
          </div>
          <div class="stat-action">
            <router-link to="/admin/users">查看用户</router-link>
          </div>
        </el-card>
      </el-col>
      
      <el-col :xs="24" :sm="12" :md="6">
        <el-card class="stat-card">
          <div class="stat-card-header">
            <el-icon class="stat-icon"><ChatDotRound /></el-icon>
            <div class="stat-info">
              <div class="stat-title">待处理反馈</div>
              <div class="stat-value">{{ stats.pendingFeedbackCount }}</div>
            </div>
          </div>
          <div class="stat-action">
            <router-link to="/admin/feedback">查看反馈</router-link>
          </div>
        </el-card>
      </el-col>
    </el-row>
    
    <el-row :gutter="20" class="dashboard-section">
      <el-col :xs="24" :md="12">
        <el-card class="chart-card">
          <template #header>
            <div class="card-header">
              <span>最近添加的图书</span>
              <router-link to="/admin/books">查看全部</router-link>
            </div>
          </template>
          
          <el-table :data="recentBooks" stripe style="width: 100%">
            <el-table-column prop="title" label="书名" />
            <el-table-column prop="author" label="作者" />
            <el-table-column prop="categoryName" label="分类" width="100" />
            <el-table-column label="操作" width="150">
              <template #default="scope">
                <router-link :to="`/books/${scope.row.id}`">
                  <el-button type="primary" size="small" icon="View">查看</el-button>
                </router-link>
                <router-link :to="`/admin/books/${scope.row.id}/edit`">
                  <el-button type="warning" size="small" icon="Edit">编辑</el-button>
                </router-link>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>
      
      <el-col :xs="24" :md="12">
        <el-card class="chart-card">
          <template #header>
            <div class="card-header">
              <span>最近反馈</span>
              <router-link to="/admin/feedback">查看全部</router-link>
            </div>
          </template>
          
          <el-table :data="recentFeedbacks" stripe style="width: 100%">
            <el-table-column prop="username" label="用户" width="100" />
            <el-table-column prop="content" label="内容">
              <template #default="scope">
                <div class="feedback-content-cell">{{ scope.row.content }}</div>
              </template>
            </el-table-column>
            <el-table-column prop="status" label="状态" width="100">
              <template #default="scope">
                <el-tag
                  :type="getFeedbackTagType(scope.row.status)"
                  size="small"
                >
                  {{ getFeedbackStatusText(scope.row.status) }}
                </el-tag>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>
    </el-row>
    
    <el-row :gutter="20" class="dashboard-section">
      <el-col :span="24">
        <el-card class="chart-card">
          <template #header>
            <div class="card-header">
              <span>快捷操作</span>
            </div>
          </template>
          
          <div class="quick-actions">
            <router-link to="/admin/books/create">
              <el-button type="primary" icon="Plus">添加图书</el-button>
            </router-link>
            
            <el-button 
              type="primary" 
              icon="Plus" 
              @click="showCategoryDialog = true"
            >
              添加分类
            </el-button>
          </div>
        </el-card>
      </el-col>
    </el-row>
    
    <!-- 添加分类对话框 -->
    <el-dialog
      v-model="showCategoryDialog"
      title="添加分类"
      width="500px"
    >
      <el-form
        ref="categoryForm"
        :model="categoryForm"
        :rules="categoryRules"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="name">
          <el-input v-model="categoryForm.name" />
        </el-form-item>
        
        <el-form-item label="分类描述" prop="description">
          <el-input
            v-model="categoryForm.description"
            type="textarea"
            :rows="3"
          />
        </el-form-item>
      </el-form>
      
      <template #footer>
        <el-button @click="showCategoryDialog = false">取消</el-button>
        <el-button type="primary" @click="createCategory" :loading="submitting">
          创建
        </el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { defineComponent, ref, reactive, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage } from 'element-plus';
import BookService from '@/services/book.service';
import FeedbackService from '@/services/feedback.service';
import CategoryService from '@/services/category.service';
import UserService from '@/services/user.service';

export default defineComponent({
  name: 'AdminDashboard',
  setup() {
    const store = useStore();
    
    const categoryForm = ref(null);
    const showCategoryDialog = ref(false);
    const submitting = ref(false);
    
    const stats = reactive({
      bookCount: 0,
      categoryCount: 0,
      userCount: 0,
      pendingFeedbackCount: 0
    });
    
    const recentBooks = ref([]);
    const recentFeedbacks = ref([]);
    
    const categoryFormData = reactive({
      name: '',
      description: ''
    });
    
    const categoryRules = {
      name: [
        { required: true, message: '请输入分类名称', trigger: 'blur' },
        { min: 2, max: 50, message: '分类名称长度在2到50个字符之间', trigger: 'blur' }
      ],
      description: [
        { max: 500, message: '分类描述不能超过500个字符', trigger: 'blur' }
      ]
    };
    
    // 获取统计数据
    const fetchStats = async () => {
      try {
        // 获取图书总数
        const booksResponse = await BookService.getAllBooks(0, 1);
        stats.bookCount = booksResponse.data.data.totalElements;
        
        // 获取分类总数
        const categoriesResponse = await CategoryService.getAllCategories();
        stats.categoryCount = categoriesResponse.data.data.length;
        
        // 获取用户总数
        const usersResponse = await UserService.getAllUsers();
        stats.userCount = usersResponse.data.data.length;
        
        // 获取待处理反馈数量
        const feedbackResponse = await FeedbackService.getFeedbackByStatus('PENDING', 0, 1);
        stats.pendingFeedbackCount = feedbackResponse.data.data.totalElements;
      } catch (error) {
        console.error('Failed to fetch stats:', error);
      }
    };
    
    // 获取最近添加的图书
    const fetchRecentBooks = async () => {
      try {
        const response = await BookService.getAllBooks(0, 5, 'id', 'desc');
        recentBooks.value = response.data.data.content;
      } catch (error) {
        console.error('Failed to fetch recent books:', error);
      }
    };
    
    // 获取最近的反馈
    const fetchRecentFeedbacks = async () => {
      try {
        const response = await FeedbackService.getAllFeedback(0, 5);
        recentFeedbacks.value = response.data.data.content;
      } catch (error) {
        console.error('Failed to fetch recent feedbacks:', error);
      }
    };
    
    // 创建分类
    const createCategory = async () => {
      await categoryForm.value.validate(async (valid) => {
        if (valid) {
          try {
            submitting.value = true;
            
            await store.dispatch('category/createCategory', {
              name: categoryFormData.name,
              description: categoryFormData.description
            });
            
            ElMessage.success('分类创建成功');
            
            // 清空表单并关闭对话框
            categoryFormData.name = '';
            categoryFormData.description = '';
            showCategoryDialog.value = false;
            
            // 刷新统计数据
            fetchStats();
          } catch (error) {
            console.error('Failed to create category:', error);
            ElMessage.error('分类创建失败，请重试');
          } finally {
            submitting.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    // 获取反馈状态对应的Tag类型
    const getFeedbackTagType = (status) => {
      switch (status) {
        case 'PENDING':
          return 'warning';
        case 'PROCESSED':
          return 'success';
        case 'REJECTED':
          return 'danger';
        default:
          return 'info';
      }
    };
    
    // 获取反馈状态的文本描述
    const getFeedbackStatusText = (status) => {
      switch (status) {
        case 'PENDING':
          return '待处理';
        case 'PROCESSED':
          return '已处理';
        case 'REJECTED':
          return '已拒绝';
        default:
          return '未知';
      }
    };
    
    onMounted(() => {
      fetchStats();
      fetchRecentBooks();
      fetchRecentFeedbacks();
    });
    
    return {
      stats,
      recentBooks,
      recentFeedbacks,
      categoryForm,
      categoryForm: categoryFormData,
      categoryRules,
      showCategoryDialog,
      submitting,
      createCategory,
      getFeedbackTagType,
      getFeedbackStatusText
    };
  }
});
</script>

<style scoped>
.admin-dashboard {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.stat-card {
  height: 100%;
}

.stat-card-header {
  display: flex;
  align-items: center;
  margin-bottom: 15px;
}

.stat-icon {
  font-size: 40px;
  margin-right: 15px;
  color: #409EFF;
}

.stat-info {
  flex: 1;
}

.stat-title {
  font-size: 14px;
  color: #606266;
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
  margin-top: 5px;
}

.stat-action {
  text-align: right;
  margin-top: 10px;
}

.dashboard-section {
  margin-top: 20px;
}

.chart-card {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.quick-actions {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.feedback-content-cell {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 200px;
}
</style>
```

Let's create the Admin Books Management Page (src/views/admin/AdminBooks.vue):

```vue
<template>
  <div class="admin-books">
    <div class="page-header">
      <h1 class="page-title">图书管理</h1>
      <router-link to="/admin/books/create">
        <el-button type="primary" icon="Plus">添加图书</el-button>
      </router-link>
    </div>
    
    <div class="search-container">
      <el-input
        v-model="searchKeyword"
        placeholder="搜索图书（标题/作者/ISBN）"
        clearable
        @keyup.enter="handleSearch"
      >
        <template #append>
          <el-button @click="handleSearch" icon="Search" />
        </template>
      </el-input>
      
      <el-select 
        v-model="categoryFilter" 
        placeholder="分类筛选" 
        clearable
        @change="handleSearch"
      >
        <el-option
          v-for="category in categories"
          :key="category.id"
          :label="category.name"
          :value="category.id"
        />
      </el-select>
    </div>
    
    <el-table
      :data="books"
      style="width: 100%"
      v-loading="loading"
      border
    >
      <el-table-column label="ID" prop="id" width="80" sortable />
      
      <el-table-column label="封面" width="100">
        <template #default="scope">
          <el-image
            :src="scope.row.coverImage || defaultCover"
            :preview-src-list="[scope.row.coverImage || defaultCover]"
            fit="cover"
            style="width: 60px; height: 80px"
          />
        </template>
      </el-table-column>
      
      <el-table-column label="书名" prop="title" sortable />
      
      <el-table-column label="作者" prop="author" sortable />
      
      <el-table-column label="ISBN" prop="isbn" width="120" />
      
      <el-table-column label="出版社" prop="publisher" width="150" />
      
      <el-table-column label="分类" prop="categoryName" width="100" />
      
      <el-table-column label="操作" width="200" fixed="right">
        <template #default="scope">
          <router-link :to="`/books/${scope.row.id}`">
            <el-button type="primary" size="small" icon="View">查看</el-button>
          </router-link>
          
          <router-link :to="`/admin/books/${scope.row.id}/edit`">
            <el-button type="warning" size="small" icon="Edit">编辑</el-button>
          </router-link>
          
          <el-button
            type="danger"
            size="small"
            icon="Delete"
            @click="handleDelete(scope.row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <el-pagination
      v-if="total > 0"
      background
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      :page-size="pageSize"
      :current-page="currentPage"
      :page-sizes="[10, 20, 50, 100]"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage, ElMessageBox } from 'element-plus';
import defaultCover from '@/assets/default-book-cover.jpg';

export default defineComponent({
  name: 'AdminBooks',
  setup() {
    const store = useStore();
    
    const currentPage = ref(1);
    const pageSize = ref(10);
    const searchKeyword = ref('');
    const categoryFilter = ref('');
    const defaultCover = ref(defaultCover);
    
    const books = computed(() => store.getters['book/allBooks']);
    const pagination = computed(() => store.getters['book/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['book/loading']);
    const categories = computed(() => store.getters['category/allCategories']);
    
    onMounted(() => {
      // 加载分类列表
      store.dispatch('category/fetchAllCategories');
      
      // 加载图书列表
      fetchBooks();
    });
    
    const fetchBooks = () => {
      const payload = {
        page: currentPage.value - 1,
        size: pageSize.value,
        sortBy: 'id',
        sortDir: 'desc'
      };
      
      if (searchKeyword.value || categoryFilter.value) {
        store.dispatch('book/searchBooks', {
          ...payload,
          keyword: searchKeyword.value,
          categoryId: categoryFilter.value || null
        });
      } else {
        store.dispatch('book/fetchAllBooks', payload);
      }
    };
    
    const handleSizeChange = (val) => {
      pageSize.value = val;
      fetchBooks();
    };
    
    const handleCurrentChange = (val) => {
      currentPage.value =


用户:
继续

Claude:
I'll continue with the Admin Books Management Page (src/views/admin/AdminBooks.vue):

```vue
<template>
  <div class="admin-books">
    <div class="page-header">
      <h1 class="page-title">图书管理</h1>
      <router-link to="/admin/books/create">
        <el-button type="primary" icon="Plus">添加图书</el-button>
      </router-link>
    </div>
    
    <div class="search-container">
      <el-input
        v-model="searchKeyword"
        placeholder="搜索图书（标题/作者/ISBN）"
        clearable
        @keyup.enter="handleSearch"
      >
        <template #append>
          <el-button @click="handleSearch" icon="Search" />
        </template>
      </el-input>
      
      <el-select 
        v-model="categoryFilter" 
        placeholder="分类筛选" 
        clearable
        @change="handleSearch"
      >
        <el-option
          v-for="category in categories"
          :key="category.id"
          :label="category.name"
          :value="category.id"
        />
      </el-select>
    </div>
    
    <el-table
      :data="books"
      style="width: 100%"
      v-loading="loading"
      border
    >
      <el-table-column label="ID" prop="id" width="80" sortable />
      
      <el-table-column label="封面" width="100">
        <template #default="scope">
          <el-image
            :src="scope.row.coverImage || defaultCover"
            :preview-src-list="[scope.row.coverImage || defaultCover]"
            fit="cover"
            style="width: 60px; height: 80px"
          />
        </template>
      </el-table-column>
      
      <el-table-column label="书名" prop="title" sortable />
      
      <el-table-column label="作者" prop="author" sortable />
      
      <el-table-column label="ISBN" prop="isbn" width="120" />
      
      <el-table-column label="出版社" prop="publisher" width="150" />
      
      <el-table-column label="分类" prop="categoryName" width="100" />
      
      <el-table-column label="操作" width="200" fixed="right">
        <template #default="scope">
          <router-link :to="`/books/${scope.row.id}`">
            <el-button type="primary" size="small" icon="View">查看</el-button>
          </router-link>
          
          <router-link :to="`/admin/books/${scope.row.id}/edit`">
            <el-button type="warning" size="small" icon="Edit">编辑</el-button>
          </router-link>
          
          <el-button
            type="danger"
            size="small"
            icon="Delete"
            @click="handleDelete(scope.row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <el-pagination
      v-if="total > 0"
      background
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      :page-size="pageSize"
      :current-page="currentPage"
      :page-sizes="[10, 20, 50, 100]"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage, ElMessageBox } from 'element-plus';
import defaultCover from '@/assets/default-book-cover.jpg';

export default defineComponent({
  name: 'AdminBooks',
  setup() {
    const store = useStore();
    
    const currentPage = ref(1);
    const pageSize = ref(10);
    const searchKeyword = ref('');
    const categoryFilter = ref('');
    const defaultCover = ref(defaultCover);
    
    const books = computed(() => store.getters['book/allBooks']);
    const pagination = computed(() => store.getters['book/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['book/loading']);
    const categories = computed(() => store.getters['category/allCategories']);
    
    onMounted(() => {
      // 加载分类列表
      store.dispatch('category/fetchAllCategories');
      
      // 加载图书列表
      fetchBooks();
    });
    
    const fetchBooks = () => {
      const payload = {
        page: currentPage.value - 1,
        size: pageSize.value,
        sortBy: 'id',
        sortDir: 'desc'
      };
      
      if (searchKeyword.value || categoryFilter.value) {
        store.dispatch('book/searchBooks', {
          ...payload,
          keyword: searchKeyword.value,
          categoryId: categoryFilter.value || null
        });
      } else {
        store.dispatch('book/fetchAllBooks', payload);
      }
    };
    
    const handleSizeChange = (val) => {
      pageSize.value = val;
      fetchBooks();
    };
    
    const handleCurrentChange = (val) => {
      currentPage.value = val;
      fetchBooks();
    };
    
    const handleSearch = () => {
      currentPage.value = 1;
      fetchBooks();
    };
    
    const handleDelete = (book) => {
      ElMessageBox.confirm(
        `确定要删除图书《${book.title}》吗？此操作不可恢复！`,
        '警告',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(async () => {
        try {
          await store.dispatch('book/deleteBook', book.id);
          ElMessage.success('图书删除成功');
          fetchBooks();
        } catch (error) {
          console.error('Failed to delete book:', error);
          ElMessage.error('图书删除失败，请重试');
        }
      }).catch(() => {
        // 用户取消操作
      });
    };
    
    return {
      books,
      total,
      loading,
      currentPage,
      pageSize,
      searchKeyword,
      categoryFilter,
      categories,
      defaultCover,
      handleSizeChange,
      handleCurrentChange,
      handleSearch,
      handleDelete
    };
  }
});
</script>

<style scoped>
.admin-books {
  margin-bottom: 40px;
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.page-title {
  margin: 0;
}

.search-container {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.el-pagination {
  margin-top: 20px;
  text-align: center;
}
</style>
```

Now, let's create the Admin Book Edit Page (src/views/admin/AdminBookEdit.vue):

```vue
<template>
  <div class="admin-book-edit">
    <h1 class="page-title">{{ isEdit ? '编辑图书' : '添加图书' }}</h1>
    
    <el-card class="book-form-card">
      <el-form
        ref="bookForm"
        :model="bookForm"
        :rules="rules"
        label-width="100px"
        v-loading="loading"
      >
        <el-row :gutter="20">
          <el-col :md="16">
            <!-- 基本信息 -->
            <el-form-item label="书名" prop="title">
              <el-input v-model="bookForm.title" />
            </el-form-item>
            
            <el-form-item label="作者" prop="author">
              <el-input v-model="bookForm.author" />
            </el-form-item>
            
            <el-form-item label="ISBN" prop="isbn">
              <el-input v-model="bookForm.isbn" />
            </el-form-item>
            
            <el-form-item label="出版社" prop="publisher">
              <el-input v-model="bookForm.publisher" />
            </el-form-item>
            
            <el-form-item label="出版日期" prop="publishDate">
              <el-date-picker
                v-model="bookForm.publishDate"
                type="date"
                placeholder="选择日期"
                style="width: 100%"
              />
            </el-form-item>
            
            <el-form-item label="分类" prop="categoryId">
              <el-select
                v-model="bookForm.categoryId"
                placeholder="选择分类"
                style="width: 100%"
              >
                <el-option
                  v-for="category in categories"
                  :key="category.id"
                  :label="category.name"
                  :value="category.id"
                />
              </el-select>
            </el-form-item>
            
            <el-form-item label="图书简介" prop="description">
              <el-input
                v-model="bookForm.description"
                type="textarea"
                :rows="5"
                placeholder="请输入图书简介"
                maxlength="2000"
                show-word-limit
              />
            </el-form-item>
          </el-col>
          
          <el-col :md="8">
            <!-- 封面上传 -->
            <el-form-item label="封面">
              <div class="cover-upload">
                <el-image
                  v-if="coverPreview"
                  :src="coverPreview"
                  fit="contain"
                  class="cover-preview"
                />
                <div v-else class="cover-placeholder">
                  <el-icon><picture /></el-icon>
                  <span>暂无封面</span>
                </div>
                
                <div class="cover-actions">
                  <el-upload
                    class="upload"
                    action="#"
                    :auto-upload="false"
                    :show-file-list="false"
                    :on-change="handleCoverChange"
                    accept="image/jpeg,image/png,image/gif"
                  >
                    <el-button type="primary">
                      {{ coverPreview ? '更换封面' : '上传封面' }}
                    </el-button>
                  </el-upload>
                  
                  <el-button 
                    v-if="coverPreview" 
                    type="danger"
                    @click="removeCover"
                  >
                    移除封面
                  </el-button>
                </div>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
        
        <div class="form-actions">
          <el-button @click="goBack">取消</el-button>
          <el-button type="primary" @click="handleSubmit" :loading="submitting">
            {{ isEdit ? '保存修改' : '添加图书' }}
          </el-button>
        </div>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import { defineComponent, ref, reactive, computed, onMounted, watch } from 'vue';
import { useStore } from 'vuex';
import { useRouter, useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';

export default defineComponent({
  name: 'AdminBookEdit',
  props: {
    id: {
      type: [String, Number],
      default: null
    }
  },
  setup(props) {
    const store = useStore();
    const router = useRouter();
    const route = useRoute();
    
    const bookForm = ref(null);
    const loading = ref(false);
    const submitting = ref(false);
    const coverFile = ref(null);
    const coverPreview = ref('');
    
    const isEdit = computed(() => !!props.id);
    
    const formData = reactive({
      title: '',
      author: '',
      isbn: '',
      publisher: '',
      publishDate: '',
      categoryId: '',
      description: '',
      coverImage: ''
    });
    
    const rules = {
      title: [
        { required: true, message: '请输入书名', trigger: 'blur' },
        { min: 1, max: 100, message: '书名长度在1到100个字符之间', trigger: 'blur' }
      ],
      author: [
        { required: true, message: '请输入作者', trigger: 'blur' },
        { min: 1, max: 100, message: '作者长度在1到100个字符之间', trigger: 'blur' }
      ],
      isbn: [
        { required: true, message: '请输入ISBN', trigger: 'blur' },
        { min: 1, max: 20, message: 'ISBN长度在1到20个字符之间', trigger: 'blur' }
      ],
      publisher: [
        { required: true, message: '请输入出版社', trigger: 'blur' },
        { min: 1, max: 100, message: '出版社长度在1到100个字符之间', trigger: 'blur' }
      ],
      publishDate: [
        { required: true, message: '请选择出版日期', trigger: 'change' }
      ],
      categoryId: [
        { required: true, message: '请选择分类', trigger: 'change' }
      ]
    };
    
    const categories = computed(() => store.getters['category/allCategories']);
    
    // 监听ID变化
    watch(() => props.id, () => {
      if (props.id) {
        fetchBookDetails();
      } else {
        resetForm();
      }
    }, { immediate: true });
    
    onMounted(() => {
      // 加载分类列表
      store.dispatch('category/fetchAllCategories');
    });
    
    // 获取图书详情
    const fetchBookDetails = async () => {
      if (!props.id) return;
      
      try {
        loading.value = true;
        
        const response = await store.dispatch('book/fetchBookById', props.id);
        
        // 填充表单数据
        formData.title = response.title;
        formData.author = response.author;
        formData.isbn = response.isbn;
        formData.publisher = response.publisher;
        formData.publishDate = response.publishDate ? new Date(response.publishDate) : '';
        formData.categoryId = response.categoryId;
        formData.description = response.description || '';
        formData.coverImage = response.coverImage || '';
        
        // 设置封面预览
        if (response.coverImage) {
          coverPreview.value = response.coverImage;
        }
      } catch (error) {
        console.error('Failed to fetch book details:', error);
        ElMessage.error('获取图书信息失败');
      } finally {
        loading.value = false;
      }
    };
    
    // 重置表单
    const resetForm = () => {
      formData.title = '';
      formData.author = '';
      formData.isbn = '';
      formData.publisher = '';
      formData.publishDate = '';
      formData.categoryId = '';
      formData.description = '';
      formData.coverImage = '';
      
      coverPreview.value = '';
      coverFile.value = null;
    };
    
    // 处理封面文件选择
    const handleCoverChange = (file) => {
      const isImage = file.raw.type === 'image/jpeg' || 
                     file.raw.type === 'image/png' || 
                     file.raw.type === 'image/gif';
                     
      if (!isImage) {
        ElMessage.error('封面只能是JPG、PNG或GIF格式的图片');
        return;
      }
      
      const isLt5M = file.size / 1024 / 1024 < 5;
      
      if (!isLt5M) {
        ElMessage.error('封面大小不能超过5MB');
        return;
      }
      
      coverFile.value = file.raw;
      
      // 生成预览
      coverPreview.value = URL.createObjectURL(file.raw);
    };
    
    // 移除封面
    const removeCover = () => {
      coverPreview.value = '';
      coverFile.value = null;
      formData.coverImage = '';
    };
    
    // 提交表单
    const handleSubmit = async () => {
      await bookForm.value.validate(async (valid) => {
        if (valid) {
          try {
            submitting.value = true;
            
            // 准备提交数据
            const bookData = { ...formData };
            
            // 格式化日期
            if (bookData.publishDate) {
              const date = new Date(bookData.publishDate);
              bookData.publishDate = date.toISOString().split('T')[0];
            }
            
            let result;
            
            if (isEdit.value) {
              // 更新图书
              result = await store.dispatch('book/updateBook', {
                id: props.id,
                bookData
              });
              
              // 如果有新封面文件，上传封面
              if (coverFile.value) {
                const formData = new FormData();
                formData.append('cover', coverFile.value);
                
                await store.dispatch('book/updateBookCover', {
                  id: props.id,
                  formData
                });
              }
              
              ElMessage.success('图书更新成功');
            } else {
              // 创建图书
              result = await store.dispatch('book/createBook', bookData);
              
              // 如果有封面文件，上传封面
              if (coverFile.value && result.id) {
                const formData = new FormData();
                formData.append('cover', coverFile.value);
                
                await store.dispatch('book/updateBookCover', {
                  id: result.id,
                  formData
                });
              }
              
              ElMessage.success('图书添加成功');
            }
            
            // 跳转回图书列表
            router.push('/admin/books');
          } catch (error) {
            console.error('Failed to save book:', error);
            
            let errorMessage = isEdit.value ? '图书更新失败' : '图书添加失败';
            
            if (error.response && error.response.data && error.response.data.message) {
              errorMessage += `：${error.response.data.message}`;
            } else {
              errorMessage += '，请重试';
            }
            
            ElMessage.error(errorMessage);
          } finally {
            submitting.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    // 返回上一页
    const goBack = () => {
      router.push('/admin/books');
    };
    
    return {
      bookForm,
      bookForm: formData,
      rules,
      loading,
      submitting,
      categories,
      isEdit,
      coverPreview,
      handleCoverChange,
      removeCover,
      handleSubmit,
      goBack
    };
  }
});
</script>

<style scoped>
.admin-book-edit {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.cover-upload {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.cover-preview {
  width: 200px;
  height: 280px;
  margin-bottom: 15px;
  border: 1px solid #dcdfe6;
}

.cover-placeholder {
  width: 200px;
  height: 280px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: #f5f7fa;
  border: 1px dashed #dcdfe6;
  margin-bottom: 15px;
}

.cover-placeholder .el-icon {
  font-size: 40px;
  color: #909399;
  margin-bottom: 10px;
}

.cover-actions {
  display: flex;
  gap: 10px;
}

.form-actions {
  margin-top: 30px;
  text-align: right;
}
</style>
```

Let's create the Admin Categories Management Page (src/views/admin/AdminCategories.vue):

```vue
<template>
  <div class="admin-categories">
    <div class="page-header">
      <h1 class="page-title">分类管理</h1>
      <el-button type="primary" @click="showAddDialog" icon="Plus">
        添加分类
      </el-button>
    </div>
    
    <el-table
      :data="categories"
      style="width: 100%"
      v-loading="loading"
      border
    >
      <el-table-column label="ID" prop="id" width="80" />
      
      <el-table-column label="分类名称" prop="name" />
      
      <el-table-column label="描述" prop="description" show-overflow-tooltip />
      
      <el-table-column label="图书数量" prop="bookCount" width="100" sortable />
      
      <el-table-column label="操作" width="200">
        <template #default="scope">
          <el-button
            type="primary"
            size="small"
            icon="Edit"
            @click="showEditDialog(scope.row)"
          >
            编辑
          </el-button>
          
          <el-button
            type="danger"
            size="small"
            icon="Delete"
            :disabled="scope.row.bookCount > 0"
            @click="handleDelete(scope.row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <!-- 添加/编辑分类对话框 -->
    <el-dialog
      v-model="dialogVisible"
      :title="isEdit ? '编辑分类' : '添加分类'"
      width="500px"
    >
      <el-form
        ref="categoryForm"
        :model="categoryForm"
        :rules="rules"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="name">
          <el-input v-model="categoryForm.name" />
        </el-form-item>
        
        <el-form-item label="分类描述" prop="description">
          <el-input
            v-model="categoryForm.description"
            type="textarea"
            :rows="3"
          />
        </el-form-item>
      </el-form>
      
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleSubmit" :loading="submitting">
          {{ isEdit ? '保存修改' : '添加分类' }}
        </el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { defineComponent, ref, reactive, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage, ElMessageBox } from 'element-plus';

export default defineComponent({
  name: 'AdminCategories',
  setup() {
    const store = useStore();
    
    const categoryForm = ref(null);
    const dialogVisible = ref(false);
    const isEdit = ref(false);
    const editingId = ref(null);
    const submitting = ref(false);
    
    const formData = reactive({
      name: '',
      description: ''
    });
    
    const rules = {
      name: [
        { required: true, message: '请输入分类名称', trigger: 'blur' },
        { min: 2, max: 50, message: '分类名称长度在2到50个字符之间', trigger: 'blur' }
      ],
      description: [
        { max: 500, message: '分类描述不能超过500个字符', trigger: 'blur' }
      ]
    };
    
    const categories = computed(() => store.getters['category/allCategories']);
    const loading = computed(() => store.getters['category/loading']);
    
    onMounted(() => {
      fetchCategories();
    });
    
    const fetchCategories = () => {
      store.dispatch('category/fetchAllCategories');
    };
    
    const resetForm = () => {
      formData.name = '';
      formData.description = '';
      editingId.value = null;
    };
    
    const showAddDialog = () => {
      resetForm();
      isEdit.value = false;
      dialogVisible.value = true;
    };
    
    const showEditDialog = (category) => {
      resetForm();
      isEdit.value = true;
      editingId.value = category.id;
      formData.name = category.name;
      formData.description = category.description || '';
      dialogVisible.value = true;
    };
    
    const handleSubmit = async () => {
      await categoryForm.value.validate(async (valid) => {
        if (valid) {
          try {
            submitting.value = true;
            
            const categoryData = {
              name: formData.name,
              description: formData.description
            };
            
            if (isEdit.value) {
              // 更新分类
              await store.dispatch('category/updateCategory', {
                id: editingId.value,
                categoryData
              });
              
              ElMessage.success('分类更新成功');
            } else {
              // 创建分类
              await store.dispatch('category/createCategory', categoryData);
              
              ElMessage.success('分类创建成功');
            }
            
            // 关闭对话框并刷新列表
            dialogVisible.value = false;
            fetchCategories();
          } catch (error) {
            console.error('Failed to save category:', error);
            
            let errorMessage = isEdit.value ? '分类更新失败' : '分类创建失败';
            
            if (error.response && error.response.data && error.response.data.message) {
              errorMessage += `：${error.response.data.message}`;
            } else {
              errorMessage += '，请重试';
            }
            
            ElMessage.error(errorMessage);
          } finally {
            submitting.value = false;
          }
        } else {
          return false;
        }
      });
    };
    
    const handleDelete = (category) => {
      if (category.bookCount > 0) {
        ElMessage.warning('该分类下有图书，无法删除');
        return;
      }
      
      ElMessageBox.confirm(
        `确定要删除分类"${category.name}"吗？`,
        '警告',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(async () => {
        try {
          await store.dispatch('category/deleteCategory', category.id);
          ElMessage.success('分类删除成功');
          fetchCategories();
        } catch (error) {
          console.error('Failed to delete category:', error);
          
          let errorMessage = '分类删除失败';
          
          if (error.response && error.response.data && error.response.data.message) {
            errorMessage += `：${error.response.data.message}`;
          } else {
            errorMessage += '，请重试';
          }
          
          ElMessage.error(errorMessage);
        }
      }).catch(() => {
        // 用户取消操作
      });
    };
    
    return {
      categories,
      loading,
      dialogVisible,
      isEdit,
      categoryForm,
      categoryForm: formData,
      rules,
      submitting,
      showAddDialog,
      showEditDialog,
      handleSubmit,
      handleDelete
    };
  }
});
</script>

<style scoped>
.admin-categories {
  margin-bottom: 40px;
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.page-title {
  margin: 0;
}
</style>
```

Let's create the Admin Feedback Management Page (src/views/admin/AdminFeedback.vue):

```vue
<template>
  <div class="admin-feedback">
    <h1 class="page-title">反馈管理</h1>
    
    <div class="filter-container">
      <el-radio-group v-model="statusFilter" @change="handleFilterChange">
        <el-radio-button label="">全部</el-radio-button>
        <el-radio-button label="PENDING">待处理</el-radio-button>
        <el-radio-button label="PROCESSED">已处理</el-radio-button>
        <el-radio-button label="REJECTED">已拒绝</el-radio-button>
      </el-radio-group>
    </div>
    
    <el-table
      :data="feedbacks"
      style="width: 100%"
      v-loading="loading"
      border
    >
      <el-table-column label="ID" prop="id" width="80" />
      
      <el-table-column label="用户" prop="username" width="120" />
      
      <el-table-column label="内容" prop="content" show-overflow-tooltip />
      
      <el-table-column label="提交时间" width="160">
        <template #default="scope">
          {{ formatDate(scope.row.createdAt) }}
        </template>
      </el-table-column>
      
      <el-table-column label="状态" width="100">
        <template #default="scope">
          <el-tag
            :type="getStatusTagType(scope.row.status)"
            size="small"
          >
            {{ getStatusLabel(scope.row.status) }}
          </el-tag>
        </template>
      </el-table-column>
      
      <el-table-column label="操作" width="200">
        <template #default="scope">
          <el-button
            type="primary"
            size="small"
            @click="showFeedbackDetail(scope.row)"
          >
            查看
          </el-button>
          
          <el-button
            v-if="scope.row.status === 'PENDING'"
            type="success"
            size="small"
            @click="updateStatus(scope.


用户:
继续

Claude:
I'll continue with the Admin Feedback Management Page (src/views/admin/AdminFeedback.vue):

```vue
<template>
  <div class="admin-feedback">
    <h1 class="page-title">反馈管理</h1>
    
    <div class="filter-container">
      <el-radio-group v-model="statusFilter" @change="handleFilterChange">
        <el-radio-button label="">全部</el-radio-button>
        <el-radio-button label="PENDING">待处理</el-radio-button>
        <el-radio-button label="PROCESSED">已处理</el-radio-button>
        <el-radio-button label="REJECTED">已拒绝</el-radio-button>
      </el-radio-group>
    </div>
    
    <el-table
      :data="feedbacks"
      style="width: 100%"
      v-loading="loading"
      border
    >
      <el-table-column label="ID" prop="id" width="80" />
      
      <el-table-column label="用户" prop="username" width="120" />
      
      <el-table-column label="内容" prop="content" show-overflow-tooltip />
      
      <el-table-column label="提交时间" width="160">
        <template #default="scope">
          {{ formatDate(scope.row.createdAt) }}
        </template>
      </el-table-column>
      
      <el-table-column label="状态" width="100">
        <template #default="scope">
          <el-tag
            :type="getStatusTagType(scope.row.status)"
            size="small"
          >
            {{ getStatusLabel(scope.row.status) }}
          </el-tag>
        </template>
      </el-table-column>
      
      <el-table-column label="操作" width="200">
        <template #default="scope">
          <el-button
            type="primary"
            size="small"
            @click="showFeedbackDetail(scope.row)"
          >
            查看
          </el-button>
          
          <el-button
            v-if="scope.row.status === 'PENDING'"
            type="success"
            size="small"
            @click="updateStatus(scope.row.id, 'PROCESSED')"
          >
            处理
          </el-button>
          
          <el-button
            v-if="scope.row.status === 'PENDING'"
            type="danger"
            size="small"
            @click="updateStatus(scope.row.id, 'REJECTED')"
          >
            拒绝
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <el-pagination
      v-if="total > 0"
      background
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      :page-size="pageSize"
      :current-page="currentPage"
      :page-sizes="[10, 20, 50, 100]"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
    
    <!-- 反馈详情对话框 -->
    <el-dialog
      v-model="dialogVisible"
      title="反馈详情"
      width="600px"
    >
      <div v-if="currentFeedback" class="feedback-detail">
        <div class="detail-item">
          <span class="label">ID:</span>
          <span>{{ currentFeedback.id }}</span>
        </div>
        
        <div class="detail-item">
          <span class="label">用户:</span>
          <span>{{ currentFeedback.username }}</span>
        </div>
        
        <div class="detail-item">
          <span class="label">状态:</span>
          <el-tag :type="getStatusTagType(currentFeedback.status)">
            {{ getStatusLabel(currentFeedback.status) }}
          </el-tag>
        </div>
        
        <div class="detail-item">
          <span class="label">提交时间:</span>
          <span>{{ formatDate(currentFeedback.createdAt) }}</span>
        </div>
        
        <div class="detail-content">
          <div class="label">反馈内容:</div>
          <div class="content">{{ currentFeedback.content }}</div>
        </div>
      </div>
      
      <template #footer>
        <el-button @click="dialogVisible = false">关闭</el-button>
        
        <template v-if="currentFeedback && currentFeedback.status === 'PENDING'">
          <el-button
            type="success"
            @click="updateStatusAndClose('PROCESSED')"
          >
            标记为已处理
          </el-button>
          
          <el-button
            type="danger"
            @click="updateStatusAndClose('REJECTED')"
          >
            标记为已拒绝
          </el-button>
        </template>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage } from 'element-plus';

export default defineComponent({
  name: 'AdminFeedback',
  setup() {
    const store = useStore();
    
    const statusFilter = ref('');
    const currentPage = ref(1);
    const pageSize = ref(10);
    const dialogVisible = ref(false);
    const currentFeedback = ref(null);
    
    const feedbacks = computed(() => store.getters['feedback/allFeedbacks']);
    const pagination = computed(() => store.getters['feedback/pagination']);
    const total = computed(() => pagination.value.totalElements);
    const loading = computed(() => store.getters['feedback/loading']);
    
    onMounted(() => {
      fetchFeedbacks();
    });
    
    const fetchFeedbacks = () => {
      if (statusFilter.value) {
        store.dispatch('feedback/fetchFeedbackByStatus', {
          status: statusFilter.value,
          page: currentPage.value - 1,
          size: pageSize.value
        });
      } else {
        store.dispatch('feedback/fetchAllFeedback', {
          page: currentPage.value - 1,
          size: pageSize.value
        });
      }
    };
    
    const handleFilterChange = () => {
      currentPage.value = 1;
      fetchFeedbacks();
    };
    
    const handleSizeChange = (val) => {
      pageSize.value = val;
      fetchFeedbacks();
    };
    
    const handleCurrentChange = (val) => {
      currentPage.value = val;
      fetchFeedbacks();
    };
    
    const formatDate = (dateString) => {
      if (!dateString) return '';
      
      const date = new Date(dateString);
      return date.toLocaleString();
    };
    
    const getStatusTagType = (status) => {
      switch (status) {
        case 'PENDING':
          return 'warning';
        case 'PROCESSED':
          return 'success';
        case 'REJECTED':
          return 'danger';
        default:
          return 'info';
      }
    };
    
    const getStatusLabel = (status) => {
      switch (status) {
        case 'PENDING':
          return '待处理';
        case 'PROCESSED':
          return '已处理';
        case 'REJECTED':
          return '已拒绝';
        default:
          return '未知';
      }
    };
    
    const showFeedbackDetail = (feedback) => {
      currentFeedback.value = feedback;
      dialogVisible.value = true;
    };
    
    const updateStatus = async (id, status) => {
      try {
        await store.dispatch('feedback/updateFeedbackStatus', { id, status });
        
        ElMessage.success(`反馈已${status === 'PROCESSED' ? '处理' : '拒绝'}`);
        
        // 刷新列表
        fetchFeedbacks();
      } catch (error) {
        console.error('Failed to update feedback status:', error);
        ElMessage.error('操作失败，请重试');
      }
    };
    
    const updateStatusAndClose = async (status) => {
      if (!currentFeedback.value) return;
      
      await updateStatus(currentFeedback.value.id, status);
      dialogVisible.value = false;
    };
    
    return {
      feedbacks,
      total,
      loading,
      statusFilter,
      currentPage,
      pageSize,
      dialogVisible,
      currentFeedback,
      handleFilterChange,
      handleSizeChange,
      handleCurrentChange,
      formatDate,
      getStatusTagType,
      getStatusLabel,
      showFeedbackDetail,
      updateStatus,
      updateStatusAndClose
    };
  }
});
</script>

<style scoped>
.admin-feedback {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}

.filter-container {
  margin-bottom: 20px;
}

.el-pagination {
  margin-top: 20px;
  text-align: center;
}

.feedback-detail {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.detail-item {
  display: flex;
  align-items: center;
}

.detail-content {
  display: flex;
  flex-direction: column;
}

.label {
  font-weight: bold;
  min-width: 80px;
  margin-right: 10px;
}

.content {
  white-space: pre-wrap;
  background-color: #f5f7fa;
  padding: 10px;
  border-radius: 4px;
  margin-top: 5px;
}
</style>
```

Let's create the Admin Users Management Page (src/views/admin/AdminUsers.vue):

```vue
<template>
  <div class="admin-users">
    <h1 class="page-title">用户管理</h1>
    
    <el-table
      :data="users"
      style="width: 100%"
      v-loading="loading"
      border
    >
      <el-table-column label="ID" prop="id" width="80" />
      
      <el-table-column label="头像" width="80">
        <template #default="scope">
          <el-avatar :size="40" :src="scope.row.avatar || defaultAvatar" />
        </template>
      </el-table-column>
      
      <el-table-column label="用户名" prop="username" />
      
      <el-table-column label="邮箱" prop="email" />
      
      <el-table-column label="角色" width="100">
        <template #default="scope">
          <el-tag
            :type="scope.row.role === 'ROLE_ADMIN' ? 'danger' : 'success'"
          >
            {{ scope.row.role === 'ROLE_ADMIN' ? '管理员' : '普通用户' }}
          </el-tag>
        </template>
      </el-table-column>
      
      <el-table-column label="操作" width="150">
        <template #default="scope">
          <el-button
            v-if="scope.row.role !== 'ROLE_ADMIN'"
            type="danger"
            size="small"
            icon="Delete"
            @click="handleDelete(scope.row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { defineComponent, ref, computed, onMounted } from 'vue';
import { useStore } from 'vuex';
import { ElMessage, ElMessageBox } from 'element-plus';
import defaultAvatar from '@/assets/default-avatar.png';

export default defineComponent({
  name: 'AdminUsers',
  setup() {
    const store = useStore();
    
    const users = ref([]);
    const loading = ref(false);
    const defaultAvatar = ref(defaultAvatar);
    
    onMounted(() => {
      fetchUsers();
    });
    
    const fetchUsers = async () => {
      try {
        loading.value = true;
        const response = await store.dispatch('auth/getAllUsers');
        users.value = response || [];
      } catch (error) {
        console.error('Failed to fetch users:', error);
        ElMessage.error('获取用户列表失败');
      } finally {
        loading.value = false;
      }
    };
    
    const handleDelete = (user) => {
      if (user.role === 'ROLE_ADMIN') {
        ElMessage.warning('管理员账户不能删除');
        return;
      }
      
      ElMessageBox.confirm(
        `确定要删除用户 "${user.username}" 吗？此操作不可恢复！`,
        '警告',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(async () => {
        try {
          await store.dispatch('auth/deleteUser', user.id);
          ElMessage.success('用户删除成功');
          fetchUsers();
        } catch (error) {
          console.error('Failed to delete user:', error);
          ElMessage.error('用户删除失败，请重试');
        }
      }).catch(() => {
        // 用户取消操作
      });
    };
    
    return {
      users,
      loading,
      defaultAvatar,
      handleDelete
    };
  }
});
</script>

<style scoped>
.admin-users {
  margin-bottom: 40px;
}

.page-title {
  margin-bottom: 20px;
}
</style>
```

Finally, let's create the NotFound Page (src/views/NotFound.vue):

```vue
<template>
  <div class="not-found">
    <div class="not-found-content">
      <el-result
        icon="warning"
        title="404"
        sub-title="抱歉，您访问的页面不存在"
      >
        <template #extra>
          <el-button type="primary" @click="goHome">返回首页</el-button>
        </template>
      </el-result>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import { useRouter } from 'vue-router';

export default defineComponent({
  name: 'NotFound',
  setup() {
    const router = useRouter();
    
    const goHome = () => {
      router.push('/');
    };
    
    return {
      goHome
    };
  }
});
</script>

<style scoped>
.not-found {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 70vh;
}

.not-found-content {
  text-align: center;
}
</style>
```

Now let's create some necessary static assets:

1. default-book-cover.jpg
2. default-avatar.png

These files should be placed in the `src/assets` directory. Since we can't generate these directly, you would need to add simple placeholder images.

Let's also update the auth.module.js to add the missing methods we've used in the Admin components:

```javascript
// Add these to the actions object in auth.module.js
getAllUsers({ commit }) {
  return new Promise((resolve, reject) => {
    commit('setLoading', true);
    AuthService.getAllUsers()
      .then(response => {
        commit('setLoading', false);
        resolve(response.data.data);
      })
      .catch(error => {
        commit('setLoading', false);
        reject(error);
      });
  });
},

deleteUser({ commit }, id) {
  return new Promise((resolve, reject) => {
    commit('setLoading', true);
    AuthService.deleteUser(id)
      .then(response => {
        commit('setLoading', false);
        resolve(response.data);
      })
      .catch(error => {
        commit('setLoading', false);
        reject(error);
      });
  });
},

changePassword({ commit }, passwordData) {
  return new Promise((resolve, reject) => {
    AuthService.changePassword(passwordData)
      .then(response => {
        resolve(response.data);
      })
      .catch(error => {
        reject(error);
      });
  });
},

deleteAccount({ commit }) {
  return new Promise((resolve, reject) => {
    AuthService.deleteAccount()
      .then(response => {
        resolve(response.data);
      })
      .catch(error => {
        reject(error);
      });
  });
}
```

And update the AuthService to add these methods:

```javascript
// Add these to the AuthService object
getAllUsers() {
  return api.get('/users/admin/all');
},

deleteUser(id) {
  return api.delete(`/users/admin/${id}`);
},

changePassword(passwordData) {
  return api.put('/users/me/password', passwordData);
},

deleteAccount() {
  return api.delete('/users/me');
}
```

## Project Organization and Deployment

Let's now create the instructions to combine all these components into a functional application.

### 1. Project Structure

The final project structure should look like this:

```
book-management-system/
├── backend/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com/
│   │   │   │       └── bookmanagement/
│   │   │   │           └── booksystem/
│   │   │   │               ├── config/
│   │   │   │               ├── controller/
│   │   │   │               ├── dto/
│   │   │   │               ├── entity/
│   │   │   │               ├── exception/
│   │   │   │               ├── repository/
│   │   │   │               ├── security/
│   │   │   │               ├── service/
│   │   │   │               │   └── impl/
│   │   │   │               ├── util/
│   │   │   │               └── BookSystemApplication.java
│   │   │   └── resources/
│   │   │       └── application.properties
│   │   └── test/
│   └── pom.xml
└── frontend/
    ├── public/
    ├── src/
    │   ├── assets/
    │   ├── components/
    │   ├── router/
    │   ├── services/
    │   ├── store/
    │   │   └── modules/
    │   ├── utils/
    │   ├── views/
    │   │   ├── admin/
    │   │   ├── auth/
    │   │   ├── book/
    │   │   └── user/
    │   ├── App.vue
    │   └── main.js
    ├── .env.development
    ├── .env.production
    ├── package.json
    └── vue.config.js
```

### 2. Integration Instructions

Here are the step-by-step instructions to set up and run the entire project:

#### Backend Setup:

1. Open IntelliJ IDEA and import the backend project.
2. Make sure you have Java 17 and MySQL 8.0+ installed.
3. Create a MySQL database named `book_management_system`.
4. Update `application.properties` with your MySQL credentials.
5. Run the `BookSystemApplication.java` main class to start the backend server.
6. The server will be available at `http://localhost:8080/api`.

#### Frontend Setup:

1. Open WebStorm or any terminal and navigate to the frontend project directory.
2. Run `npm install` to install all dependencies.
3. Create the necessary assets:
   - Create `src/assets/default-book-cover.jpg` - a simple book cover placeholder
   - Create `src/assets/default-avatar.png` - a simple user avatar placeholder
4. Run `npm run serve` to start the development server.
5. The frontend will be available at `http://localhost:8080`.

### 3. Initial Data Setup

For the system to function properly, you should create an admin user and some initial categories:

1. After starting the backend, use an API client (like Postman) to register a user:
   - Send a POST request to `http://localhost:8080/api/auth/register` with:
   ```json
   {
     "username": "admin",
     "email": "admin@example.com",
     "password": "admin123456"
   }
   ```

2. Update the user to be an admin in the database:
   ```sql
   UPDATE users SET role = 'ROLE_ADMIN' WHERE username = 'admin';
   ```

3. Log in as admin and create some initial categories (e.g., Fiction, Non-fiction, Science, History, etc.).

4. Start adding books to the system.

### 4. Production Deployment

To deploy the application to production:

#### Backend:
1. Build the backend using Maven:
   ```
   mvn clean package
   ```
2. Run the generated JAR file:
   ```
   java -jar target/booksystem-0.0.1-SNAPSHOT.jar
   ```

#### Frontend:
1. Build the frontend:
   ```
   npm run build
   ```
2. Deploy the generated `dist` folder to a web server (Apache, Nginx, etc.).

### 5. Advanced Features

The system includes several advanced features you can use:

1. **Book Recommendations**: The system automatically recommends books based on user behavior.
2. **Search Functionality**: Users can search books by title, author, or ISBN.
3. **User Feedback**: Users can provide feedback to administrators.
4. **Admin Dashboard**: Administrators have a comprehensive dashboard to manage the system.

## Conclusion

This complete book management system provides a robust solution for managing books, users, and related information. It includes all the features requested in the initial requirements:

- User registration, login, and profile management
- Book CRUD operations with cover images
- Categorization of books
- Favorites and browsing history
- Book recommendations
- User feedback system
- Admin panel for system management

The separation of backend and frontend makes the system modular and easy to maintain. The use of Spring Boot for the backend and Vue.js for the frontend provides a solid foundation for further enhancements.


用户:
请将Project Organization and Deployment之后的内容翻译成中文

Claude:
## 项目组织与部署

### 1. 项目结构

最终的项目结构应该如下所示：

```
book-management-system/
├── backend/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com/
│   │   │   │       └── bookmanagement/
│   │   │   │           └── booksystem/
│   │   │   │               ├── config/
│   │   │   │               ├── controller/
│   │   │   │               ├── dto/
│   │   │   │               ├── entity/
│   │   │   │               ├── exception/
│   │   │   │               ├── repository/
│   │   │   │               ├── security/
│   │   │   │               ├── service/
│   │   │   │               │   └── impl/
│   │   │   │               ├── util/
│   │   │   │               └── BookSystemApplication.java
│   │   │   └── resources/
│   │   │       └── application.properties
│   │   └── test/
│   └── pom.xml
└── frontend/
    ├── public/
    ├── src/
    │   ├── assets/
    │   ├── components/
    │   ├── router/
    │   ├── services/
    │   ├── store/
    │   │   └── modules/
    │   ├── utils/
    │   ├── views/
    │   │   ├── admin/
    │   │   ├── auth/
    │   │   ├── book/
    │   │   └── user/
    │   ├── App.vue
    │   └── main.js
    ├── .env.development
    ├── .env.production
    ├── package.json
    └── vue.config.js
```

### 2. 集成说明

以下是设置和运行整个项目的分步说明：

#### 后端设置：

1. 打开IntelliJ IDEA并导入后端项目。
2. 确保已安装Java 17和MySQL 8.0+。
3. 创建一个名为`book_management_system`的MySQL数据库。
4. 使用您的MySQL凭据更新`application.properties`。
5. 运行`BookSystemApplication.java`主类以启动后端服务器。
6. 服务器将在`http://localhost:8080/api`上可用。

#### 前端设置：

1. 打开WebStorm或任何终端，并导航到前端项目目录。
2. 运行`npm install`以安装所有依赖项。
3. 创建必要的资源：
   - 创建`src/assets/default-book-cover.jpg` - 一个简单的书籍封面占位图
   - 创建`src/assets/default-avatar.png` - 一个简单的用户头像占位图
4. 运行`npm run serve`以启动开发服务器。
5. 前端将在`http://localhost:8080`上可用。

### 3. 初始数据设置

为了使系统正常运行，您应该创建一个管理员用户和一些初始分类：

1. 在启动后端后，使用API客户端（如Postman）注册用户：
   - 向`http://localhost:8080/api/auth/register`发送POST请求，内容为：
   ```json
   {
     "username": "admin",
     "email": "admin@example.com",
     "password": "admin123456"
   }
   ```

2. 在数据库中将用户更新为管理员：
   ```sql
   UPDATE users SET role = 'ROLE_ADMIN' WHERE username = 'admin';
   ```

3. 以管理员身份登录并创建一些初始分类（例如，小说、非小说、科学、历史等）。

4. 开始向系统添加书籍。

### 4. 生产部署

要将应用程序部署到生产环境：

#### 后端：
1. 使用Maven构建后端：
   ```
   mvn clean package
   ```
2. 运行生成的JAR文件：
   ```
   java -jar target/booksystem-0.0.1-SNAPSHOT.jar
   ```

#### 前端：
1. 构建前端：
   ```
   npm run build
   ```
2. 将生成的`dist`文件夹部署到Web服务器（Apache、Nginx等）。

### 5. 高级功能

该系统包含几个您可以使用的高级功能：

1. **图书推荐**：系统根据用户行为自动推荐图书。
2. **搜索功能**：用户可以通过标题、作者或ISBN搜索图书。
3. **用户反馈**：用户可以向管理员提供反馈。
4. **管理员仪表盘**：管理员拥有综合仪表盘来管理系统。

## 结论

这个完整的图书管理系统提供了一个强大的解决方案，用于管理图书、用户和相关信息。它包括了初始需求中要求的所有功能：

- 用户注册、登录和个人资料管理
- 图书CRUD操作，带有封面图片
- 图书分类
- 收藏和浏览历史
- 图书推荐
- 用户反馈系统
- 系统管理的管理员面板

后端和前端的分离使系统模块化并易于维护。使用Spring Boot作为后端和Vue.js作为前端为进一步增强提供了坚实的基础。


