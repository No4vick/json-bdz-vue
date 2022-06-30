<template>
  <div>
    <input type="file" ref="doc" @change="readFile()" />
    <button @click="$emit('generateJson');">Скачать</button>
<!--    <pre>{{ content }}</pre>-->
  </div>
</template>

<script>

import {jsmin} from "jsmin";

export default {
  name: "FileWorker",
  emits: ['content', 'isJson', 'generateJson'],
  props: {
    newJson: Object
  },
  data() {
    return {
      file: null,
      content: null,
      isJson: false
    }
  },
  methods: {
    readFile() {
      this.file = this.$refs.doc.files[0];
      if (!this.file.name.endsWith(".json")){
        this.$emit('isJson', false);
        return;
      }
      const reader = new FileReader();
      reader.onload = (res) => {
        this.content = JSON.parse(jsmin(res.target.result));
        this.$emit('content', this.content);
      }
      reader.readAsText(this.file);
    },
    saveFile() {
      // const data = JSON.stringify(this.arr)
      let data = JSON.stringify(this.newJson);
      const blob = new Blob([data], {type: 'text/plain'})
      // const e = document.createEvent('MouseEvents');
      const a = document.createElement('a');
      const e = new MouseEvent('click', {
          bubbles: true,
          cancelable: false,
      })
      a.download = "test.json";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
      // e.initEvent('click', true, false);
      a.dispatchEvent(e);
    }
  },
  watch: {
    newJson(newJson){
      console.log(newJson)
      this.saveFile();
    }
  }
}
</script>

<style scoped>

</style>