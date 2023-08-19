<svelte:head>
	<title>Backtesting Setup</title>
	<meta name="description" content="Backtesting" />
</svelte:head>

<script>
	import RadioButtonGroup from "./RadioButtonGroup.svelte";
	import TitleDivider from "./TitleDivider.svelte";
	import Slider from '@bulatdashiev/svelte-slider';

	let timeframe_range = [0, 100]
	// get available assets
	// get available datasets
	// get all values on run

	// set all vales if rerun
</script>

<div class="text-column">
	<TitleDivider title={"Asset"}/>
	<RadioButtonGroup options={['BTCUSDT', 'ETHUSDT', '+']}/>

	<TitleDivider title={"Interval"}/>
	<RadioButtonGroup options={['1d', '1h', '1m', '1s']}/>

	<TitleDivider title={"Additional Dataset"}/>
	<p>Choose additional dataset like tweets, etc</p>

	<TitleDivider title={"Timeframe"}/>
	<div class="timeframe">
		<p>01.05.2023</p>
		<Slider bind:value={timeframe_range} range on:input={e => console.log(e.detail)}/>
		<p>31.07.2023</p>
	</div>

	<TitleDivider title={"Fees"}/>
	<RadioButtonGroup options={['None', 'Fixed XXX', 'XXX percent of value with fixed minimum XXX']}/>

	<TitleDivider title={"Real Life Limits"}/>
	<p>Assuming limits of supply and demand, not all actions will be filled in the same timeframe they were placed. Mark what is the probability the action will be filled in:</p>
	<div class="limits">
		<li>1st timeframe (instant) <input class="text-input" type="text" placeholder="70"/>%</li>
		<li>2nd timeframe <input class="text-input" type="text" placeholder="20"/>%</li>
		<li>3rd timeframe <input class="text-input" type="text" placeholder="5"/>%</li>
		<li>4th timeframe <input class="text-input" type="text" placeholder="3"/>%</li>
		<li>5th timeframe <input class="text-input" type="text" placeholder="2"/>%</li>
	</div>

	<TitleDivider title={"Code"}/>
	<p>Write the code that will take actions.</p> 
	<p>Here are the available variables:
        {#each ['VAR1', 'VAR2'] as v}
			<span class="var">{v}</span><span>, </span>
		{/each}
	</p> 
	<textarea rows=15></textarea>

	<TitleDivider title={"Run"}/>
	<button id="run">RUN</button>
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
</style>