<script>
    export let name
    export let values
    export let suffix = ''
    export let digits = 0

    import { min, max, mean, std } from 'mathjs'


    let v_min
    let v_max
    let v_mean
    let v_std

    $: if(values) {
        v_min = min(values)
        v_max = max(values)
        v_mean = mean(values)
        v_std = std(values)
    }
</script>

<p>{name}:
    min <span class="info">{v_min.toFixed(digits)}{suffix}</span>
    mean <span class="info">{v_mean.toFixed(digits)}{suffix}</span>
    std <span class="info">{v_std.toFixed(digits)}{suffix}</span>
    max <span class="info">{v_max.toFixed(digits)}{suffix}</span>
</p>

<style>
    p {
        margin: 0;
        color: #0005;
    }
    .info {
        color: #0008;
    }
</style>