<template>
  <div id="app">
    <div class="controller">
      <div>
        <button @click="parseNestedText()">parse</button>
        <button @click="clearModel()">clear</button>
      </div>
      <h4>Nest2JSON Convert</h4>
      <div>
        <button @click="copyResult()">copy</button>
      </div>
    </div>
    <div class="wrap">
      <textarea class="resole" v-model="inputModel"></textarea>
      <div class="jsonEditor" ref="jsonEditor"></div>
    </div>
  </div>
</template>

<script>
import JSONEditor from "jsoneditor";

export default {
  name: "App",
  components: {},
  data() {
    return {
      inputModel: "",
      outputResult: "",
      jsonEditor: null,
    };
  },
  mounted() {
    const container = this.$refs.jsonEditor;
    this.jsonEditor = new JSONEditor(container, {
      mode: "tree", // 初始視圖模式：可以是 'tree'、'view' 或 'code'
    });
  },
  methods: {
    clearModel() {
      this.inputModel = "";
    },

    copyResult() {
      navigator.clipboard.writeText(this.outputResult).then(() => {
        alert("已複製 JSON 數據到剪貼板！");
      });
    },

    parseNestedText() {
      let input = this.inputModel;
      const lines = input.split("\n").filter((line) => line.trim() !== "");

      let minLevel = Infinity;
      lines.forEach((line) => {
        let match = line.match(/^(\d+\.)+/);
        if (match) {
          let level = match[0].split(".").length - 1;
          if (level < minLevel) minLevel = level;
        }
      });

      function parseLines(startIndex, level) {
        let result = [];
        let linesProcessed = 0;

        for (let i = startIndex; i < lines.length; i++) {
          let line = lines[i];
          let regex = /^(\d+\.)+/;
          let match = line.match(regex);
          if (!match) continue;

          let lineLevel = match[0].split(".").length - 1;
          if (lineLevel < level) break; // 完成當前層級的解析

          let title = line.replace(regex, "").trim();
          let node = { title: title };
          linesProcessed++; // 計算處理的行數

          // 檢查下一行是否為子層級
          if (i + 1 < lines.length) {
            let nextLine = lines[i + 1];
            let nextLineMatch = nextLine.match(regex);
            let nextLineLevel = nextLineMatch
              ? nextLineMatch[0].split(".").length - 1
              : 0;

            if (nextLineLevel > lineLevel) {
              let { result: subResult, linesProcessed: subLinesProcessed } =
                parseLines(i + 1, nextLineLevel);
              node.content = subResult;
              i += subLinesProcessed; // 跳過子層級行數
              linesProcessed += subLinesProcessed;
            }
          }

          result.push(node);
        }

        return { result, linesProcessed };
      }

      // 將沒有子層的對象轉換為字符串
      function flattenNodes(nodes) {
        return nodes.map((node) => {
          if (node.content && node.content.length > 0) {
            node.content = flattenNodes(node.content);
          } else {
            return node.title;
          }
          return node;
        });
      }

      let { result: output } = parseLines(0, minLevel);
      let formatOutput = flattenNodes(output);
      this.outputResult = JSON.stringify(formatOutput);
      this.jsonEditor.set(formatOutput);
    },
  },
};
</script>

<style>
body {
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100vw;
  height: 100vh;
}
.controller {
  max-width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: end;
  padding: 1rem;
}
.wrap {
  max-width: 100%;
  height: 80%;
  background-color: #ddd;
  display: flex;
  gap: 1.5rem;
}
.resole {
  flex: 1;
}
.jsonEditor {
  width: 100%;
  height: 100%;
  flex: 1;
  background-color: lightblue;
}
</style>
