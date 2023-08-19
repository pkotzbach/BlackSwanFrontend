<svelte:head>
	<title>Backtesting History</title>
	<meta name="description" content="Backtesting" />
</svelte:head>

<script>
	import HistoryItem from "./HistoryItem.svelte";
	import add from "$lib/images/add.svg"
	let items = [
		{
			name: 'name 1',
			datetime: '19.08.2023 21:26',
			dataset: 'DS1',
			timeframe_start: '1.05.2023',
			timeframe_end: '1.07.2023',
			return_method: 158.85,
			return_hold: 34.71,
		},
		{
			name: 'name 2',
			datetime: '19.08.2023 21:26',
			dataset: 'DS1',
			timeframe_start: '1.05.2023',
			timeframe_end: '1.07.2023',
			return_method: 158.85,
			return_hold: 34.71,
		},
		{
			name: 'name 3',
			datetime: '19.08.2023 21:26',
			dataset: 'DS1',
			timeframe_start: '1.05.2023',
			timeframe_end: '1.07.2023',
			return_method: 158.85,
			return_hold: 34.71,
		},
	];
</script>

<div>
	<div class="title">
		<h1>History</h1>
		<img class="add" src={add} on:click={() => window.location.href = '/backtesting-setup'} />
	</div>

	<div class="two-column">
		<div class="history">
			{#each items as item (item)}
				<HistoryItem bind:item={item} />
			{/each}
		</div>
		<div class="details">
			<p>details</p>
		</div>
	</div>
</div>

<style>
	.title {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
	}
	h1 {
		margin: 0 1rem 0 0;
	}
	.add {
		width: 2.5rem;
		margin-top: 0.5rem;
		cursor: pointer;
	}
	.two-column {
		display: grid;
		margin: auto;
		width: 72rem;
		grid-template-columns: 1fr 1fr;
	}
</style>