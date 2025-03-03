<script lang="js">
import Diagram from '@2112-lab/ahu3d-diagram'
import JsonEditorVue from 'json-editor-vue'

export default {
  components: {
    JsonEditorVue,
  },
  data() {
    return {
      configs: {
        colors: {
          background: '#222222',
          foreground: '#FFFFFF',
          dotColor: '#FFFFFF',
          activeDotColor: '#F44336',
          edgeColor: '#F44336',
        },
        sizes: {
          width: 500,
          height: 500,
          rows: 8,
          columns: 8,
          dotRadius: 8,
          edgeThickness: 4,
        },
        properties: {
          flow: 'startToEnd',
          size: 0,
        },
      },
      jsonConfig: '',
      mode: 'addDots',
      diagram: null,
    }
  },
  watch: {
    // Sync `configs` to `jsonConfig`
    configs: {
      handler(newVal) {
        this.jsonConfig = JSON.stringify(newVal, null, 2)
      },
      deep: true,
    },
    // Sync `jsonConfig` to `configs`
    jsonConfig(newJson) {
      try {
        this.configs = JSON.parse(newJson)
        this.diagram = new Diagram('#diagramContainer', this.configs)
      } catch (e) {
        console.error('Invalid JSON', e)
      }
    },
  },
  mounted() {
    this.jsonConfig = JSON.stringify(this.configs, null, 2)
    this.diagram = new Diagram('#diagramContainer', this.configs)
  },
  methods: {
    switchMode(mode) {
      this.mode = mode
      this.diagram.setMode(mode)
    },
    resetDiagram() {
      this.mode = 'addDots'
      this.diagram.resetDiagram()
    },
    exportDiagram() {
      const json = this.diagram.exportData()
      const blob = new Blob([JSON.stringify(json)], { type: 'application/json' })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url
      a.download = 'diagram.json'
      document.body.appendChild(a)
      a.click()
      document.body.removeChild(a)
      URL.revokeObjectURL(url)
    },
    importDiagram(event) {
      const file = event.target.files[0]
      if (!file) return

      const reader = new FileReader()
      reader.onload = (e) => {
        try {
          const data = JSON.parse(e.target.result)
          this.diagram.importData(data)
        } catch (error) {
          console.error('Invalid JSON file', error)
        }
      }
      reader.readAsText(file)
    },
  },
}
</script>

<template>
  <div class="app">
    <div>
      <button
        class="button"
        :class="{ 'button-active': mode === 'addDots' }"
        @click="switchMode('addDots')"
      >
        Add Dots
      </button>
      <button
        class="button"
        :class="{ 'button-active': mode === 'addEdges' }"
        @click="switchMode('addEdges')"
      >
        Add Edges
      </button>
      <button class="button" @click="resetDiagram">Reset</button>
      <button class="button" @click="exportDiagram">Export</button>
      <input class="import-btn" type="file" @change="importDiagram" />
      <div id="diagramContainer"></div>
    </div>

    <div>
      <JsonEditorVue v-model="jsonConfig" v-bind="{ mode: 'tree' }" />
    </div>
  </div>
</template>

<style scoped>
.button {
  background-color: #4caf50;
  border: none;
  color: white;
  padding: 10px;
  text-align: center;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
  border-radius: 8px;
}

.button-active {
  background-color: #000;
}

.import-btn {
  background-color: #f1f1f1;
  border: 2px solid #ccc;
  color: #333;
  padding: 10px;
  text-align: center;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
  border-radius: 8px;
}

.import-btn:hover {
  background-color: #ddd;
  border-color: #888;
}
.app {
  max-width: 80%;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  gap: 50px;
}
</style>
