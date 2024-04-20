<script>
	import { fade } from 'svelte/transition';
    let contentKey = 0;

    $: contentKey += 1;

</script>

<div class="yay">
	<a href="/user"><h1>CleverCal</h1></a>
	{#key contentKey}
        <div class="ins" in:fade={{ duration: 300 }} out:fade={{ duration: 300 }}>
            <slot></slot>
        </div>
    {/key}
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap');
	* {
		font-family: 'Lato', sans-serif;
	}
	.yay {
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100vh;
		gap: 2rem;
		background-color: #e1e5ee;
		color: #2a324b;
		font-family: Roboto, sans-serif;
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
	}
	.ins {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		border-radius: 1rem;
	}
    a {
        all: unset;
        cursor: pointer;
    }
    a:hover {
        text-decoration: underline;
    }
</style>
