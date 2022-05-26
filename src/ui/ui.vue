<script setup lang="ts">
import { onMounted, ref, watch, reactive, nextTick } from "vue";
import { handleEvent, dispatch } from "./uiMessageHandler";
import "figma-plugin-ds/dist/figma-plugin-ds.css";
import { attach } from "@frsource/autoresize-textarea";


let textNodeArray = ref([] as TextNode[]);

let state = reactive({
  isSelected: false,
});

onMounted(() => {
  handleEvent("selectionchange", (data) => {
    console.log("ui detect change");
    // console.log(data.textNodeArray);

    state.isSelected = data.select;

    textNodeArray.value = data.textNodeArray;
    // setTimeout(() => {
    //   document.getElementById("0")?.focus()
    // }, 10)

    nextTick(() => {
      for (const textarea of document.querySelectorAll(".textarea")) {
        attach(textarea);
        console.log("attach");
      }
    });
  });
});

function handleChange(event: Event, textNode: TextNode) {
  console.log("onchange");

  dispatch("change-text", { id: textNode.id, characters: textNode.characters });
}

function toNextInput(event: KeyboardEvent, index: number) {
  console.log("enter");

  if (event.isComposing) {
    return;
  }
  // document.getElementById(`${index + 1}`)?.focus()
}

function handleFocus(event: FocusEvent, item: TextNode) {
  dispatch("input-focus", { id: item.id });
}

function handleBlur(event: FocusEvent, item: TextNode) {
  dispatch("input-blur", { id: item.id });
}
</script>

<template>
  <div
    class="img-container"
    v-if="state.isSelected == false"
    style="width: 100%; display: flex; flex-direction: column; align-items: center"
  >
    <img
      src="./default.png"
      alt="seletc any layer to go"
      style="width: 117px; height: 116px; margin: auto"
    />
  </div>

  <div id="ui" v-else>
    <div
      class="info"
      :style="{
        'background-color': textNodeArray.length == 0 ? 'F8EAE5' : 'E4F2FE',
      }"
    >
      <div class="icon icon--search"></div>
      <div class="type">
        {{
          `${textNodeArray.length} text ${
            textNodeArray.length == 1 ? "layer" : "layers"
          } found in your selection`
        }}
      </div>
    </div>

    <div class="img-container" v-if="textNodeArray.length == 0">
      <img src="./404.png" alt="no text layer found" style="width: 145px; height: 58px" />
    </div>

    <div id="input-container" v-if="textNodeArray.length != 0">
      <div v-for="(item, index) in textNodeArray" :key="item.id" class>
        <div class="head-container type">
          <div class="icon t">𝚃</div>
          <div class id="input-head">
            {{ item.autoRename ? "Auto Rename Layer" : item.name }}
          </div>
        </div>

        <textarea
          type="text"
          v-model="item.characters"
          @input="handleChange($event, item)"
          @keypress.enter="toNextInput($event, index)"
          :id="index.toString()"
          rows="1"
          class="textarea"
          @focus="handleFocus($event, item)"
          @blur="handleBlur($event, item)"
        />
      </div>
    </div>

    <div class="tips type" v-if="textNodeArray.length != 0">
      <div>
        <strong>Tips:</strong>
      </div>
      <div><strong>⇥ tab</strong>: switch to the next one</div>
      <div><strong>⇧ shift + ⇥ tab</strong>: switch to the previous one</div>
    </div>
  </div>
</template>

<style>
#app {
  background-color: var(--figma-color-bg);
}

#ui {
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

#input-head {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
}

.head-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: flex-start;
}

.icon .t {
  transform: scale(1.2, 1);
}

.icon {
  height: 24px;
  width: 24px;
  font-size: 16px;
}

#input-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.textarea {
  /* resize: vertical; */
  min-height: 12px !important;
}

.tips {
  /* margin-top: 16px; */
}

.head-container {
  color: #b2b2b2;
  display: flex;
  flex-direction: row;
  align-items: center;
}

.info {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 8px;
  gap: 4px;
  border-radius: 4px;
}

.img-container {
  align-self: center;
  margin: 64px 0;
}
</style>
