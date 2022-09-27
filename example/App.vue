
<script lang='ts'>
import { ref } from 'vue'
import { data } from '../src/listdata'

</script>
<script setup lang='ts'>
import virtualList from '../src/index'
const list = ref(data)
</script>

<template>
  <virtualList
    class="list-demo"
    :data="list"
    :initSize="6"
    :bufferScale="2"
  >
    <template #default="{item}">
      <div>
        {{item.index}}.
        {{item.value}}
      </div>
    </template>
  </virtualList>
</template>

<style lang="scss" scoped>
  .list-demo {
    width: 400px;
    height: 600px;
    margin: 100px auto;
  }
</style>
