<template lang="pug">
div
  #controls
    label size: 
    input#size(type="number" v-model="size")
  #game
    //- -let len = 10;for(let i=0;i<len;i++)
    .row(v-for="(row, i) in gameMatrix" id=("row"+i))
      .tile(v-for="(tile, j) in row" @contextmenu="mark($event, tile)" @click="tilePress(tile)" id=("tile"+i+""+j) :style="{height:(tileWidthVMin+'vmin'), lineHeight:(tileWidthVMin+'vmin'), width:(tileWidthVMin+'vmin'), backgroundColor:(tile.hasBomb && showBombs)?'red':(tile.number==0 && tile.showN?'white':(tile.showN?'rgb(255, 204, 110)':(tile.marked?'rgb(145, 255, 0)':'')))}") {{tile.showN?tile.number:""}}
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "App",
  data: () => ({
    size: 10 as number,
    gameMatrix: [] as Array<Array<Tile>>,
    difficulty: 1,
    showBombs: false,
  }),
  computed: {
    tileWidthVMin(): number {
      return (10 / this.size) * 7.5;
    },
  },
  methods: {
    mark(ev:Event, tile:Tile){
      ev.preventDefault();
      tile.marked = !tile.marked;
    },
    tilePress(tile: Tile, ignore?: Tile[]): void {
      ignore = ignore || [];
      if(ignore.length==0 && tile.showN)return;
      if (tile.hasBomb) {
        this.showBombs = true;
        return alert("You Lose!");
      }
      tile.showN = true;
      for (let k = tile.x - 1; k <= tile.x + 1; k++) {
        for (let l = tile.y - 1; l <= tile.y + 1; l++) {
          if (
            !(k == tile.x && l == tile.y) &&
            !(k < 0 || l < 0 || k >= this.size || l >= this.size)
          ) {
            if (!this.gameMatrix[k][l].hasBomb)
              this.gameMatrix[k][l].showN = true;
            let isZero = (ii: number, jj: number) =>
              this.gameMatrix[ii][jj].number == 0 &&
              !this.gameMatrix[ii][jj].hasBomb
                ? false
                : true;
            if (!isZero(k, l) && !ignore?.includes(this.gameMatrix[k][l])) {
              ignore.push(this.gameMatrix[k][l]);
              this.tilePress(this.gameMatrix[k][l], ignore);
            }
          }
        }
      }
    },
    generate() {
      this.showBombs = false;
      this.gameMatrix = [];
      for (let i = 0; i < this.size; i++) {
        this.gameMatrix.push([]);
        for (let j = 0; j < this.size; j++) {
          this.gameMatrix[i].push(new Tile(i, j));
        }
      }
      for (let i = 0; i < (this.size**2/5) * this.difficulty; i++) {
        this.gameMatrix[Math.floor(Math.random() * (this.size - 1))][
          Math.floor(Math.random() * (this.size - 1))
        ].hasBomb = true;
      }
      for (let i = 0; i < this.size; i++) {
        for (let j = 0; j < this.size; j++) {
          let hasbomb = (ii: number, jj: number) =>
            ii < 0 || jj < 0 || ii >= this.size || jj >= this.size
              ? 0
              : this.gameMatrix[ii][jj]?.hasBomb
              ? 1
              : 0;
          if (hasbomb(i, j)) {
            continue;
          }
          this.gameMatrix[i][j].number =
            hasbomb(i - 1, j) +
            hasbomb(i + 1, j) +
            hasbomb(i, j - 1) +
            hasbomb(i, j + 1) +
            hasbomb(i - 1, j - 1) +
            hasbomb(i + 1, j + 1) +
            hasbomb(i + 1, j - 1) +
            hasbomb(i - 1, j + 1);
        }
      }
    },
  },
  watch: {
    size: {
      handler: function() {
        this.generate();
      },
      immediate: true,
    },
  },
});
class Tile {
  hasBomb: boolean;
  number: number;
  showN: boolean;
  marked: boolean;
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.hasBomb = false;
    this.number = 0;
    this.showN = false;
    this.marked = false;
    this.x = x;
    this.y = y;
  }
}
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Roboto&display=swap");
#app {
  font-family: "Roboto", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#controls,
#game {
  margin: 10px;
  padding: 5px;
}
#game {
  display: flex;
  flex-direction: column;
  flex-wrap: nowrap;
  justify-content: space-around;
  width: 90vmin;
  height: 90vmin;
  margin: auto;
  box-sizing: border-box;
}
.row {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-around;
  margin: 0;
}
.tile {
  border-radius: 3px;
  background-color: rgb(255, 166, 0);
  margin: 0;
}
</style>
