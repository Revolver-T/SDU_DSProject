<template class="flamingo">
  <graphToTree></graphToTree>
</template>

<script>
import graphToTree from './components/graphToTree.vue';
export default {
  components: {
    graphToTree
  },

};
</script>

<style>
.flamingo {
    background-image: radial-gradient(73% 147%, #EADFDF 59%, #ECE2DF 100%),
    radial-gradient(91% 146%, rgba(255,255,255,0.50) 47%, rgba(0,0,0,0.50) 100%);
    background-blend-mode: screen;
}

</style>
