<template>
  <div id="bingo-helper">
    <div id="curtains"></div>

    <div id="container">
      <h1 id="heading">ACECRAFT Bingo Helper</h1>
      
      <div id="card-outer">
        <div id="card-border">
          <div id="card-inner">
            <div id="note">NOTE: Click a cell to mark it as Unknown, Win, or Lose</div>
      
            <div id="grid">
              <div
                v-for="(cell, index) in cells"
                :key="index"
                class="cell"
                :class="[cell.state, { hint: bestMoves.includes(index) }]"
                @click="cycleState(index)"
              >
                {{ cellDisplay(cell, index) }}
              </div>
            </div>

            <div id="hint">HINT: {{ suggestion }}</div>
            <button id="button" @click="resetGrid">Reset</button>
          </div>
        </div>
      </div>

      <div id="copyright">Made by <a href="https://github.com/bogdancondorachi/acecraft-helpers" target="_blank">Bogdan Condorachi</a></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const gridSize = 4;
const states = ["unknown", "win", "lose"];

// cells array with reactive state
const cells = ref(
  Array.from({ length: gridSize * gridSize }, (_, i) => ({
    state: "unknown",
    index: i,
  }))
);

function cycleState(i) {
  const cell = cells.value[i];
  const next =
    states[(states.indexOf(cell.state) + 1) % states.length];
  cell.state = next;
}

// Reset grid
function resetGrid() {
  cells.value.forEach((cell, i) => {
    cell.state = "unknown";
    cell.index = i;
  });
}

// Define winning lines
function getWinningLines() {
  const lines = [];
  for (let r = 0; r < gridSize; r++) {
    lines.push([...Array(gridSize).keys()].map(c => r * gridSize + c));
  }
  for (let c = 0; c < gridSize; c++) {
    lines.push([...Array(gridSize).keys()].map(r => r * gridSize + c));
  }
  lines.push([...Array(gridSize).keys()].map(i => i * gridSize + i));
  lines.push([...Array(gridSize).keys()].map(i => i * gridSize + (gridSize - 1 - i)));
  return lines;
}

// Best moves calculation
const bestMoves = computed(() => {
  const lines = getWinningLines();

  const hasWinningLine = lines.some(line =>
    line.every(i => cells.value[i].state === "win")
  );
  if (hasWinningLine) return [];

  const score = new Array(gridSize * gridSize).fill(0);

  for (let line of lines) {
    const statesLine = line.map(i => cells.value[i].state);

    if (statesLine.includes("lose")) continue;
    if (statesLine.every(s => s === "win")) continue;

    const unknownIndices = line.filter(i => cells.value[i].state === "unknown");
    const winCount = statesLine.filter(s => s === "win").length;

    for (let i of unknownIndices) {
      score[i] += 1 + winCount * 10;
    }
  }

  const maxScore = Math.max(...score);
  if (maxScore === 0) return [];

  return score
    .map((s, i) => (s === maxScore ? i : -1))
    .filter(i => i !== -1);
});

// Suggestion text
const suggestion = computed(() => {
  const hasWinningLine = getWinningLines().some(line =>
    line.every(i => cells.value[i].state === "win")
  );
  if (hasWinningLine) return "🎉 You have a winning line!";

  if (bestMoves.value.length === 0) return "No possible winning line left.";

  return "Try cells " + bestMoves.value.map(i => i + 1).join(", ");
});

// Cell display helper
function cellDisplay(cell, index) {
  if (cell.state === "unknown") return index + 1;
  if (cell.state === "win") return "✔";
  if (cell.state === "lose") return "✖";
  return index + 1;
}
</script>

<style scoped>
#bingo-helper {
  display: grid;
  align-items: center;
  justify-content: center;
  height: 100svh;
  padding-inline: 12px;
  background-image: url(../assets/images/background.jpg);
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}
#curtains {
  position: absolute;
  top: 0;
  left: 50%;
  width: 100%;
  height: 100%;
  background-image: url(../assets/images/curtains.png);
  background-position: 50% 0;
  background-repeat: no-repeat;
  background-size: cover;
  transform: translateX(-50%);
}
#container {
  z-index: 1;
}
#heading {
  font-size: 34px;
  line-height: 1.4;
  color: #FFCE7A;
  align-content: end;
  margin-block: 80px 30px;
  filter: drop-shadow(0 0 2px black) drop-shadow(0 0 2px black);
  background: linear-gradient(180deg, rgb(255 221 114) 40%, rgb(255, 255, 209) 70%);
  background-clip: text;
  -webkit-text-fill-color: transparent;
}
#card-outer {
  background: rgb(123, 96, 53);
  padding: 6px;
  border: 2px solid #190900;
  border-radius: 15px;
}
#card-border {
  background: rgb(123, 96, 53);
  padding: 3px;
  border: 2px dashed #583a1e;
  border-radius: 15px;
}
#card-inner {
  padding: 30px;
  background-color: #d1bda2;
  box-shadow: inset 1px 5px 30px 15px #caa98a;
  border: 2px solid #040201;
  border-radius: 12px;
}
#grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0,1fr));
  gap: 14px;
  margin: 20px auto;
}
.cell {
  grid-column: span 1 / span 4;
  color: #F9E9D5;
  aspect-ratio: 1 / 1;
  border: 3px solid #b0ad9e;
  outline: 1px solid #50311d;
  border-radius: 2px;
  font-size: 22px;
  cursor: pointer;
  user-select: none;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.3s ease;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 6px 12px -2px, rgba(0, 0, 0, 0.3) 0px 3px 7px -3px;
}
.cell:hover {
  transform: scale(1.02);
}
.unknown {
  background-color: #574e3d;
}
.win {
  background-color: #505445;
  border-color: #8cae58;
}
.lose {
  background-color: #714345;
  border-color: #f7796b;
}
.hint {
  background-color: #846342;
  border-color: #ffcd5a;
}
#button {
  font-family: inherit;
  color: #E9CEA7;
  width: 100px;
  height: 35px;
  padding: 9px;
  margin-top: 20px;
  background-color: transparent;
  background-image: url(../assets/images/button.png);
  background-repeat: no-repeat;
  background-size: cover;
  border: none;
  outline: none;
  cursor: pointer;
}
#copyright {
  font-size: 11px;
  color: #F9E9D5;
  margin-block: 30px;
}
</style>
