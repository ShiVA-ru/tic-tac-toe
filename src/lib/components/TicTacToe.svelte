<script lang="ts">
	import { onMount } from 'svelte';
	import { slide } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';

	interface cellCoordinates {
		row: number;
		column: number;
	}

	interface Result {
		player: 1 | 2;
		start: cellCoordinates;
		end: cellCoordinates;
	}

	let canvas: HTMLCanvasElement;
	let context: CanvasRenderingContext2D;
	let activePlayer: 1 | 2 = 1;
	let stepCount = 0;
	let winnerName = '';

	const CELL_SIZE = 100;
	const FIELD_SIZE = CELL_SIZE * 3;
	const LINE_WIDTH = 5;

	const field = [
		[0, 0, 0],
		[0, 0, 0],
		[0, 0, 0]
	];

	onMount(() => {
		// const context = canvas.getContext('2d') as CanvasRenderingContext2D;
		if (!canvas) {
			throw new Error('Canvas not found');
		}

		context = canvas.getContext('2d')!;

		if (!context) {
			throw new Error('Canvas context not found');
		}

		// const canvasWidth = canvas.width; // Удалить?
		// const canvasHeight = canvas.height; // Удалить?

		context.lineWidth = LINE_WIDTH;
		context.strokeStyle = 'lightcoral';

		fieldDraw(context);
	});

	function fieldDraw(context: CanvasRenderingContext2D) {
		context.beginPath();

		for (let i = CELL_SIZE; i < FIELD_SIZE; i += CELL_SIZE) {
			context.moveTo(i, 0);
			context.lineTo(i, FIELD_SIZE);
			context.moveTo(0, i);
			context.lineTo(FIELD_SIZE, i);
		}

		context.stroke();
	}

	function crossDraw(context: CanvasRenderingContext2D, x: number = 0, y: number = 0) {
		context.beginPath();
		context.moveTo(25 + x, 25 + y);
		context.lineTo(75 + x, 75 + y);
		context.moveTo(25 + x, 75 + y);
		context.lineTo(75 + x, 25 + y);
		context.stroke();
	}

	function circleDraw(context: CanvasRenderingContext2D, x: number = 0, y: number = 0) {
		context.beginPath();
		context.arc(50 + x, 50 + y, 25, 0, Math.PI * 2, true);
		context.stroke();
	}

	function get_cell(event: MouseEvent) {
		const { clientX, clientY } = event;
		const { left, top } = canvas.getBoundingClientRect();
		const row = Math.floor((clientX - left) / CELL_SIZE);
		const column = Math.floor((clientY - top) / CELL_SIZE);
		console.log(row, column);
		return { row, column };
	}

	function checkResult() {
		if (field[0][0] === field[1][1] && field[0][0] === field[2][2] && field[0][0] !== 0) {
			let start: cellCoordinates = { row: 0, column: 0 };
			let end: cellCoordinates = { row: 2, column: 2 };
			return { player: field[0][0], start, end };
		} else if (field[0][2] === field[1][1] && field[0][2] === field[2][0] && field[0][2] !== 0) {
			let start: cellCoordinates = { row: 2, column: 0 };
			let end: cellCoordinates = { row: 0, column: 2 };
			// return field[0][2];
			return { player: field[0][2], start, end };
		} else {
			for (let i = 0; i < 3; i++) {
				if (field[i][0] === field[i][1] && field[i][0] === field[i][2] && field[i][0] !== 0) {
					let start: cellCoordinates = { row: i, column: 0 };
					let end: cellCoordinates = { row: i, column: 2 };
					return { player: field[i][0], start, end };
					// return field[i][0];
				} else if (
					field[0][i] === field[1][i] &&
					field[0][i] === field[2][i] &&
					field[0][i] !== 0
				) {
					let start: cellCoordinates = { row: 0, column: i };
					let end: cellCoordinates = { row: 2, column: i };
					return { player: field[0][i], start, end };
					// return field[0][i];
				}
			}
		}
		return 0;
	}

	function draw(event: MouseEvent) {
		const { row, column } = get_cell(event);
		const x = row * CELL_SIZE;
		const y = column * CELL_SIZE;

		if (field[row][column] !== 0) return;

		if (activePlayer === 1) {
			crossDraw(context, x, y);
		} else {
			circleDraw(context, x, y);
		}
		field[row][column] = activePlayer;

		stepCount++;

		if (stepCount === 9) {
			console.log('Ничья');
		}

		const result: Result | 0 = checkResult();

		if (result) {
			// winnerName = `Player ${player} `;
			victoryHandler(result);
		}

		activePlayer = activePlayer === 1 ? 2 : 1;
	}

	function victoryHandler(result: Result) {
		winnerName = `Player ${result.player} `;
		console.log(result);
		context.moveTo(100 * result.start.row, 100 * result.start.column);
		context.lineTo(100 * result.end.row + 100, 100 * result.end.column + 100);
		context.stroke();
		// context.fillStyle = 'rgba(0, 0, 0, 0.5)';
		// context.fillRect(0, 0, FIELD_SIZE, FIELD_SIZE);
	}

	function restartGame() {
		for (let i = 0; i < 3; i++) {
			for (let j = 0; j < 3; j++) {
				field[i][j] = 0;
			}
		}
		context.clearRect(0, 0, FIELD_SIZE, FIELD_SIZE);
		fieldDraw(context);
		winnerName = '';
		stepCount = 0;
		activePlayer = 1;
		// location.reload();
	}
</script>

<div class="game">
	{#if winnerName}
		<p
			transition:slide={{ delay: 250, duration: 500, easing: quintOut, axis: 'y' }}
			class="game__winner"
		>
			{winnerName} is win! Flawless victory!
		</p>
	{/if}
	{#if stepCount === 9}
		<p
			transition:slide={{ delay: 250, duration: 500, easing: quintOut, axis: 'y' }}
			class="game__winner"
		>
			Nobody is win! Try again!
		</p>
	{/if}
	<canvas
		bind:this={canvas}
		on:pointerdown={draw}
		class="game__canvas"
		width="300"
		height="300"
	>
	</canvas>

	<button
		on:click={restartGame}
		disabled={!winnerName}
		class="game__restart"
	>
		Restart game
	</button>
</div>

<style lang="scss">
	:root {
		font-family: 'Courier New', Courier, monospace;
		--main-color: lightcoral;
	}
	.game {
		display: flex;
		flex-direction: column;
		align-items: center;

		color: var(--main-color);
	}

	.game__winner {
		font-size: 15px;
		font-weight: 600;
		text-decoration: underline dashed;
		transition: opacity 0.5s ease-in-out;
	}

	.game__canvas {
		border: 5px var(--main-color) solid;
		margin-bottom: 20px;
	}

	.game__restart {
		padding: 10px 20px;

		font-family: 'Courier New', Courier, monospace;
		text-transform: uppercase;
		background-color: var(--main-color);
		border: none;
		border-radius: 20px;
		color: white;

		transition: opacity 0.5s ease-in-out;

		&:disabled {
			opacity: 0.5;
		}
	}

	// .canvas::after + winner_active {
	//   content: 'Winner!!!';
	// }
</style>
