<template>
  <div class="home  ">
    <div class="card" style="width: 18rem;">
      
            <!-- <img src="..." c lass="card-img-top" alt="not Image"> -->
            <div class="card-body" :v-if="product.length">
                <h5 class="card-title">{{ product.name }}</h5>
                <p class="card-text">{{ product.price }}</p>
                <!-- <a href="/users/bucket/add/<%= product.id %>" class="btn btn-primary">Добавить в корзину</a> -->
                <router-link :to="`/users/bucket/add/${product.id}`" class="btn btn-primary">Добавить в корзину</router-link>
            </div>
          </div>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'Product',
  data(){
    return{
      product: {},
      isAuth: window.localStorage.getItem('auth') == 'true'
    }
  },
  async mounted(){
    console.log(this.$route.params.productID)
    fetch(`http://localhost:4000/product/${this.$route.params.productID}`, {
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
        this.product = JSON.parse(result)
      });
  }
}
</script>
