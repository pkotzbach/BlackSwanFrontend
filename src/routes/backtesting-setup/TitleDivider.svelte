<script>
    export let title;
</script>

<div class="divider">
    <div class="line"></div>
    <p class="text">{title}</p>
    <div class="line"></div>
</div>

<style>
    .divider {
        width: 48rem;
        display: flex;
        align-items: center;
        margin-top: 1rem;
    }
    .text {
        color: #0005;
        margin: 0 1rem;
    }
    .line {
        height: 1px;
        background: #0005;
        flex: 1;
    }
</style>