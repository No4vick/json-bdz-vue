<template>
  <div class="container">
    <div class="header">{{title? title : "Title Here"}}</div>
    <div class="filters">
      <ol class="sortingBtns">
        <li><button class="filterBtn" @click="sorting='order'; sortingDirection=!sortingDirection">ПОР</button></li>
        <li><button class="filterBtn" @click="sorting='alphabet'; sortingDirection=!sortingDirection">АЛФ</button></li>
      </ol>
      <input type="text" class="filterText" v-model="wordFilter">
    </div>
    <div class="itemsDisplay">
      <ul v-for="field in filteredFields" :key="field" class="itemList">
        <li class="listItem" style="display: flex; flex-direction: row;">
            <input type="checkbox" v-model="this.level[field].toDelete">
            <button @click="showEditModal = true; addEditModal(field)">РЕД</button>
            <div @click="openSubLevel(field)" v-bind:style="clickableLevel(field)" v-if="selectedField !== field">{{field}}</div>
            <div @click="openSubLevel(field)" v-bind:style="clickableLevel(field)" v-else><b><i>{{field}}</i></b></div>
        </li>
      </ul>
    </div>
    <div class="editBtns">
      <div class="addBlock">
        <div class="addBtn" @click="showAddModal = true; addNewModal()">+</div>
        <a>Добавить</a>
      </div>
      <div class="deleteBlock">
        <div class="delBtn" @click="delSelected()">-</div>
        <a>Удалить выбранные</a>
      </div>
    </div>
  </div>
  <Teleport to="body" v-for="addModal in addModals" :key="addModal">
    <!-- use the modal component, pass in the prop -->
    <AddModal :show="showAddModal" :id="addModal.id" :title="addModal.title" :height="addModal.height"
              @close="(id, title) => {removeAddModal(id, title); abortChanges()}"
              @placeOnTop="(id, title) => moveAddModalTop(id, title)"
              @save="(id, title, field) => {addItem(field); removeAddModal(id, title)}">
      <template #header>
        <h3>Добавление элемента</h3>
      </template>
      <template>
      </template>
    </AddModal>
  </Teleport>
  <Teleport to="body" v-for="editModal in editModals" :key="editModal">
    <!-- use the modal component, pass in the prop -->
    <EditModal :show="showEditModal" :id="editModal.id" :title="editModal.title" :height="editModal.height"
               :prop="editModal.prop"
              @close="(id, title) => { removeEditModal(id, title); abortChanges() }"
              @placeOnTop="(id, title) => moveEditModalTop(id, title)"
              @save="() => { removeEditModal(id, title); saveChanges()}">
      <template #header>
        <h3>Просмотр {{editModal.field}}</h3>
      </template>
      <template>
      </template>
    </EditModal>
  </Teleport>
</template>

