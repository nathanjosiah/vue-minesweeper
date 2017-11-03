<template>
	<div class="game">
		Mines Remaining: {{minesRemaining}}<br />
		You won: {{won}}
		<div class="board">
			<template v-for="row in cells">
				<div class="cell" v-for="cell in row" :class="{mine:cell.mine}">
					<div v-if="gameover && cell.mine" class="mine"></div>
					<div v-else-if="(gameover || cell.touched) && cell.n > 0" :class="getCellClass(cell)"></div>
					<button v-else-if="!gameover && !cell.touched" @click="click(cell)" @contextmenu.prevent="flag(cell)" :class="{flagged:cell.flagged}"></button>
				</div>
			</template>
		</div>
	</div>
</template>

<script>
	// Must match the css grid
	const numRows = 10;
	const numColumns = 6;
	export default {
		data() {
			return {
				gameover: false,
				won: false,
				mineCount: 0,
				flaggedMines: 0,
				cells: []
			}
		},
		computed: {
			minesRemaining() {
				return this.mineCount - this.flaggedMines;
			}
		},
		methods: {
			getCellClass(cell) {
				let classname = '';
				if(cell.flagged) {
					classname += ' flagged';
				}
				if(!cell.mine) {
					classname += ' number-' + cell.n;
				}
				return classname;
			},
			click(cell) {
				if(cell.flagged) {
					return;
				}

				cell.touched = true;
				if(cell.mine) {
					this.gameover = true;
					this.won = false;
					alert('Game over!');
				}
				else if(cell.n === 0) {
					for(let connected_cell of this.getConnectedBlankCells(cell.row,cell.column,this.cells)) {
						if(!connected_cell.flagged) {
							connected_cell.touched = true;
						}
					}
				}

				let won = true;
				for(let row of this.cells) {
					for(let cell of row) {
						if(!cell.touched && !cell.mine) {
							won = false;
						}
					}
				}
				this.won = won;

			},
			flag(cell) {
				this.flaggedMines += 1 * (cell.flagged ? -1 : 1);
				cell.flagged = !cell.flagged;
			},
			getConnectedBlankCells(row,column,cells,visited) {
				const connected = [];
				visited = visited || [];
				for(let cell of this.getSurroundingCells(row,column,cells)) {
					if(visited.indexOf(cell) > -1) {
						continue;
					}
					visited.push(cell);
					if(cell.n) {
						// include the boundary cells
						connected.push(cell);
						continue;
					}
					connected.push(cell);

					for(let other_cell of this.getConnectedBlankCells(cell.row,cell.column,cells,visited)) {
						connected.push(other_cell);
					}
				}
				return connected;
			},
			getSurroundingCells(row,column,cells) {
				const surrounding = [];
				// N
				if(parseInt(row)-1 >= 0) {
					surrounding.push(cells[parseInt(row)-1][column]);
					// NW
					if(parseInt(column)-1 >= 0) {
						surrounding.push(cells[parseInt(row)-1][parseInt(column)-1]);
					}
					// NE
					if(parseInt(column)+2 <= numColumns) {
						surrounding.push(cells[parseInt(row)-1][parseInt(column)+1]);
					}
				}

				// S
				if(parseInt(row)+2 <= numRows) {
					surrounding.push(cells[parseInt(row)+1][column]);
					// SE
					if(parseInt(column)+2 <= numColumns) {
						surrounding.push(cells[parseInt(row)+1][parseInt(column)+1]);
					}
					// SW
					if(parseInt(column)-1 >= 0) {
						surrounding.push(cells[parseInt(row)+1][parseInt(column)-1]);
					}
				}
				// E
				if(parseInt(column)+2 <= numColumns) {
					surrounding.push(cells[row][parseInt(column)+1]);
				}
				// W
				if(parseInt(column)-1 >= 0) {
					surrounding.push(cells[row][parseInt(column)-1]);
				}
				return surrounding;
			},
			generateGrid() {
				const cells = [];

				for (var ri = 0; ri < numRows; ri++) {
					let row = [];
					for (var ci = 0; ci < numColumns; ci++) {
						const isMine = Math.random() * 1000 < 200;

						if(isMine) {
							this.mineCount += 1;
						}
						row.push({
							row: ri,
							column: ci,
							n: 0,
							flagged: false,
							touched: false,
							mine: isMine
						});
					}
					cells.push(row);
				}

				for(var row in cells) {
					for(var column in cells[row]) {
						let cell = cells[row][column];
						if(!cell.mine) {
							continue;
						}
						for(let cell of this.getSurroundingCells(row,column,cells)) {
							cell.n += 1;
						}
					}
				}

				this.cells = cells;
				return cells;
			}
		},
		created() {
			this.cells = this.generateGrid();
		}
	}
</script>

<style lang="scss">
	body {
		background-color: #575757;
	}
	.sprite {
		background: url('../assets/minesweeper_tiles.jpg') no-repeat 0 0;
		background-size: 240px 159px;
	}
	.board {
		padding-top: 16px;
		width: 359px;
		height: 553px;
		display: grid;
		grid-template-columns: repeat(6,1fr);
		grid-template-rows: repeat(10,1fr);

		.cell {
			@extend .sprite;
			font-size: 40px;
			display: flex;
			justify-content: center;
			align-items: center;
			background-position: -180px 0;

			&.mine {
				background-position: -120px 0;
			}

			.number {
				@extend .sprite;
				width: 100%;
				height: 100%;
			}
			.number-1 {
				@extend .number;
				background-position: 0 -53px;
			}
			.number-2 {
				@extend .number;
				background-position: -60px -53px;
			}
			.number-3 {
				@extend .number;
				background-position: -120px -53px;
			}
			.number-4 {
				@extend .number;
				background-position: -180px -53px;
			}
			.number-5 {
				@extend .number;
				background-position: 0 -106px;
			}
			.number-6 {
				@extend .number;
				background-position: -60px -106px;
			}
			.number-7 {
				@extend .number;
				background-position: -120px -106px;
			}
			.number-8 {
				@extend .number;
				background-position: -180px-106px;
			}
		}
		button {
			@extend .sprite;
			width: 100%;
			height: 100%;
			border: 0 none;
			appearance: none;
			padding: 0;
			margin: 0;

			&.flagged {
				background-position: -60px 0;
			}
		}
	}
</style>