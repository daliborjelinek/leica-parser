<template>
  <div style="display: flex">
    <div class="input">
      <v-file-input v-model="sourceFile" label="DFX file"></v-file-input>
      <v-textarea auto-grow prepend-icon="mdi-anchor" v-model="anchorsText" label="Anchor list"></v-textarea>
    </div>
    <div style="width: 100%;">
      <v-data-table width="100%" :items-per-page="0" :items="changes">
        <template #bottom></template>
        <template #no-data>
          <v-alert
            type="info"
            class="ma-3"
            title="Instructions"
            variant="tonal">
            <p class="text-left mt-2">
            <ul>
              <li>1. insert source DFX file</li>
              <li>2. insert list of anchor names
            (every name on new line in same order of measurement). You can use automatic generator by pressing magic wand in top left corner.</li>
            <li>3. Press parse button</li>
            <li>4. Press download button</li>
            </ul>
              
            
            
          </p>
          </v-alert>
         </template>
      </v-data-table>
    </div>
  </div>
  <portal to="header-button">
    <v-btn variant="outlined" :disabled="result === ''" color="success" @click="parse">download</v-btn>
    <v-btn :disabled="sourceFile === null || anchorsText === ''" class="ml-1" color="blue" @click="parse">parse</v-btn>

  </portal>

  <portal to="sidebar">
    <div class="ma-3">
      <v-text-field v-model="generator.prefix" prepend-icon="mdi-alphabetical" label="Prefix"></v-text-field>
      <v-text-field v-model="generator.start" type="number" prepend-icon="mdi-numeric" label="start"></v-text-field>
      <v-text-field v-model="generator.end" type="number" prepend-icon="mdi-numeric" label="end"></v-text-field>
      <v-btn class="mb-2 w-100" color="primary" @click="generate">generate</v-btn>
    </div>
  </portal>
</template>

<script setup>
import { ref, computed } from 'vue'
import { store } from '/src/state/state'
import { saveAs } from 'file-saver';
const sourceFile = ref(null)
const anchorsText = ref("")
const changes = ref([])
const generator = ref({
  start: 0,
  end: 10,
  prefix: "A-"
})
const result = ref("")

function generate() {
  for (let i = generator.value.start; i <= generator.value.end; i++) {
    anchorsText.value += `${generator.value.prefix}${i}\n`
  }
  store.drawer = false
}


async function readFileAsync(fileInput) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();

    reader.onload = function (event) {
      resolve(event.target.result);
    };

    reader.onerror = function (event) {
      reject(event.target.error);
    };

    reader.readAsText(fileInput);
  });
}

function save(text, filename) {
  var blob = new Blob([text], { type: "text/plain;charset=utf-8" });
  saveAs(blob, filename);
}

async function parse() {
  changes.value = []
  const anchors = anchorsText.value.split("\n")
  let text = await readFileAsync(sourceFile.value[0])

  const hledanyVyraz = /ByLayer\r\n\S+\r\n\S+\r\n\S+\r\nAcDbText\r\n\S+\r\n(?<anchor>.+)/gmd
  let match
  let counter = 0
  while ((match = hledanyVyraz.exec(text)) && counter < anchors.length) {
    const rep = {
      from: match.indices.groups.anchor[0],
      to: match.indices.groups.anchor[1],
      old: match.groups.anchor,
      new: anchors[counter],
    }
    counter++
    hledanyVyraz.lastIndex = rep.from + rep.new.length
    text = replaceSubstring(text, rep.from, rep.to, rep.new)
    console.log(`${rep.old} was replaced by ${rep.new}`)
    changes.value.push({ index: counter, old: rep.old, new: rep.new })
  }
  result.value = text
  //save(text,sourceFile.value[0].name+"_PARSED.txt")
}

function replaceSubstring(originalString, startIndex, endIndex, replacement) {
  const before = originalString.substring(0, startIndex);
  const after = originalString.substring(endIndex + 1);
  const newString = before + replacement + after;
  return newString
}
</script>

<style>
.input {
  margin: 15px;
  min-width: 300px;

}
</style>