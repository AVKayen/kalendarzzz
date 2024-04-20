<script>
    import { onMount } from 'svelte';

    onMount(() => {
        let num = fetch('http://localhost:8921/today')
            .then((response) => response.json())
            .then((data) => {
                window.location.href = `/calendar/${data.week}`;
            });
    });
</script>