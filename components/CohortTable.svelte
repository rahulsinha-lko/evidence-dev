<script context="module">
	export const evidenceInclude = true;
</script>

<script>
	import { QueryLoad } from '@evidence-dev/core-components';
	import CohortTable from './_CohortTable.svelte';
	export let data;

	// Remove any undefined props (e.g. w/o defaults) to prevent them from being passed
	$: spreadProps = Object.fromEntries(Object.entries($$props).filter(([, v]) => v !== undefined));
</script>

<!-- Pass all the props through-->
<QueryLoad {data} let:loaded>
	<CohortTable {...spreadProps} data={loaded?.__isQueryStore ? Array.from(loaded) : loaded}>
		<slot />
	</CohortTable>
</QueryLoad>
