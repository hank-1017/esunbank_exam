<template>
  <h1>全部文章</h1>
  <br>
  <div v-for="post in posts">
    <div class="card">
      <div class="card-header">
        發文者：{{ post.user.userName }}
        <span v-if="!post.editOpen">
        <button class="btn btn-outline-danger btn-sm float-right" @click="deletePost(post.postID)">刪除</button>
        <button class="btn btn-outline-success btn-sm float-right" @click="post.editOpen = true">編輯</button>
          </span>
        <span v-else>
          <button class="btn btn-outline-success btn-sm float-right" @click="editPost(post)">送出</button>
          <button class="btn btn-outline-danger btn-sm float-right"
                  @click="post.editOpen = false&getPosts()">取消</button>
        </span>
        <span class="float-right">時間：{{ post.createdTime }} #</span>
      </div>
      <div class="card-body">
        <textarea v-if="!post.editOpen" class="form-control" disabled>{{ post.content }}</textarea>
        <textarea v-else class="form-control" v-model="post.content">{{ post.content }}</textarea>
        <br>
        <span v-for="comment in post.comment">
          <span>{{ comment.user.userName }}: {{ comment.content }}</span>
          <span class=" float-right">{{
              format(new Date(comment.createdTime), "a hh 時 mm 分 ss 秒 ## yyyy 年 MM 月 dd 日 EEEE", {locale: zhTW})
            }}</span>
          <br>
        </span>
        <br>
        <div v-if="post.commentOpen">
          留言: <input type="text" class="form-control" v-model="comment"/>
          <button class="btn btn-outline-primary btn-sm" @click="sendComment(post.postID)">送出</button>
        </div>
        <div v-else>
          <button class="btn btn-outline-primary btn-sm" @click="post.commentOpen = !post.commentOpen">留言</button>
        </div>

      </div>
    </div>
    <br>
  </div>

</template>

<script setup>
import axios from "axios";
import {onMounted, ref} from "vue";
import {useRouter} from 'vue-router';
import {format} from 'date-fns'
import {zhTW} from 'date-fns/locale'

const router = useRouter();
const posts = ref([]);
const comment = ref('');
const URL = import.meta.env.VITE_API_JAVAURL;

onMounted(() => {
  getPosts();
});

// 取得全部文章
const getPosts = async () => {
  try {
    const response = await axios.get(`${URL}post/show`, {withCredentials: true});
    // console.log("posts: " + response.data);
    posts.value = response.data.map(post => ({
      ...post,
      createdTime: format(new Date(post.createdTime), "a hh 時 mm 分 ss 秒 ## yyyy 年 MM 月 dd 日 EEEE", {locale: zhTW}),
      commentOpen: false,
      editOpen: false,
    })).reverse();
  } catch (error) {
    console.error('AJAX error:', error);
  }
}

// 更新文章
const editPost = async (post) => {
  try {
    post.createdTime = Date.parse(post.createdTime);
    const response = await axios.put(`${URL}post/edit`, post, {withCredentials: true});
    console.log("edit post: " + response.data);
    if (response.data === "Y") {
      alert("文章已更新!!")
      post.editOpen = false;
      await getPosts();
    } else if (response.data === "NotLogin") {
      alert("請先登入")
      post.editOpen = false;
      await router.push('/login');
    } else if (response.data === "NotOwner") {
      alert("非文章擁有者")
      post.editOpen = false;
      await getPosts();
    } else {
      alert("系統錯誤")
      post.editOpen = false;
      await getPosts();
    }

  } catch (error) {
    console.error('AJAX error:', error);
  }
}

// 刪除文章
const deletePost = async (id) => {
  try {
    const response = await axios.delete(`${URL}post/delete/${id}`, {withCredentials: true});
    console.log("delete post: " + response.data);
    if (response.data === "Y") {
      alert("文章已刪除!!")
      await getPosts();
    }
  } catch (error) {
    console.error('AJAX error:', error);
  }
}

// 留言
const sendComment = async (postID) => {
  try {
    const formData = new FormData();
    formData.append('postid', postID);
    formData.append('content', comment.value);
    const response = await axios.post(`${URL}comment/add`, formData, {withCredentials: true});
    console.log("send comment: " + response.data);
    if (response.data === "Y") {
      comment.value = '';
      await getPosts();
    } else {
      alert("請先登入")
      await router.push('/login');
    }
  } catch (error) {
    console.error('AJAX error:', error);
  }
}


</script>


<style scoped>
.float-right {
  float: right;
}
</style>