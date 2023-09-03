<svelte:head>
	<title>Backtesting History</title>
	<meta name="description" content="Backtesting" />
</svelte:head>

<script>
	import { onMount } from "svelte";

	import HistoryItem from "./HistoryItem.svelte";
	import add from "$lib/images/add.svg"
	import Details from "./Details.svelte";

	function get(url, pass, fail) {
		fetch(`http://localhost:8000/backtesting/${url}`)
		.then(response => response.json())
		.then(data => pass(data))
		.catch(error => fail(error))
	}
	function remove(url, pass, fail) {
		fetch(`http://localhost:8000/backtesting/${url}`, { method: 'DELETE'})
		.then(response => response.json())
		.then(data => pass(data))
		.catch(error => fail(error))
	}
	function get_history() {
		get("history",
		(data) => items = data,
		(error) => alert(`Failed getting history: ${error}`),
	) }

	let items = []
	onMount(async () => get_history())
	
	let details = null
    function load_details(timestamp) {
		get(`history/${timestamp}`,
		(data) => details = data,
		(error) => alert(`Failed getting details: ${error}`),
	) }
	function delete_item_from_history(timestamp) {
		remove(`history/${timestamp}`,
		(data) => items = items.filter((i) => i.timestamp != timestamp),
		(error) => alert(`Failed getting history: ${error}`),
	) }
</script>

<div>
	<div class="title">
		<h1>History</h1>
		<img class="add" src={add} on:click={() => window.location.href = '/backtesting-setup'} />
	</div>

	<div class="two-column">
		<div class="history">
			{#each items as item (item)}
				<HistoryItem bind:item={item} load_details={load_details} delete_item={delete_item_from_history}/>
			{/each}
		</div>
		<div class="details">
			<Details bind:details={details}/>
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
		grid-template-columns: 1fr 2fr;
		grid-gap: 1rem;
	}
</style>