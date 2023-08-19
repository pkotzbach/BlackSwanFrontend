<script>
    export let item;
	import remove from "$lib/images/remove.svg"
	import restart from "$lib/images/restart.svg"
</script>

<div class="history-item">
    <p><span class="bold">{item.name}</span></p>
    <p>run on <span class="info">{item.datetime}</span></p>
    <p>dataset: <span class="info">{item.dataset}</span> from <span class="info">{item.timeframe_start}</span> to <span class="info">{item.timeframe_end}</span></p>
    <p>returned <span class="info">{item.return_method}%</span> (compared to {item.return_hold}% on hold)</p>
    <img class="action" style="top: 4rem;" src={remove} />
    <img class="action" style="top: 1rem;" src={restart} on:click={() => window.location.href = '/backtesting-setup'}/>

</div>

<style>
    .history-item {
        background: #eee;
        padding: 0.5rem 1rem;
        margin: 0.5rem;
        position: relative;
    }
    p {
        margin: 0;
        color: #0005;
    }
    .bold {
        font-weight: bold;
        font-size: 1.5rem;
    }
    .info {
        color: #0008;
    }
    .action {
		width: 2.5rem;
        position: absolute;
        right: 1rem;
        cursor: pointer;
    }
</style>
