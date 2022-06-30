<script>
import FileWorker from "@/FileWorker"
import LevelComponent from "@/LevelComponent";

export default {
  components: {LevelComponent, FileWorker},
  data() {
    return {
      json: "",
      el: "",
      head: null,
      levelTitles: [],
      levels: [],
      levelTitlesText: [],
      levelId: 0,
      newJson: null
    }
  },
  methods: {
    sortJson() {
      this.el = this.json.el;
      this.head = this.json.data.head;
      this.levelTitles = this.json.data.body.levels_title;
      // this.levelTitles.sort((a, b) => a.id - b.id);
      // this.levels = this.json.data.body.level;

      // let id = 0;
      // let nextLevel = this.json.data.body.level;
      let curLevel = this.json.data.body.level;
      curLevel._id = this.levelId++;
      this.levels.push(curLevel);
      // console.log(nextLevel)
      // while (nextLevel != null) {
      //   for (let k in nextLevel) {
      //     let curLevel = nextLevel;
      //     // console.log(curLevel);
      //     if (nextLevel[k].level !== undefined) {
      //       nextLevel = nextLevel[k].level;
      //     }
      //     else {
      //       nextLevel = null;
      //     }
      //     // delete curLevel[k].level;
      //     curLevel._id = id++;
      //     this.levels.push(curLevel);
      //     if (nextLevel === null){
      //       break;
      //     }
      //   }
      // }
    },
    getLevelTitle(id){
      return this.levelTitlesText[id];
    },
    addSubLevel(level){
      if (this.levels.find((item) => item === level)){
        return;
      }
      level._id = this.levelId++;
      this.levels.push(level);
    },
    removeSubLevels(id){
      this.levels = this.levels.filter((t) => t._id <= id)
      this.levelId = id + 1;
    },
    closeLevel(id){
      this.levels.splice(id, 1)
    },
    removeKeys(obj){
      if (!obj) {
        return;
      }
      if (!obj.level){
        return
      }
      delete obj.level._id;
      delete obj.toDelete;
      for (let k in obj.level){
        console.log(obj.level[k])
        this.removeKeys(obj.level[k])
      }
    },
    generateJson(){
      let nj = {};
      nj.el = this.el;
      nj.data = {};
      nj.data.head = this.head;
      nj.data.body = {levels_title: this.levelTitles}
      let levelCopy = JSON.parse(JSON.stringify(this.json.data.body));
      this.removeKeys(levelCopy);
      nj.data.body = levelCopy
      this.newJson = nj;
    }
  },
  watch: {
    json(newJson, oldJson){
      if (oldJson && oldJson !== "not_json"){
        return
      }
      if (newJson && newJson !== "not_json") {
        this.sortJson();
        this.levelTitlesText = [];
        for (let levelTitle in this.levelTitles) {
          this.levelTitlesText.push(levelTitle);
        }
      }
    }
  },
  updated() {
  }
}
</script>

<template>
  <FileWorker :new-json="newJson" @content="(newValue) => {this.json = newValue}"
              @isJson="(is) => this.json = 'not_json'" @generateJson="() => generateJson()"></FileWorker>
  <h1 v-if="json !== 'not_json' && json && head">{{head.title}}</h1>
  <pre v-else-if="json === 'not_json'">Not json!</pre>
  <pre v-else>Nothing</pre>
<!--  <div v-for="val in levelTitlesText" :key="val">{{val}}</div>-->
  <div class="tables">
    <div v-for="level in levels" :key="level" class="level">
      <LevelComponent :passedLevel="level" :title="getLevelTitle(level._id)" :level-id="level._id"
                      @subLevel="(level, id) => { removeSubLevels(id); addSubLevel(level) }" @close="(id) => closeLevel(id)"
      ></LevelComponent>
    </div>
  </div>

</template>

<style>

.tables {
  display: flex;
  flex-direction: row;
}

</style>