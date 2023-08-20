<script>
    export let options = [];
    export let selected_option = null;
  
    function handleAddingNewAsset() {
        selected_option = null
        const newAsset = window.prompt('Add new asset')
        if (!newAsset) return
        fetch(`http://localhost:8000/backtesting/assets/${newAsset}`, {
            method: 'PUT',
			body: JSON.stringify({name: newAsset})
        })
		.then(response => response.json())
		.then(data => {
            if (data.name && data.start_time) {
                options = options.slice(0, -1).concat([data, {'name': '+'}])
                selected_option = newAsset
            } else {
                alert(`Failed adding asset ${newAsset}: ${data.detail}`)
            }
        })
		.catch(error => alert(`Failed adding asset ${newAsset}: ${error}`))
    }

    function handleOptionChange(event) {
        if (event.target.value == '+') {
            handleAddingNewAsset()
        } else {
            selected_option = event.target.value;
        }
    }

    function last(a, i){ return i == a.length - 1 }

</script>

<div class="radio-group">
{#each options.map(o => o.name ? o.name : o) as option}
    <label class="radio-item {selected_option === option && 'selected'}">
    <input type="radio" name="radio-group" value={option} class="radio-button" bind:group={selected_option} on:change={handleOptionChange}/>
    {#if option.includes('XXX')}
        {#each option.split('XXX') as part, i}
            {part}
            {#if !last(option.split('XXX'), i)}
                <input class="text-inside" type="text"/>
            {/if}
        {/each}
    {:else}
        {option}
    {/if}
    </label>
{/each}
</div>

<style>
.radio-group {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
}

.radio-item {
    border: 1px solid #ccc;
    border-radius: 0.5rem;
    padding: 0.5rem 0.7rem 0.5rem 0;
    margin: 0 0.5rem 0.5rem 0.5rem;
    cursor: pointer;
    transition: border-color 0.3s ease-in-out;
}

.radio-button {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
}

.radio-item.selected {
    border-color: #ff3e00;
    background-color: #ff3e0011;
}

.radio-item:hover {
    background-color: #ff3e0005;
}

.text-inside {
    border-top: none;
    border-left: none;
    border-right: none;
    background: transparent;
    width: 4ch;
    text-align: center;
}

</style>