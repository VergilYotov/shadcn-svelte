<script lang="ts">
	import TimeRangeInput from "$lib/registry/blocks/time-range-input.svelte";
	import * as Card from "$lib/registry/ui/card/index.js";
	import { Button } from "$lib/registry/ui/button/index.js";
	import { parseDate } from "chrono-node";

	// Helper function to format date as "YYYY-MM-DD HH:MM:SS"
	function formatDateWithTime(date: Date): string {
		const year = date.getFullYear();
		const month = String(date.getMonth() + 1).padStart(2, "0");
		const day = String(date.getDate()).padStart(2, "0");
		const hours = String(date.getHours()).padStart(2, "0");
		const minutes = String(date.getMinutes()).padStart(2, "0");
		const seconds = String(date.getSeconds()).padStart(2, "0");
		return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
	}

	// Initialize with "5 days ago" to "now" using chrono-node
	let fromDate = parseDate("5 days ago", new Date(), { forwardDate: true });
	let toDate = parseDate("now", new Date(), { forwardDate: true });

	let fromValue = $state(fromDate ? formatDateWithTime(fromDate) : "");
	let toValue = $state(toDate ? formatDateWithTime(toDate) : "");

	function handleExport() {
		alert(`Exporting data from:\n${fromValue}\nto:\n${toValue}`);
	}

	function handleClear() {
		fromValue = "";
		toValue = "";
	}
</script>

<Card.Root class="w-full max-w-md">
	<Card.Header>
		<Card.Title>Export Data by Date Range</Card.Title>
		<Card.Description>
			Select a date range to export your data. Type natural language like "today 00:00"
			to "now" or "-5d" to "today", or use the calendar picker.
		</Card.Description>
	</Card.Header>
	<Card.Content class="flex flex-col gap-4">
		<TimeRangeInput id="export-range" bind:fromValue bind:toValue />
	</Card.Content>
	<Card.Footer class="flex justify-between">
		<Button variant="outline" onclick={handleClear}>Clear</Button>
		<Button onclick={handleExport}>Export Data</Button>
	</Card.Footer>
</Card.Root>
