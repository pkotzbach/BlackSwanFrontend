<svelte:head>
	<title>Backtesting Setup</title>
	<meta name="description" content="Backtesting" />
</svelte:head>

<script>
	import { onMount } from "svelte";
	
	import RadioButtonGroup from "./RadioButtonGroup.svelte";
	import TitleDivider from "./TitleDivider.svelte";
	import Loader from "./Loader.svelte";
	import Slider from '@bulatdashiev/svelte-slider';

	import progress_fail from "$lib/images/progress_fail.svg"
	import progress_pass from "$lib/images/progress_pass.svg"
	import progress_wait from "$lib/images/progress_wait.svg"
	import progress_skip from "$lib/images/progress_skip.svg"
	import progress_running from "$lib/images/progress_running.svg"

	function get(url, pass, fail) {
		fetch(`http://localhost:8000/backtesting/${url}`)
		.then(response => response.json())
		.then(data => pass(data))
		.catch(error => fail(error))
	}
	function get_assets() {
		get("assets",
		(data) => asset_options = data.concat([{'name': '+'}]),
		(error) => alert(`Failed getting available assets: ${error}`),
	) }
	function get_default() {
		const url = new URL(window.location.href)
		const timestamp = url.searchParams.get('timestamp')
		if (!timestamp) return
		get(`history/${timestamp}`,
			(data) => {
				console.log(data)
				test_name = data.name
				asset_selected = data.asset
				interval_selected = data.interval
				timeframe_min = data.timeframe_min
				timeframe_max = data.timeframe_max
				fees_value_fixed = data.fee_fixed
				fees_value_percent = data.fee_percent
				fees_value_minimum = data.fee_minimum
				action_probability = JSON.parse(data.action_probability)
				code_text = data.code
			},
			(error) => alert(`Failed getting details: ${error}`),
		)
	}
	
	let test_name = ''
	let asset_options = [{'name': '+'}]
	let asset_selected = null
	let prev_asset_selected = null
	let interval_options = ['1d', '1h', '1m', '1s']
	let interval_selected = null
	let timeframe_min = null
	let timeframe_max = null
	let timeframe_range = [null, null]
	let fees_options = ['None', 'Fixed XXX', 'XXX % of value with fixed minimum XXX']
	let fees_selected = null
	let fees_value_fixed = null
	let fees_value_percent = null
	let fees_value_minimum = null
	let action_probability = [70, 20, 5, 4, 1]
	let code_text = null

	onMount(async () => {
		get_assets()
		get_default()
	})
	$: if (asset_selected && asset_selected !== '+') {
		if (asset_selected !== prev_asset_selected) {
			timeframe_min = asset_options.filter(a => a.name == asset_selected)[0].start_time
			timeframe_max = Date.now()
			timeframe_range = [timeframe_min, timeframe_max]
			prev_asset_selected = asset_selected
		}
	}

	let editable = true
	let execution_socket = null
	let execution_stages = []
	let loader_cls = "loader"

	function parseUnixTimestamp(timestamp) {
		return new Date(timestamp).toLocaleDateString("pl-PL")
	}

	function validate() {
		let errors = ''
		if (!test_name) errors += `* Specify unique and recognizable test name \n`
		if (!asset_selected) errors += `* Select asset \n`
		if (!interval_selected) errors += `* Select interval \n`
		if (!timeframe_min && !timeframe_max) errors += `* Select timeframe \n`
		// fees
		if (!fees_selected) errors += `* Select fees type \n`
		else {
			fees_value_fixed = null
			fees_value_percent = null
			fees_value_minimum = null
			// get selected fee value
			let field = document.querySelector("#fees .radio-item.selected")
			if (field?.innerHTML.includes("Fixed")) {
				fees_value_fixed = parseFloat(field.querySelector("input[type=text]")?.value.replace(",", "."))
				if (!fees_value_fixed) errors += `* Fill values for selected fees type \n`
			}
			else if (field?.innerHTML.includes("% of value with fixed minimum")) {
				fees_value_percent = parseFloat(field.querySelectorAll("input[type=text]")[0]?.value.replace(",", "."))
				fees_value_minimum = parseFloat(field.querySelectorAll("input[type=text]")[1]?.value.replace(",", "."))
				console.log(fees_value_minimum, fees_value_percent)
				if (fees_value_percent == undefined || fees_value_minimum == undefined) errors += `* Fill values for selected fees type \n`
			}
		}
		// limits
		let action_probability_sum = action_probability.map(p => parseFloat(p)).reduce((s, c) => s + c, 0)
		if (action_probability_sum != 100) {
			errors += `* All the probabilities for action fill timeframe have to sum to 100%, not ${action_probability_sum}%\n`
		}
		if (action_probability.some(p => p < 0)) {
			errors += `* All the probabilities for action fill timeframe cannot be below 0% \n`
		}
		// code TODO
		if (!code_text) errors += `* Create login code \n`

		if (!errors) return true
		alert(errors)
		return false
	}
	function run(event) {
		if (!validate()) return
		// editable = false
		execution_socket = new WebSocket("ws://localhost:8000/backtesting/run")
		execution_socket.addEventListener("open", () => {
			// send params
			execution_socket.send(JSON.stringify({
				name: test_name,
				asset: asset_selected,
				interval: interval_selected,
				timeframe_min: timeframe_min,
				timeframe_max: timeframe_max,
				fee_fixed: fees_value_fixed,
				fee_minimum: fees_value_minimum,
				fee_percent: fees_value_percent,
				action_probability: action_probability,
				code: code_text
			}))
			loader_cls = "loader visible"
		})
		execution_socket.addEventListener("close", () => {
			loader_cls = "loader"
		})
		execution_socket.addEventListener("message", (event) => {
			let data = JSON.parse(event.data)
			if (data.statuses)
				execution_stages = data.statuses
			if (data.redirect)
				window.location.pathname = "/backtesting-history";
		})
	}
