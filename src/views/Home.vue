<template>

  <div class="home">
    {{ info }}
  </div>
</template>

<script>

export default {
  data(){
    return{
      info:null
    }
  },
  mounted(){
   this.$http.get(`${process.env.VUE_APP_URL}/api/v1/mask`).then((response) => {
     console.log(response.data)
     this.info = response.data
})
}
}
</script>
