<script>
    export let prices
    export let buys_timestamps
    export let sells_timestamps
    export let fees

    import Chart from 'chart.js/auto';
    let zoomPlugin

    import { onMount } from 'svelte';

    let chartCanvas
    let chart

    function calculateBaseAccountValue() {
        let multiplier = 1000000 / prices[0].price
        return prices.map((p) => p.price * multiplier)
    }

    function calculateAccountValue(buys, sells, fees) {
        let values = [1000000]
        let buys_list = JSON.parse(JSON.stringify(buys))
        let sells_list = JSON.parse(JSON.stringify(sells))
        let fees_list = JSON.parse(JSON.stringify(fees))
        let prices_list = JSON.parse(JSON.stringify(prices))
        let has_actions = false
        prices_list.reduce((prev, curr) => {
            let fee = 0
            if (!prev) return curr
            if (buys_list.includes(curr.timestamp)) {
                has_actions = true
                fee = fees_list.shift()
                buys_list.shift()
            }
            else if (sells_list.includes(curr.timestamp)) {
                has_actions = false
                fee = fees_list.shift()
                sells_list.shift()
            }
            if (has_actions) values.push(((curr.price / prev.price) * values.slice(-1)[0]) - fee)
            else values.push(values.slice(-1)[0])
            
            return curr
        }, 0)
        return values
    }

    function prepareChart() {
        const datasets = [{
            label: 'stock price',
            data: calculateBaseAccountValue(),
            borderColor: "hsl(15,100%,60%)",
            backgroundColor: "hsl(15,100%,60%)",
            pointStyle: false,
        }]
        for (let i = 0; i < buys_timestamps.length; i++) {
            datasets.push({
                label: `test run ${i}`,
                data: calculateAccountValue(buys_timestamps[i], sells_timestamps[i], fees[i]),
                borderColor: "hsl(15,100%,80%)",
                backgroundColor: "hsl(15,100%,80%)",
                pointStyle: function(context) {
                    if (buys_timestamps[i].includes(prices[context.dataIndex].timestamp)) return 'circle'
                    if (sells_timestamps[i].includes(prices[context.dataIndex].timestamp)) return 'circle'
                    return false
                },
                pointBackgroundColor: function(context) {
                    if (buys_timestamps[i].includes(prices[context.dataIndex].timestamp)) return 'blue'
                    if (sells_timestamps[i].includes(prices[context.dataIndex].timestamp)) return 'green'
                    return ''
                }
            })
        }

        const data = {
            labels: prices.map((p) => p.timestamp),
            datasets: datasets
        }
        const config = {
            type: 'line',
            data: data,
            options: {
                responsive: true,
                plugins: {
                    legend: {
                        position: 'top',
                    },
                    title: {
                        display: true,
                        text: 'Stock price & account value'
                    },
                    zoom: {
                        zoom: {
                            wheel: {
                                enabled: true,
                            },
                            pinch: {
                                enabled: true
                            },
                            mode: 'x',
                        },
                        pan: {
                            enabled: true,
                            mode: 'x',
                        },
                    },
                    tooltip: {
                        enabled: true,
                        callbacks: {
                            title: (index) => {
                                let d = new Date(prices[index[0].dataIndex].timestamp)
                                return d.toLocaleDateString("pl-PL") + " " + d.toLocaleTimeString("pl-PL")
                            }
                        },
                    }
                },
                scales: {
                    x: {
                        ticks: {
                            callback: (index) => new Date(prices[index].timestamp).toLocaleDateString("pl-PL")
                        }
                    }
                },
            }
        }
        if (chart) chart.destroy()
        chart = new Chart(chartCanvas.getContext('2d'), config)
    }

    onMount(async () => {

        // loaded on client, server side rendering throws error in dependency Hammer.js
        zoomPlugin = (await import('chartjs-plugin-zoom')).default
        Chart.register(zoomPlugin);

        prepareChart()
    })

    $: if (chartCanvas && prices && buys_timestamps && sells_timestamps && fees) {
        prepareChart()
    }

</script>

<canvas bind:this={chartCanvas}></canvas>
