<template>
    <div>
       <nav class="navbar navbar-expand-lg navbar-light bg-light">
          <div class="container-fluid">
            <a class="navbar-brand" href="#">VueSuperShop</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
              <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
              <ul class="navbar-nav">
              <div v-if="isAuth">
                <li class="nav-item">
                  <!-- <a href="/?useremail=<%= useremail %>&error=no">
                    Домой
                  </a> -->
                  <router-link :to="{name:'Home'}">
                    Домой
                  </router-link>
                </li>
                <li class="nav-item">
                  <!-- <a class="nav-item" @click="getBucket" href="/users/bucket?useremail=<%= useremail %>">
                      Корзина
                  </a> -->
                  <router-link class="nav-item" :to="{name:'Bucket', query:{ 'useremail': useremail }}">
                      Корзина
                  </router-link>
                </li>
              </div>
              <div v-else >
                <li class="nav-item">
                  <!-- <a href="/users/login">
                      Вход
                  </a> -->
                  <router-link class="nav-item" :to="{name:'UsersLogin'}">
                      Вход
                  </router-link>
                </li>
                <li class="nav-item">
                    <!-- <a href=/users/register>
                        Регистрация
                    </a> -->
                  <router-link class="nav-item" :to="{name:'UsersRegistry'}">
                    Регистрация
                  </router-link>
                </li>
              </div>
              <div v-if="isAuth">
                <li>
                    <span class="badge bg-primary">{{ useremail }}</span>
                </li>
                <li>
                    <span class="badge bg-primary"><a class="badge bg-primary" @click="logoutUser" >Выйти</a></span>
                </li>
                <li>
                    <span class="badge bg-primary"><a class="badge bg-primary" href="/users/amount?useremail=<%= useremail %>&amount=<%= 0 %>">+</a></span>
                </li>
              </div>
            </ul>
          </div>
        </div>
      </nav>
      <div class="form-control">
          <input class="myamount" v-model="myamount" type="text">
      </div>
      <a @click="addAmount" class="form-control btn btn-primary">
          Пополнить счёт
      </a>
      <!-- <router-link href="/users/amount?useremail=<%= useremail %>&amount=<%= 20 %>" class="amountBtn form-control btn btn-primary">
          Пополнить счёт
      </router-link>  -->
  
      <!-- footer -->
      <p class="mt-5 mb-3 text-muted">© {{ new Date().toLocaleDateString() }}</p> 
    </div>
</template>

<script>
export default {
  name: 'Amount',
  data(){
    return {
      myamount: 0,
      useremail: window.localStorage.getItem('auth') == 'true' ? window.localStorage.getItem('useremail') : "",
      isAuth: window.localStorage.getItem('auth') == 'true'
    }
  },
  methods:{
    async addAmount(){
      localStorage.setItem("userlogin", "true")
      await fetch(`http://localhost:4000/users/amount?useremail=${this.useremail}&amount=${this.myamount}`, {
          mode: 'cors',
          method: 'GET'
        }).then(response => response.body).then(rb  => {
          const reader = rb.getReader()
          return new ReadableStream({
            start(controller) {
              function push() {
                reader.read().then( ({done, value}) => {
                  if (done) {
                    console.log('done', done);
                    controller.close();
                    return;
                  }
                  controller.enqueue(value);
                  console.log(done, value);
                  push();
                })
              }
              push();
            }
          });
      }).then(stream => {
          return new Response(stream, { headers: { "Content-Type": "text/html" } }).text();
        })
        .then(result => {
          console.log(result);
          if(!result.includes("user not found")){
            // console.log(JSON.parse(result))
            window.localStorage.setItem("auth", "true");
            window.localStorage.setItem("useremail", this.useremail);
            this.$router.push('/')
          }
        });
    }
  }
}
</script>