<script>
	import { fly } from 'svelte/transition';
</script>

<div in:fly={{ y: -50, duration: 250, delay: 251 }} out:fly={{ y: -50, duration: 250 }}>
	<slot />
</div>
