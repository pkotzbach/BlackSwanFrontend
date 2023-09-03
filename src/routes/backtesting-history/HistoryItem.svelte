<script>
    export let item;
    export let load_details;
    export let delete_item;
	import removeImg from "$lib/images/remove.svg"
	import restartImg from "$lib/images/restart.svg"

    function parseUnixTimestamp(timestamp, time=false) {
		let d = new Date(timestamp)
        if (!time) return d.toLocaleDateString("pl-PL")
        return d.toLocaleDateString("pl-PL") + " " + d.toLocaleTimeString("pl-PL")
	}

    function retry() {
        console.log('retry')
        window.location.href = `/backtesting-setup?timestamp=${item.timestamp}&`
    }

    

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div class="history-item">
    <p class="title" on:click={load_details(item.timestamp)}><span class="bold">{item.name}</span></p>
    <p>run on <span class="info">{parseUnixTimestamp(item.timestamp, true)}</span></p>
    <p>dataset: <span class="info">{item.asset} {item.interval}</span></p>
    <p>from <span class="info">{parseUnixTimestamp(item.timeframe_min)}</span>
        to <span class="info">{parseUnixTimestamp(item.timeframe_max)}</span></p>
    <p>returned <span class="info">{parseInt(item.return * 100)}%</span>
        (compared to {parseInt(item.return_hold * 100)}% on hold)</p>
    <img class="action" style="top: 4rem;" src={removeImg} on:click={delete_item(item.timestamp)}/>
    <img class="action" style="top: 1rem;" src={restartImg} on:click={retry}/>
</div>

<style>
    .history-item {
        background: #eee;
        padding: 0.5rem 1rem;
        margin: 0.5rem;
        position: relative;
    }
    .title {
        cursor: pointer;
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
