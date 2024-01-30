<script lang="ts">
	import TicTacToe from '$lib/components/TicTacToe.svelte';
</script>

<div class="container">
	<h1>Tic-tac-toe</h1>
	<p>Play with me</p>
	<TicTacToe />
</div>

<style lang="scss">
	.container {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;

		max-width: fit-content;
		margin: 0 auto;

		color: var(--main-color);
	}
</style>
