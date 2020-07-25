<template>
  <div class="app">
    <div class="game">
      <h3>Nonogram</h3>
      <h4>
        Lives left: {{livesLeft}}<br>
        Games won: {{gamesWon}} Games lost: {{gamesLost}}
      </h4>
      <table>
        <tr>
          <td class="rowGroupsCell"></td>
          <td :key="'gr' + group +'cl'  + columnIndex" class="columnGroupsCell"
              v-bind:class="{cellCompleted: completedColumns[columnIndex]}"
              v-for="(group, columnIndex) in cellsOccupiedInColumns">
            <span :key="'ci' + columnIndex + 'igi' + inGroupIndex" class="columnGroupsSpan"
                  v-for="(singleGroup, inGroupIndex) in group">{{singleGroup}}<br></span>
          </td>
        </tr>
        <tr :key="line + lineIndex" v-for="(line, lineIndex) in playBoard">
          <td class="rowGroupsCell" v-bind:class="{cellCompleted: completedRows[lineIndex]}">
            <span :key="'li' + lineIndex + 'gi' + groupIndex + 'gr' + group"
                  class="rowGroupsSpan" v-for="(group, groupIndex) in cellsOccupiedInRows[lineIndex]">{{group}}  </span>
          </td>
          <td :key="'li' + lineIndex+ 'ci' + cellIndex+ 'cv' + cell" @click="cellClick(lineIndex, cellIndex)"
              v-bind:class="{wrong: playBoard[lineIndex][cellIndex] === 1, good: playBoard[lineIndex][cellIndex] === 2, bottomFrameWide: lineIndex == 4, rightFrameWide: cellIndex == 4}"
              v-for="(cell, cellIndex) in line">
          </td>
        </tr>
      </table>
      <div class="lostGame" v-show="livesLeft === 0"><H2>You lost this game...</H2>
        <p>
          <button @click.prevent="initGame" type="button">Restart</button>
        </p>
      </div>
      <div class="lostGame" v-show="gameWon"><H2>You won this game...</H2>
        <p>
          <button @click.prevent="initGame" type="button">Restart</button>
        </p>
      </div>
    </div>
  </div>
</template>

