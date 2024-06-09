<template>
  <div>
    <svg-icon icon-class="github" @click="goto" />
  </div>
</template>

<script>
export default {
  name: "SourceGit",
  data() {
    return {
      url: "https://gitee.com/cx293824/lili-ui.git",
    };
  },
  methods: {
    goto() {
      window.open(this.url);
    },
  },
};
</script>
