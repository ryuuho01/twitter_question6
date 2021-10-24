<template>
  <div>
<!-- -------------------------左　　　側--------------------------- -->
    <div class="flex">
      <div class="container-left">
        <p><NuxtLink class="homettl" to="/">　ホーム</NuxtLink></p><br>
        <a class="logoutttl" @click="logout">　ログアウト</a>
        <div class="share">
          <p>シェア</p><br>

          <textarea v-model="newPost" name="post" cols="30" rows="7" id="share"></textarea><br>
          <button @click="insertPost" class="share-button">シェアする</button>
        </div>
      </div>
<!-- -------------------------右　　　側--------------------------- -->
      <div class="container-right">
        <div class="description">
          <p>ホーム</p>
        </div>
        <div class="postdescription" v-for="item in postCurrent" :key="item.id">
          <!-- <form @submit.prevent="insertLike" action="http://127.0.0.1:8000/api/like/" method="POST"> -->
            <input :value="userEmail" name="client_email" type="hidden">
            <input :value="item.id" name="post_id" type="hidden">
            <button @click="insertLike">Like</button>
          <!-- </form> -->
          <p>投稿のID：{{item.id}}</p>
          <p>likeの数：{{item.likes}}</p>
          <p>投稿者の名前：{{item.name}}</p>
          <p>投稿内容：{{item.post}}</p><br>
        </div>
      </div>

    </div>

  </div>

</template>

<script>
import firebase from '~/plugins/firebase'
import axios from "axios";
export default {
  data() {
    return {
      post_id: "",
      userEmail: "",
      clientId: "",
      newPost: "",
      postId: "",
      postCurrent: [],
      updatePostId: "",
    }
  },

  created() {
    this.getloginInfo();
  },
  mounted() {
    axios
      .get("http://127.0.0.1:8000/api/post/")
      .then((response) => (this.postCurrent = response.data.data));
  },

  methods: {
    // -----------↓FireBase↓----------
    logout() {
      firebase
        .auth()
        .signOut()
        .then(() => {
          alert('ログアウトが完了しました')
          this.loginCheck = false
          this.$router.replace('/')
        })
    },
    getloginInfo() {
        firebase.auth().onAuthStateChanged((user) => {
        if (user) {
          console.log("sign in");
          this.userEmail = user.email;
        } else {
          console.log("sign out");
          this.$router.replace('/')
        }
      });
    },
    // -----------↑FireBase↑----------
    async getclientInfo() {
      const user = await firebase.auth().currentUser.email;
      console.log(user);
      return user;
    },

    async getPost() {
      // postデータの取得
      const postresData = await this.$axios.get("http://127.0.0.1:8000/api/post/");
      this.postCurrent = postresData.data.data;
    },

    async insertPost() {
      const user = firebase.auth().currentUser;
      if(user !== null) {
        let Id = 0;
        const email = user.email;
        const clientresData = await this.$axios.get("http://127.0.0.1:8000/api/client/");
        for(let i =0; i < clientresData.data.data.length; i++) {
          if(clientresData.data.data[i]["email"] == email) {
            Id = clientresData.data.data[i]["id"];
          };
        }
        this.clientId = Id;
        
        const sendpostData = {
          post: this.newPost,
          client_id: this.clientId,
        };
        await this.$axios.post("http://127.0.0.1:8000/api/post/", sendpostData);
        this.newPost = '';

        this.getPost();
      };
    },

    async insertLike() {
      const user = firebase.auth().currentUser;
      if(user !== null) {
        let Id = 0;
        const email = user.email;
        const clientresData = await this.$axios.get("http://127.0.0.1:8000/api/client/");
        for(let i =0; i < clientresData.data.data.length; i++) {
          if(clientresData.data.data[i]["email"] == email) {
            Id = clientresData.data.data[i]["id"];
          };
        }
        this.clientId = Id;
        const Data = {
          client_email: this.userEmail,
          client_id: this.clientId,
          post_id: "34",
        }
        await axios
        .post("http://127.0.0.1:8000/api/like/",Data);
        await axios
        .get("http://127.0.0.1:8000/api/post/")
        .then((response) => (this.postCurrent = response.data.data));
      };
    },
  },
}
</script>

<style>
.flex {
  display: flex;
}
.container-left {
  color: white;
  margin-right: 20px;
}
.container-right {
  color: white;
  width: 100%;
  margin-right: 10px;
}
.homettl {
  text-decoration: none;
  color: white;
}
.homettl::before {
  margin-left: 10px;
  content: "";
  background: url(/home.png);
  background-size: cover;
  vertical-align: bottom;
  height: 20px;
  width: 20px;
  display: inline-block;
}
.logoutttl {
  cursor: pointer;
}
.logoutttl::before {
  margin-left: 10px;
  content: "";
  background: url(/logout.png);
  background-size: cover;
  vertical-align: bottom;
  height: 20px;
  width: 20px;
  display: inline-block;
}

.share {
  padding: 30px 0 0 10px;
  position: relative;
}

textarea {
  resize:none;
  background: rgb(31, 33, 42);
  border-radius: 10px;
  color: white;
}

.share-button {
  margin-top: 20px;
  border: none;
  position: absolute;
  right: 0%;
}

.description {
  border: 1px solid rgb(129, 129, 129);
  padding: 15px 10px;
}

.postdescription {
  border: 1px solid rgb(129, 129, 129);
  padding: 15px 10px;
  border-top: none;
}

li {
  list-style-type: none;
}

</style>