<template>
  <div class="test">{{hello}}</div>
</template>
<script>
export default {
  name: 'test',
  data () {
    return {
      hello: 'hello world'
    }
  }
}
</script>
<style lang="less" scoped>
  .test {
    color: #42b984;
    font-size: 1.1rem;
    font-weight: 600;
  }
</style>
