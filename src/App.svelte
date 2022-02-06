<script>
	import TaxCalculator from './TaxCalculator.svelte';
</script>

<main>
	<TaxCalculator/>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 280px;
		margin: 5px;
		background-color: black;
		color: #4d54b3;
		border: "#4d54b3S";
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>