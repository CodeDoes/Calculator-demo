
<script setup lang="ts">
// interface Slice {
//   type: "number"|"operator";
//   toString: ()=>string;
// }
import _ from "lodash"
import { computed, reactive, ref } from "vue"

const operators = ('-+/*')
const numbers = ('1234567890')
const state = reactive({
  stack: ['999999999999999999'],
  calculation_failed: false
})
// let stack = ["0"]
// let calculation_failed = ref(false)
function isNumber(slice: string) {
  // console.log()
  return _.intersection([...numbers], [...slice]).length > 0
}
function isInteger(slice: string) {
  return _.intersection(["."], [...slice]).length > 0
}
function isFloat(slice: string) {
  return isNumber(slice) && _.intersection([...slice], ["."]).length > 0
}
function isOperator(slice: string) {

  return slice.length == 1 && _.intersection([...operators], [...slice]).length > 0
}
function handleClickOperator(char: string) {
  state.calculation_failed = false
  const top_slice = _.last(state.stack)
  if (top_slice) {
    if (isOperator(top_slice)) {
      state.stack.splice(-1, 1, char)
    } else {
      state.stack.push(char)
    }
  } else if (char === "-") {
    state.stack.push("0")
  }
}
function handleClickNumber(char: string) {
  state.calculation_failed = false
  let top_slice = _.last(state.stack)

  if (top_slice && isNumber(top_slice)) {
    console.log(1, JSON.stringify(state.stack))
    if (!isFloat(top_slice)) { top_slice = top_slice.replace(/^0+/, "") }
    state.stack.splice(-1, 1, top_slice + char)
    console.log(2, JSON.stringify(state.stack))

  } else {
    state.stack.push(char)
  }
}
function handleClickComma() {
  state.calculation_failed = false
  const top_slice = _.last(state.stack)
  console.log(top_slice)
  if (top_slice && isNumber(top_slice)) {
    console.log(top_slice.includes("."))
    if (!isFloat(top_slice)) {
      state.stack.splice(-1, 1, top_slice + ".")
    }
  } else {
    state.stack.push(".")
  }

}
function handleClearAll() {
  state.calculation_failed = false
  state.stack = ["0"]
}
function handleBackspace() {
  state.calculation_failed = false
  let top_slice = _.last(state.stack)
  if (top_slice) {
    top_slice = top_slice.substring(0, top_slice.length - 1)
    console.log(top_slice)
    state.stack.splice(-1, 1, top_slice)
    if (top_slice.length == 0) {
      state.stack.pop()
      if (state.stack.length == 0) {
        state.stack.splice(0)
        state.stack.push("0")
      }
    }
  }
}
function handleNegPos() {
  state.calculation_failed = false
  let top_slice = _.last(state.stack)
  if (top_slice && isNumber(top_slice)) {
    if (_.first(top_slice) === "-") {
      state.stack.splice(-1, 1, top_slice.substring(1))
    } else {
      state.stack.splice(-1, 1, "-" + top_slice)
    }
  }
}
function calculateLine() {
  // window.alert(JSON.stringify(state.stack))
  try {
    const val = eval(state.stack.join(""))
    state.stack = [_.toString(val)]
  } catch {
    state.calculation_failed = true
  }
}
const output = computed(() => state.stack.length > 0 ? state.stack.join("").replace(/\./g, ",") : "0")
</script>

<template>
  <div class="grid">
    <output :class="{error:state.calculation_failed}" style="max-width: 100%;">
      {{output}}_
    </output>
    <!-- <output>{{stack}}</output> -->
    <button class="backspace" v-on:click="handleBackspace">⌫</button>
    <button v-for="i in operators" v-on:click="()=>handleClickOperator(i)">{{i}}</button>
    <button v-for="i in numbers" :class="{zero:i==='0'}" v-on:click="()=>handleClickNumber(i)">{{i}}</button>
    <!-- <button class="zero" v-on:click="()=>handleClickNumber('0')">0</button> -->
    <button class="comma" v-on:click="handleClickComma">,</button>
    <button class="equals" v-on:click="calculateLine">=</button>

    <button class="negpos" v-on:click="handleNegPos">-/+</button>
    <button class="clearall" v-on:click="handleClearAll">AC</button>
  </div>
</template>

<style scoped>
* {
  font-size: inherit;
}
.grid{
}

output {
  grid-column: 1/4;
  background-color: black;
  color: white;
  padding-inline: 0.5em;
  float:right;
}

button {
  background-color: #eaeaea;
  border-radius: 0.3em;
  width: 3em;
}

.backspace {
  background-color: #f0595f;
  /* grid-area: ; */
}

.error {
  background-color: #f0595f;
}

.zero {
  /* grid-area: 6/1/7/3; */
  width: auto;
  grid-column: 1/3;
}

.clearall {
  /* grid-area: 1/4/2/5; */
  width: auto;
  height: auto;
  grid-column: 4/5;
}

.equals {
  /* grid-area: 2/4/6/5; */
  /* grid-column: 4/5; */
  /* g */
  grid-area: 3/4/7/5;
  background-color: #2e86c0;
}

.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  margin-inline: auto;
  width: min-content;
  gap: 0.5em;
  /* font-size: 2rem; */
  background-color: darkgreen;
  padding:1em;
  font-size: 5vmin;
}

button {
  vertical-align: top;
}
</style>