<svelte:head>
	<title>Backtesting Setup</title>
	<meta name="description" content="Backtesting" />
</svelte:head>

<script>
	import RadioButtonGroup from "./RadioButtonGroup.svelte";
	import TitleDivider from "./TitleDivider.svelte";
	import Slider from '@bulatdashiev/svelte-slider';

	import progress_fail from "$lib/images/progress_fail.svg"
	import progress_pass from "$lib/images/progress_pass.svg"
	import progress_wait from "$lib/images/progress_wait.svg"
	import progress_running from "$lib/images/progress_running.svg"

	let asset_options = ['BTCUSDT', 'ETHUSDT', '+']
	let asset_selected = null
	let interval_options = ['1d', '1h', '1m', '1s']
	let interval_selected = null
	let timeframe_min = 1502942400000
	let timeframe_max = Date.now()
	let timeframe_range = [timeframe_min, timeframe_max]
	let fees_options = ['None', 'Fixed XXX', 'XXX percent of value with fixed minimum XXX']
	let fees_selected = null
	let action_probability = [70, 20, 5, 4, 1]

	let editable = true
	let execution_stages = ['pass', 'pass', 'fail', 'running', 'waiting', 'waiting', 'waiting']

	
	// get available assets (check which datasets are downloaded)
	// get available datasets (check which datasets are created)
	// start run (on websocket)
	// show results (probably final message from start run)

	// set all vales if rerun

	function parseUnixTimestamp(timestamp) {
		return new Date(timestamp).toLocaleDateString("pl-PL")
	}


	function run(event) {
		editable = false
	}
</script>

<div class="text-column">
	<TitleDivider title={"Asset"}/>
	{#if editable}
		<RadioButtonGroup bind:options={asset_options} bind:selected_option={asset_selected}/>
	{/if}

	<TitleDivider title={"Interval"}/>
	{#if editable}
		<RadioButtonGroup bind:options={interval_options} bind:selected_option={interval_selected}/>
	{/if}

	<TitleDivider title={"Additional Dataset"}/>
	{#if editable}
		<p>TODO</p>
	{/if}

	<TitleDivider title={"Timeframe"}/>
	{#if editable}
		<div class="timeframe">
		<p>{parseUnixTimestamp(timeframe_range[0])}</p>
		<Slider bind:value={timeframe_range} min={timeframe_min} max={timeframe_max} range on:input={() => {
			if (timeframe_range[0] > timeframe_range[1]) timeframe_range = [timeframe_range[1], timeframe_range[0]]}
		}/>
		<p>{parseUnixTimestamp(timeframe_range[1])}</p>
	</div>
	{/if}

	<TitleDivider title={"Fees"}/>
	{#if editable}
		<RadioButtonGroup bind:options={fees_options} bind:selected_option={fees_selected}/>
	{/if}

	<TitleDivider title={"Real Life Limits"}/>
	{#if editable}
		<p>Assuming limits of supply and demand, not all actions will be filled in the same timeframe they were placed. Mark what is the probability the action will be filled in:</p>
		<div class="limits">
			<li>1st timeframe (instant) <input class="text-input" type="text" placeholder={action_probability[0].toString()}/>%</li>
			<li>2nd timeframe <input class="text-input" type="text" placeholder={action_probability[1].toString()}/>%</li>
			<li>3rd timeframe <input class="text-input" type="text" placeholder={action_probability[2].toString()}/>%</li>
			<li>4th timeframe <input class="text-input" type="text" placeholder={action_probability[3].toString()}/>%</li>
			<li>5th timeframe <input class="text-input" type="text" placeholder={action_probability[4].toString()}/>%</li>
		</div>
	{/if}

	<TitleDivider title={"Code"}/>
	{#if editable}
		<p>Write the code that will take actions.</p> 
		<p>Here are the available variables:
			{#each ['VAR1', 'VAR2'] as v}
				<span class="var">{v}</span><span>, </span>
			{/each}
		</p> 
		<textarea rows=15></textarea>
	{/if}

	<TitleDivider title={"Run"}/>
	{#if editable}
		<button id="run" on:click={run}>RUN</button>
	{/if}

	{#if !editable}
	{#each execution_stages as stage, i}
		<div class="progress-row">
			{#if stage == 'pass'}
			<img src={progress_pass}/>
			{:else if stage == 'fail'}
			<img src={progress_fail}/>
			{:else if stage == 'running'}
			<img src={progress_running} class="spin"/>
			{:else if stage == 'waiting'}
			<img src={progress_wait}/>
			{/if}
			<p>
				{#if i == 0}
				Downloading
				{:else if i == execution_stages.length - 1}
				Analyzing results
				{:else}
				Running test {i}
				{/if}
			</p>
		</div>
		{/each}
	{/if}

</div>

<style>
	p {
		margin: 0 1rem;
	}
	
	.limits {
	  	margin: 0 2rem;
	}
	
	.text-input {
		border-top: none;
		border-left: none;
		border-right: none;
		background: transparent;
		width: 4ch;
		text-align: center;
	}
	
	textarea {
		margin: 0.5rem 1rem;
		resize: vertical;
	}
	
	.var {
		color: #0005;
        font-weight: bold;
		border: 1px solid #0005;
		border-radius: 0.5rem;
		padding: 0.2rem;
    }

	.timeframe {
		--progress-bg: #ff3e0055;
		--thumb-bg: #ff3e00;
		display: flex;
		justify-content: center;
	}
	.timeframe p {
		margin: 0.4rem 1rem 0 1rem;
		padding: 0;
	}

	#run {
		margin: auto;
		background: #ff3e00aa;
		border: none;
		padding: 1rem 3rem;
		cursor: pointer;
	}

	.progress-row {
		display: flex;
		padding: 0.2rem;
	}
	
	.progress-row img {
		width: 1.5rem;
	}

	.progress-row img.spin {
		animation: rotation 3s infinite linear;
	}
	@keyframes rotation {
		from { transform: rotate(359deg); }
		to { transform: rotate(0deg); }
	}
</style>