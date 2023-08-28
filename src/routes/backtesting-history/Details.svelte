<script>
	import Loader from "../backtesting-setup/Loader.svelte";
	import Chart from "./Chart.svelte";
	import RangeValue from "./RangeValue.svelte";
    import { sum } from 'mathjs'

    export let details;
</script>

<div>
    {#if details}
    <div class="details">
        <p>asset: <span class="info">{details.asset}</span></p>
        <p>interval: <span class="info">{details.interval}</span></p>
        {#if details.fee_fixed}
            <p>fees: <span class="info">Fixed {details.fee_fixed}</span></p>
        {:else if details.fee_percent != undefined && details.fee_minimum != undefined}
            <p>fees: <span class="info">{details.fee_percent}% with minimum {details.fee_minimum}</span></p>
        {:else}
            <p>fees: <span class="info">None</span></p>
        {/if}
        <p>probability of filling order in 1st timeframe: <span class="info">{JSON.parse(details.action_probability)[0]}%</span></p>
        <p>probability of filling order in 2nd timeframe: <span class="info">{JSON.parse(details.action_probability)[1]}%</span></p>
        <p>probability of filling order in 3rd timeframe: <span class="info">{JSON.parse(details.action_probability)[2]}%</span></p>
        <p>probability of filling order in 4th timeframe: <span class="info">{JSON.parse(details.action_probability)[3]}%</span></p>
        <p>probability of filling order in 5th timeframe: <span class="info">{JSON.parse(details.action_probability)[4]}%</span></p>
        <p>code:<br/><span class="info">{details.code}</span></p>
        <p>runs: <span class="info">{details.variants.length}</span></p>
        <RangeValue name="number of buy transactions" values={details.variants.map((v) => v.n_buys)}/>
        <RangeValue name="number of sell transactions" values={details.variants.map((v) => v.n_sells)}/>
        <RangeValue name="paid fees" values={details.variants.map((v) => sum(JSON.parse(v.fees)))} />
        <RangeValue name="return of this method" values={details.variants.map((v) => v.return * 100)} digits=2 suffix='%' />
        <RangeValue name="return of HODL" values={details.variants.map((v) => v.return_hold * 100)} digits=2 suffix='%' />
        <Chart
            prices={details.price}
            buys_timestamps={details.variants.map((v) => JSON.parse(v.buys_timestamps))}
            sells_timestamps={details.variants.map((v) => JSON.parse(v.sells_timestamps))}
            fees={details.variants.map((v) => JSON.parse(v.fees))}/>
    </div>
    {:else}
        <Loader time="10s"/>
	{/if}
</div>

<style>
    .details {
        background: #eee;
        padding: 0.5rem 1rem;
        margin: 0.5rem;
        position: relative;
    }
    p {
        margin: 0;
        color: #0005;
    }
    .info {
        color: #0008;
    }
</style>