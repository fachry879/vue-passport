<template>
  <div class="container-fluid mt-5">
    <div class="row">
      <div class="col-md-4">
        <div class="card">
          <div class="card-body">
            MAIN MENU
            <hr />
            <ul class="list-group">
              <router-link :to="{ name: 'dashboard' }" class="list-group-item text-dark text-decoration-none">DASHBOARD</router-link>
              <li @click.prevent="logout" class="list-group-item text-dark text-decoration-none" style="cursor: pointer">LOGOUT</li>
            </ul>
          </div>
        </div>
      </div>
      <div class="col-md-8">
        <div class="card">
          <div class="card-body">
            DASHBOARD, Selamat Datang <strong>{{ user.name }}</strong>
            <hr />
            <router-link :to="{ name: 'create' }" class="btn btn-md btn-success">TAMBAH POST</router-link>
            <table class="table table-striped table-bordered mt-4">
              <thead class="thead-dark">
                <tr>
                  <th scope="col">TITLE</th>
                  <th scope="col">CONTENT</th>
                  <th scope="col">OPTIONS</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(post, index) in posts" :key="index">
                  <td>{{ post.title }}</td>
                  <td>{{ post.content }}</td>
                  <td class="text-center">
                    <router-link :to="{ name: 'edit', params: { id: post.id } }" class="btn btn-sm btn-primary mr-1">EDIT</router-link>
                    <button @click.prevent="postDelete(post.id)" class="btn btn-sm btn-danger ml-1">DELETE</button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";

export default {
  name: "Index-dashboard",
  setup() {
    //state token
    const token = localStorage.getItem("token");

    //inisialisasi vue router on Composition API
    const router = useRouter();

    //state user
    const user = ref("");

    //mounted properti
    onMounted(() => {
      //check Token exist
      if (!token) {
        return router.push({
          name: "login",
        });
      }

      //get data user
      axios.defaults.headers.common.Authorization = `Bearer ${token}`;
      axios
        .get("http://localhost:8000/api/user")
        .then((response) => {
          //console.log(response.data.name)
          user.value = response.data;
        })
        .catch((error) => {
          console.log(error.response.data);
        });
    });

    //crud post

    //reactive state
    let posts = ref([]);

    //mounted
    onMounted(() => {
      //get API from Laravel Backend
      axios
        .get("http://localhost:8000/api/post")
        .then((response) => {
          //assign state posts with response data
          posts.value = response.data.data;
        })
        .catch((error) => {
          console.log(error.response.data);
        });
    });

    //method logout
    function logout() {
      //logout
      axios.defaults.headers.common.Authorization = `Bearer ${token}`;
      axios
        .post("http://localhost:8000/api/logout")
        .then((response) => {
          if (response.data.success) {
            //remove localStorage
            localStorage.removeItem("token");

            //redirect ke halaman login
            return router.push({
              name: "login",
            });
          }
        })
        .catch((error) => {
          console.log(error.response.data);
        });
    }

    function postDelete(id) {
      //delete data post by ID
      axios
        .delete(`http://localhost:8000/api/post/${id}`)
        .then(() => {
          // splice posts
          const index = this.posts.findIndex((post) => post.id === id); // find the post index
          if (~index) {
            // if the post exists in array
            this.posts.splice(index, 1);
          }
        })
        .catch((error) => {
          console.log(error.response.data);
        });
    }

    return {
      token, // <-- state token
      user, // <-- state user
      posts,
      postDelete,
      logout, // <-- method logout
    };
  },
};
</script>
