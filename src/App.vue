<template>
  <router-view/>
</template>

<script>
  export default {
    name: 'App'
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Montserrat:100,300,600,900');

  body {
    background: #f3f3f3;
    font-family: 'Montserrat', sans-serif;
  }

  ::selection {
    background: black;
    color: #fff;
  }

  *:focus {
    outline: none;
  }

  button, a {
    cursor: pointer;
  }
</style>