</script>

<div class="text-column">
	<TitleDivider title={"Name"}/>
	{#if editable}
		<input class="text-input name" type="text" bind:value={test_name}/>
	{/if}
	
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

	{#if !asset_selected}
		<TitleDivider title={"Timeframe (select asset)"}/>
	{:else}
		<TitleDivider title={"Timeframe"}/>
		{#if editable}

			<div class="timeframe">
			<p>{parseUnixTimestamp(timeframe_range[0])}</p>
			<Slider bind:value={timeframe_range} min={timeframe_min} max={timeframe_max} step=1000 range on:input={() => {
				if (timeframe_range[0] > timeframe_range[1]) timeframe_range = [timeframe_range[1], timeframe_range[0]]}
			}/>
			<p>{parseUnixTimestamp(timeframe_range[1])}</p>
		</div>
		{/if}
	{/if}

	<TitleDivider title={"Fees"}/>
	{#if editable}
		<RadioButtonGroup id={"fees"} bind:options={fees_options} bind:selected_option={fees_selected}/>
	{/if}

	<TitleDivider title={"Real Life Limits"}/>
	{#if editable}
		<p>Assuming limits of supply and demand, not all actions will be filled in the same timeframe they were placed. Mark what is the probability the action will be filled in:</p>
		<div class="limits">
			<li>1st timeframe (instant) <input class="text-input" type="text" bind:value={action_probability[0]}/>%</li>
			<li>2nd timeframe <input class="text-input" type="text" bind:value={action_probability[1]}/>%</li>
			<li>3rd timeframe <input class="text-input" type="text" bind:value={action_probability[2]}/>%</li>
			<li>4th timeframe <input class="text-input" type="text" bind:value={action_probability[3]}/>%</li>
			<li>5th timeframe <input class="text-input" type="text" bind:value={action_probability[4]}/>%</li>
		</div>
	{/if}

	<TitleDivider title={"Code"}/>
	{#if editable}
		<p>Write the code that will take actions.</p>
		<p>If you want to buy, assign <span class="var">action='buy'</span>.</p>
		<p>If you want to buy, assign <span class="var">action='sell'</span>.</p> 
		<p>If you don't want to do anuthing, dont assign any value.</p> 
		<p>Here are the available variables:
			{#each [
				'price_open', 'price_high', 'price_low', 'price_close', 'volume', 'asset_volume_quote',
				'trades_number', 'asset_volume_taker_base', 'asset_volume_taker_quote', 'rsi', 'cci', 'williams', 'sar', 'ma', 'ema', 'obv',
				'adl', 'adx', 'High-Low', 'High-PrevClose', 'Low-PrevClose', 'TR', 'UpMove', 'DownMove', 'adx_dm_pos', 'adx_dm_neg',
				'adx_di_pos', 'adx_di_neg', 'adx_dx', 'aroon', 'High_Rolling_Max', 'Low_Rolling_Min', 'Periods_Since_Highest',
				'Periods_Since_Lowest', 'Aroon_Up', 'Aroon_Down', 'macd', 'EMA_12', 'EMA_26', 'MACD_Line', 'MACD_Signal_Line',
				'so', '%K', 'Middle Band', 'Standard Deviation', 'boll_up', 'boll_down', 'ichi_conversion', 'ichi_base', 'ichi_span_a',
				'ichi_span_b', 'ichi_span_lagging'
			] as v}
				<span class="var">{v}</span><span>, </span>
			{/each}
		</p> 
		<textarea bind:value={code_text} rows=15></textarea>
	{/if}

	<TitleDivider title={"Run"}/>
	{#if editable}
		<button id="run" on:click={run}>RUN</button>
	{/if}

	<!-- {#if !editable} -->
	<div class="progress">
		<div class={loader_cls}><Loader/></div>
		<div class="stages">
			{#each execution_stages as stage}
			<div class="progress-row">
				{#if stage.stage == 'pass'}
				<img src={progress_pass}/>
				{:else if stage.stage == 'fail'}
				<img src={progress_fail}/>
				{:else if stage.stage == 'running'}
				<img src={progress_running} class="spin"/>
				{:else if stage.stage == 'waiting'}
				<img src={progress_wait}/>
				{:else if stage.stage == 'skip'}
				<img src={progress_skip}/>
				{/if}
				<p>{stage.name}</p>
			</div>
			{/each}
		</div>
	</div>
	<!-- {/if} -->

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

	.name {
		width: 50ch;
		margin: auto;
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

	.progress {
		display: flex;
		position: relative;
		justify-content: center;
		align-items: center;
		width: 48rem;
		height: 28rem;
	}

	.progress .stages {
		background: #fff9;
	}
	.progress .loader {
		position: absolute;
		opacity: 0.5;
		width: 80%;
		z-index: -1;
		visibility: hidden;
	}
	.progress .loader.visible {
		visibility: visible;
	}
</style>