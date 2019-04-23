<template>
  <div>
    <div class="gameStatus" :class="gameStatusColor">
        {{ gameStatusMessage }}
    </div>

    <table class="grid">
    <tr>
      <cell name="1"></cell>
      <cell name="2"></cell>
      <cell name="3"></cell>
    </tr>
    <tr>
      <cell name="4"></cell>
      <cell name="5"></cell>
      <cell name="6"></cell>
    </tr>
    <tr>
      <cell name="7"></cell>
      <cell name="8"></cell>
      <cell name="9"></cell>
    </tr>
    </table>
  </div>
</template>

<script>
  import Cell from './Cell.vue'

  export default {
    components: { Cell },

    data () {
      return {
        activePlayer: 'X', // either X or O
        gameStatus: 'turn', // either turn, win or draw
        gameStatusMessage: `X's turn`,
        gameStatusColor: 'statusTurn', // either statusTurn, statusWin, or statusDraw
        moves: 0,
        cells: {
          1: '', 2: '', 3: '',
          4: '', 5: '', 6: '',
          7: '', 8: '', 9: '',
        },
        // contains all (8) possible win conditions
        winConditions: [
          [1,2,3], [4,5,6], [7,8,9], // rows
          [1,4,7], [2,5,8], [3,6,9], // columns
          [1,5,9], [3,5,7], // diagonals
        ],
      }
    },
    
    computed: {
      // helper property to get the non-active player
      nonActivePlayer () {
        if (this.activePlayer === 'O') {
          return 'X'
        }
        return 'O'
      }
    },

    watch: {
      // watches for change in the value of gameStatus and changes the status 
      // message and color accordingly
      gameStatus () {
        if (this.gameStatus === 'win') {
          this.gameStatusColor = 'statusWin'
          return
        }
        else if (this.gameStatus === 'draw') {
          this.gameStatusColor = 'statusDraw'
          this.gameStatusMessage = 'Draw !'
          return
        }

        this.gameStatusMessage = `${this.activePlayer}'s turn`
      }
    },

    methods: {
      changePlayer () {
        this.activePlayer = this.nonActivePlayer
      },

      checkForWin () {
        for (let i = 0; i < this.winConditions.length; i++) {
          // gets a single win condition wc from the win conditions array
          let wc = this.winConditions[i]
          let cells = this.cells
          // compare 3 cellvalues based on the cells in the condition
          if (this.areEqual(cells[wc[0]], cells[wc[1]], cells[wc[2]])) {
            return true
          }
        }
        return false
      },

      gameIsWon () {
        // fires a win event for the App component to change the score
        Event.$emit('win', this.activePlayer)
        this.gameStatusMessage = `${this.activePlayer} Wins!`
        Event.$emit('freeze')
        return 'win'
      },

      changeGameStatus () {
        if (this.checkForWin()) {
          return this.gameIsWon()
        }
        else if (this.moves === 9) {
          return 'draw'
        }
        return 'turn'
      },

      areEqual () {
        let len = arguments.length

        for (let i = 1; i < len; i++) {
          if (arguments[i] === '' || arguments[i] !== arguments[i-1]) {
            return false
          }
        }
        return true
      },
    },
    
    created () {
      // listens for a strike made by the user on a cell
      // it is called by the Cell component
      Event.$on('strike', (cellNumber) => {
        // sets either X or O in the clicked cell of the cells array
        this.cells[cellNumber] = this.activePlayer
        this.moves++
        this.gameStatus = this.changeGameStatus()
        this.changePlayer()

        if (this.gameStatus === 'turn') {
          this.gameStatusMessage = `${this.activePlayer}'s turn`
        }
      })

      // listens for a restart button press
      // the data of the component is reinitialized
      // it is called by the App component
      Event.$on('gridReset', () => {
        Object.assign(this.$data, this.$options.data())
      })
    }
  }
</script>

<style>
  .grid {
    background-color: #34495ed4;
    color: #fff;
    width: 100%;
    border-collapse: collapse;
  }

  .gameStatus {
    margin: 0px;
    padding: 15px;
    border-top-left-radius: 16px;
    border-top-right-radius: 16px;
    background-color: #ffcc00;
    color: #fff;    
    font-size: 1.4em;
    font-weight: bold;
  }

  .statusTurn {
    background-color: #ffcc00;
  }

  .statusWin {
    background-color: #2ab163;
  }

  .statusDraw {
    background-color: #9b59b6;
  }
</style>