<script>


  export default {
    name: 'app',
    data: function () {
      return {
        playBoard: [],
        resultBoard: [[0, 2, 0, 0, 0, 0, 0, 0, 2, 0],
          [0, 2, 0, 0, 2, 2, 0, 0, 2, 2],
          [2, 0, 0, 2, 2, 2, 2, 0, 2, 0],
          [2, 2, 2, 2, 2, 2, 2, 2, 2, 2],
          [2, 2, 0, 0, 2, 2, 0, 0, 2, 2],
          [2, 2, 0, 2, 2, 2, 2, 0, 2, 0],
          [2, 2, 0, 0, 2, 2, 2, 0, 2, 2],
          [2, 2, 0, 0, 2, 2, 0, 0, 2, 2],
          [2, 2, 0, 2, 0, 2, 2, 0, 2, 2],
          [2, 2, 2, 0, 2, 2, 2, 2, 2, 2],
        ],
        livesLeft: 3,
        gameWon: false,
        resultBoardUsedCells: 0,
        playBoardUsedCells: 0,
        completedRows: [false, false, false, false, false, false, false, false, false, false],
        completedColumns: [false, false, false, false, false, false, false, false, false, false],
        cellsOccupiedInRows: "",
        cellsOccupiedInColumns: "",
        gamesWon: 0,
        gamesLost: 0
      }
    },
    created: function () {
      this.initGame();
    },

    methods: {
      cellClick: function (row, column) {
        if (this.playBoard[row][column] == 0 && this.resultBoard[row][column] == 0 && this.livesLeft > 0 && this.gameWon == false) {
          this.playBoard[row][column] = 1;
          this.livesLeft -= 1;
        } else if (this.playBoard[row][column] == 0 && this.resultBoard[row][column] == 2 && this.livesLeft > 0 && this.gameWon == false) {
          this.playBoard[row][column] = 2;
          this.playBoardUsedCells++;
        }
        this.completedRows[row] = this.checkRowCompleted(row);
        this.completedColumns[column] = this.checkColumnCompleted(column);
        this.$forceUpdate();
        if (this.playBoardUsedCells === this.resultBoardUsedCells) {
          this.gameWon = true;
          this.gamesWon++;
        }

      },
      initGame: function () {
        this.playBoard.length = 0;
        for (let i = 0; i < 10; i++) {
          let arrayLine = [];
          for (let j = 0; j < 10; j++) {
            arrayLine.push(0);
          }
          this.playBoard.push(arrayLine);
        }
        if (this.livesLeft === 0) this.gamesLost++;
        this.livesLeft = 3;
        this.gameWon = false;
        this.resultBoardUsedCells = 0;
        this.playBoardUsedCells = 0;
        this.completedRows = [false, false, false, false, false, false, false, false, false, false];
        this.completedColumns = [false, false, false, false, false, false, false, false, false, false];
        let ref = this;
        this.resultBoard.forEach(function (row, rowIndex) {
          row.forEach(function (value, valueIndex) {
            let x = Math.random();
            if (x >= 0.5) ref.resultBoard[rowIndex][valueIndex] = 2;
            else ref.resultBoard[rowIndex][valueIndex] = 0;
          })
        });
        this.cellsOccupiedInColumns = this.findCellsOccupiedInColumns();
        this.cellsOccupiedInRows = this.findCellsOccupiedInRows();
        this.resultBoard.forEach(function (row) {
          row.forEach(function (cell) {
            if (cell === 2) ref.resultBoardUsedCells++;
          })
        })
      },
      checkRowCompleted(row) {
        let result = true;
        let ref = this;
        this.playBoard[row].forEach(function (value, index) {
          if (ref.resultBoard[row][index] === 2 && value != ref.resultBoard[row][index]) result = false;
        })
        return result;
      },
      checkColumnCompleted(column) {
        let result = true;
        //let ref = this;

        for (let i = 0; i < this.playBoard.length; i++) {
          if (this.resultBoard[i][column] === 2 && this.playBoard[i][column] != this.resultBoard[i][column]) result = false;
        }
        return result
      },
      findCellsOccupiedInRows() {
        let result = [];
        this.resultBoard.forEach(function (row) {
          let groupsInRow = [];
          let numberOfCellsInGroup = 0;
          for (let i = 0; i < row.length; i++) {
            if (row[i] === 2) {
              numberOfCellsInGroup++;
              if (i == row.length - 1 && numberOfCellsInGroup > 0) groupsInRow.push(numberOfCellsInGroup);
            } else if (numberOfCellsInGroup > 0) {
              groupsInRow.push(numberOfCellsInGroup);
              numberOfCellsInGroup = 0;
            }
          }
          result.push(groupsInRow);
        })
        return result
      },
      findCellsOccupiedInColumns() {
        let result = [];
        let rowLength = this.resultBoard[0].length;
        let numberOfRows = this.resultBoard.length;
        for (let i = 0; i < rowLength; i++) {
          let groupsInColumn = [];
          let numberOfCellsInGroup = 0;
          for (let j = 0; j < numberOfRows; j++) {

            if (this.resultBoard[j][i] === 2) {
              numberOfCellsInGroup++;
              if (j == numberOfRows - 1 && numberOfCellsInGroup > 0) groupsInColumn.push(numberOfCellsInGroup);
            } else if (numberOfCellsInGroup > 0) {
              groupsInColumn.push(numberOfCellsInGroup);
              numberOfCellsInGroup = 0;
            }
          }
          result.push(groupsInColumn);
        }
        return result
      },
    },
  }
</script>

<style>
  .app {
    position: absolute;
    margin: 0px;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  .game {
    z-index: 2;
    position: absolute;
    margin: 0px;
    width: 50%;
    height: 100%;
    left: 25%;
    text-align: center;
    overflow: hidden;
  }

  table, td, tr {
    border: 1px solid black;
    border-collapse: collapse;
    background-color: lightgray;
  }

  table {
    width: 100%;
    height: 80%;
  }

  .wrong {
    background-color: darkred;
  }

  .good {
    background-color: rgba(0, 0, 0, 0.7);
  }

  .lostGame {
    position: relative;
    top: -300px;
    left: 0px;
    width: 200px;
    border: 2px solid darkblue;
    padding: 20px;
    margin: auto;
    align: center;
    text-align: center;
    background-color: lightblue;
    color: darkred;
  }

  .rowGroupsSpan {
    width: 10px;
    text-align: right
  }

  .rowGroupsCell {
    text-align: right;
    width: 75px;
    min-width: 75px;
    padding-right: 5px;
    background-color: lightgray
  }

  .columnGroupsSpan {
    width: 10px;
    text-align: center
  }

  .columnGroupsCell {
    text-align: center;
    vertical-align: bottom;
    width: 50px;
    max-width: 50px;
    background-color: lightgray
  }

  .cellCompleted {
    background-color: rgba(0, 0, 0, 0.7);;
    animation-name: szerokosc;
    animation-duration: 5s;
  }

  .bottomFrameWide {
    border-bottom: 3px solid black;
  }

  .rightFrameWide {
    border-right: 3px solid black;
  }

  @keyframes completed {
    0% {
      background-color: blue
    }
    100% {
      background-color: darkred
    }
  }


  @-webkit-keyframes completed {
    0% {
      background-color: blue
    }
    100% {
      background-color: darkred
    }
  }

</style>