<script>
import AddModal from "@/AddModal";
import EditModal from "@/EditModal"
export default {
  name: "LevelComponent",
  components: {AddModal, EditModal},
  props: {
    title: String,
    passedLevel: Object,
    levelId: Number
  },
  emits: ['subLevel', 'close'],
  data(){
    return {
      fields: [],
      showAddModal: false,
      addModalsId: 0,
      modalsMaxHeight: 0,
      addModals: [],
      editModals: [],
      editModalsId: 0,
      showEditModal: false,
      id: 0,
      levelChanged: false,
      level: null,
      oldLevel: null,
      selectedField: null,
      wordFilter: "",
      sorting: "order",
      sortingDirection: true // true - прямо, false - обратно
    }
  },
  methods: {
    addItem(field){
      // this.testFields.push({key: field.field, value: field.value});
      if (this.fields.find((t) => t === field.title)){
        return;
      }
      this.fields.push(field.title);
      this.level[field.title] = { toDelete: false };
      this.level[field.title][field.field] = field.value;
    },
    addNewModal(){
      this.addModals.push({id: this.addModalsId++, title: `sad${this.addModalsId}`, height: this.modalsMaxHeight++});
    },
    removeAddModal(id, title){
      this.addModalsId--
      this.addModals = this.addModals.filter((item) => item.id !== id && item.title !== title)
    },
    moveAddModalTop(id, title){
      this.modalsMaxHeight++;
      this.addModals.find((t) => t.id === id && t.title === title).height = this.modalsMaxHeight;
    },
    addEditModal(field){
      let _prop;
      if (!this.level[field].prop){
        _prop = this.level[field];
      }
      else {
        _prop = this.level[field].prop;
      }
      this.editModals.push({id: this.editModalsId++, title: `sad${this.editModalsId}`, height: this.modalsMaxHeight++,
                            field: field, prop: _prop});
    },
    removeEditModal(id, title){
      this.editModalsId--
      this.editModals = this.editModals.filter((item) => item.id !== id && item.title !== title)
    },
    moveEditModalTop(id, title){
      this.modalsMaxHeight++;
      this.editModals.find((t) => t.id === id && t.title === title).height = this.modalsMaxHeight;
    },
    openSubLevel(field) {
      if (this.level[field].level) {
        this.selectedField = field;
        this.$emit('subLevel', this.level[field].level, this.levelId);
      }
    },
    saveChanges() {
      this.oldLevel = JSON.parse(JSON.stringify(this.level));
    },
    abortChanges() {
      for (let key in this.oldLevel){
        this.level[key] = this.oldLevel[key];
      }
    },
    clickableLevel(field) {
      if (this.level[field].level) {
        return "cursor: pointer"
      }
    },
    delSelected(){
      for (let key in this.level){
        if (this.level[key].toDelete){
          delete this.level[key];
          this.fields = this.fields.filter((item) => item !== key)
        }
      }
      if (this.fields.length === 0){
        console.log(this.levelId)
        // this.$emit('close', this.levelId);
      }
    }
  },
  created() {
    this.level = this.passedLevel;
    for (let key in this.level){
      if (key !== "_id") {
        this.level[key].toDelete = false;
      }
    }
    this.oldLevel = JSON.parse(JSON.stringify(this.level));
  },
  watch: {
    level(newLevel){
      if (!newLevel){
        return
      }
      for (let k in newLevel){
        if (this.fields.indexOf(k) <= -1 && k !== "_id"){
          this.fields.push(k);
        }
      }
    }
  },
  computed: {
    filteredFields(){
      if (this.wordFilter !== ""){
        return this.fields.filter((t) => t.toLowerCase().includes(this.wordFilter))
      }
      let copy = JSON.parse(JSON.stringify(this.fields));
      if (this.sorting === "order") {
        return this.sortingDirection
            ? this.fields
            : copy.reverse()
      }
      if (this.sorting === "alphabet") {
        return this.sortingDirection
            ? copy.sort()
            : copy.sort().reverse()
      }
      else {
        return this.fields
      }
    }
  }
}
</script>

<style scoped>

.container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.filters {
  display: flex;
  flex-direction: row;
}
.sortingBtns {
  /*align-items: flex-start;*/
  list-style: none;
  padding-left: 0;
  margin-top: 0;
  margin-bottom: 0;
  width: auto;
}
.filterBtn {
  width: max-content;
}
.filterText {
}
.itemsDisplay {
  padding-top: 16px;
}
.itemList{
  display: flex;
  list-style: none;
  flex-direction: column;
  align-items: flex-start;
  margin-top: 8px;
  margin-bottom: 0;
  padding-left: 0;
}
.listItem {
  box-shadow: black;
}
.addBlock {
  display: flex;
  flex-direction: row;
}
.addBtn {
  margin: 8px;
  background: greenyellow;
  cursor: pointer;
  user-select: none;
}
.deleteBlock {
  display: flex;
  flex-direction: row;
}
.delBtn {
  margin: 8px;
  background: #ff2f2f;
  cursor: pointer;
  user-select: none;
}
</style>