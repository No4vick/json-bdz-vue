<template>
  <Transition name="modal">
<!--    <div v-if="show" class="modal-mask">-->
      <div class="modal-wrapper" v-bind:id="id" v-if="props.show"
           v-bind:style="{top : updateModalPos.y - 30 + 'px', left : updateModalPos.x - 50 + 'px', zIndex : 500 + height}"
           @mousedown="$emit('placeOnTop', id, title)"
      >
        <div class="modal-container">
          <a @mousedown="state.onMove=true; updatePos()" @mouseup="state.onMove=false; updatePos()" style="user-select: none">--------</a>
          <button
              class="modal-close-button"
              @click="$emit('close', id, title)"
          >X</button>
          <div class="modal-header">
            <slot name="header">default header</slot>
          </div>
          <div class="modal-body">
            <div class="modal-text-fields" v-for="(value, key) in state._prop" :key="key">
              <div v-if="state.editedKey !== key && key !== 'toDelete'">
                <div v-if="isObject(value) && typeof(value) !== 'string'">{{key}} : <br><pre>{{value}}</pre><br></div>
                <div v-else>{{key}} : {{value}}</div>
                <button @click="setEditable(key)">РЕД</button>
                <button @click="delField(key)">УДАЛ</button>
              </div>
              <div v-else-if="key !== 'toDelete'">
                {{key}} :
                <input v-if="!state.newValueJson" type="text" v-model="state.newValue" @keydown.enter="saveEdit()">
                <textarea v-else v-model="state.newValue" @keydown.enter="saveEdit()"></textarea>
                <button @click="saveEdit()">СОХР</button>
              </div>
            </div>
            <div class="modal-new-field" v-if="state.addingNewField">
              <input class="modal-text-field" type="text" placeholder="Поле" v-model="state.newKey"
                     @keydown.enter="saveNewField()" @keydown.esc="abortNewField">
              <input class="modal-text-field" type="text" placeholder="Значение" v-model="state.newValue"
                     @keydown.enter="saveNewField()" @keydown.esc="abortNewField">
              <button @click="saveNewField()">СОХР</button>
            </div>
            <div class="modal-fields-buttons">
              <div class="editBtns">
                <div class="addBlock">
                  <div class="addBtn" @click="addNewField()">+</div>
                  <a>Добавить</a>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <slot name="footer">
<!--              X = {{ state.posX + 'px' }}, Y = {{ state.posY }}-->
              <button
                  class="modal-default-button"
                  @click="$emit('save', id, title, {field: state.key, value: state.value})"
              >Сохранить</button>
            </slot>
          </div>
        </div>
      </div>
<!--    </div>-->
  </Transition>
</template>

<script setup>
import {useMouse} from "@/mouse";
import {reactive, defineProps, computed, onMounted} from "vue";


const { x, y } = useMouse()
const state = reactive({
  posX: x.value,
  posY: y.value,
  onMove: false,
  _prop: null,
  editedKey: null,
  newValue: "",
  newValueJson: false,
  newKey: null,
  addingNewField: false
})
const props = defineProps({show: Boolean, id: Number, title: String, height: Number, prop: Object})
// const emits = defineEmits(['close', 'save'])

function getRndInteger(min, max) {
  return Math.floor(Math.random() * (max - min + 1) ) + min;
}

onMounted(() => {
  state.posY = 200 + getRndInteger(-100, 100);
  state.posX = 300 + getRndInteger(-100, 100);
  state._prop = props.prop;
  console.log(`KEK!${state.posX} ${state.posY}`);
})

function updatePos(){
  state.posX = x.value;
  state.posY = y.value;
}

function setEditable(key){
  state.editedKey = key;
  state.newValue = state._prop[key];
  state.newValueJson = this.isObject(state.newValue);
  if (state.newValueJson) {
    state.newValue = JSON.stringify(state.newValue);
  }
}

function saveEdit(){
  if (state.newValueJson) {
    state.newValue = JSON.parse(state.newValue);
  }
  state._prop[state.editedKey] = state.newValue;
  state.editedKey = null;
}

function addNewField(){
  state.addingNewField = true;
}

function abortNewField(){
  state.newKey = null;
  state.newValue = "";
  state.addingNewField = false;
}

function saveNewField(){
  if (!state.newKey || !state.newValue){
    state.addingNewField = false;
    return
  }
  if (!state._prop){
    state._prop = {};
  }
  state._prop[state.newKey] = state.newValue;
  state.addingNewField = false;
  state.newKey = null;
  state.newValue = "";
}

function delField(key){
  // state._prop = state._prop.filter((field) => field)
  delete state._prop[key]
}

function isObject(obj) {
  return typeof (obj) === "object";
}

const updateModalPos = computed(() => {
  let params = {x: state.posX, y: state.posY}
  if (state.onMove) {
    params.x = x.value;
    params.y = y.value;
  }
  return params;
})

</script>

<style scoped>

.modal-mask {
  position: fixed;
  z-index: 500;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  /*background-color: rgba(0, 0, 0, 0.5);*/
  display: table;
  transition: opacity 0.3s ease;
}

.modal-wrapper {
  position: absolute;
  /*z-index: 500;*/
  /*top: 0;*/
  /*left: 0;*/
  /*width: 100%;*/
  /*height: 100%;*/
  transition: opacity 0.3s ease;
  display: table-cell;
  vertical-align: middle;
}

.modal-container {
  position: absolute;
  width: 300px;
  margin: 0 auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-body {
  margin: 20px 0;
}

.modal-default-button {
  float: right;
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

/*
 * The following styles are auto-applied to elements with
 * transition="modal" when their visibility is toggled
 * by Vue.js.
 *
 * You can easily play with the modal transition by editing
 * these styles.
 */

.modal-enter-from {
  opacity: 0;
}

.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-container,
.modal-leave-to .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}

</style>