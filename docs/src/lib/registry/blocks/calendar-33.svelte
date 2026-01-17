<script lang="ts">
	import { Label } from "$lib/registry/ui/label/index.js";
	import * as Popover from "$lib/registry/ui/popover/index.js";
	import { Button } from "$lib/registry/ui/button/index.js";
	import RangeCalendar from "$lib/registry/ui/range-calendar/range-calendar.svelte";
	import { Input } from "$lib/registry/ui/input/index.js";
	import CalendarIcon from "@lucide/svelte/icons/calendar";
	import type { DateRange } from "bits-ui";
	import { CalendarDate, type DateValue } from "@internationalized/date";
	import { parseDate } from "chrono-node";
	import { untrack } from "svelte";

	function pad2(n: number) {
		return `${n}`.padStart(2, "0");
	}

	function formatDateTime(date: DateValue | undefined, boundary: "start" | "end") {
		if (!date) return "";
		const time = boundary === "start" ? "00:00:00" : "23:59:59";
		return `${date.year}-${pad2(date.month)}-${pad2(date.day)} ${time}`;
	}

	function formatRange(range: DateRange | undefined) {
		if (!range?.start || !range?.end) return "";
		return `from ${formatDateTime(range.start, "start")} to ${formatDateTime(range.end, "end")}`;
	}

	function parseNaturalDate(value: string) {
		const date = parseDate(value);
		if (!date || Number.isNaN(date.getTime())) return undefined;
		return new CalendarDate(date.getFullYear(), date.getMonth() + 1, date.getDate());
	}

	const id = $props.id();

	let open = $state(false);
	let fromInputValue = $state("Today");
	let toInputValue = $state("In 7 days");
	let value = $state<DateRange | undefined>(
		untrack(() => {
			const start = parseNaturalDate(fromInputValue);
			const end = parseNaturalDate(toInputValue);
			if (!start && !end) return undefined;
			return { start, end } as DateRange;
		})
	);

	function handleRangeChange(v: DateRange | undefined) {
		if (!v) {
			fromInputValue = "";
			toInputValue = "";
			return;
		}

		if (v.start) fromInputValue = formatDateTime(v.start, "start");
		if (v.end) toInputValue = formatDateTime(v.end, "end");
		else toInputValue = "";

		if (v.start && v.end) open = false;
	}
</script>

<div class="flex flex-col gap-4">
	<Popover.Root bind:open>
		<div class="grid gap-4 sm:grid-cols-2">
			<div class="flex flex-col gap-3">
				<Label for="{id}-from" class="px-1">From</Label>
				<div class="relative flex gap-2">
					<Input
						id="{id}-from"
						bind:value={
							() => fromInputValue,
							(v) => {
								fromInputValue = v;
								if (!v.trim()) {
									value = value?.end
										? ({ start: undefined, end: value.end } as DateRange)
										: undefined;
									return;
								}

								const start = parseNaturalDate(v);
								if (start) value = { start, end: value?.end } as DateRange;
							}
						}
						placeholder="Today or next week"
						class="bg-background pe-10"
						onkeydown={(e) => {
							if (e.key === "ArrowDown") {
								e.preventDefault();
								open = true;
							}
						}}
					/>
					<Popover.Trigger id="{id}-date-range-picker-from">
						{#snippet child({ props })}
							<Button
								{...props}
								variant="ghost"
								class="absolute end-2 top-1/2 size-6 -translate-y-1/2"
							>
								<CalendarIcon class="size-3.5" />
								<span class="sr-only">Select date range</span>
							</Button>
						{/snippet}
					</Popover.Trigger>
				</div>
			</div>

			<div class="flex flex-col gap-3">
				<Label for="{id}-to" class="px-1">To</Label>
				<div class="relative flex gap-2">
					<Input
						id="{id}-to"
						bind:value={
							() => toInputValue,
							(v) => {
								toInputValue = v;
								if (!v.trim()) {
									value = value?.start
										? ({ start: value.start, end: undefined } as DateRange)
										: undefined;
									return;
								}

								const end = parseNaturalDate(v);
								if (end) value = { start: value?.start, end } as DateRange;
							}
						}
						placeholder="In 7 days or next Friday"
						class="bg-background pe-10"
						onkeydown={(e) => {
							if (e.key === "ArrowDown") {
								e.preventDefault();
								open = true;
							}
						}}
					/>
					<Popover.Trigger id="{id}-date-range-picker-to">
						{#snippet child({ props })}
							<Button
								{...props}
								variant="ghost"
								class="absolute end-2 top-1/2 size-6 -translate-y-1/2"
							>
								<CalendarIcon class="size-3.5" />
								<span class="sr-only">Select date range</span>
							</Button>
						{/snippet}
					</Popover.Trigger>
				</div>
			</div>
		</div>

		<Popover.Content class="w-auto overflow-hidden p-0" align="end">
			<RangeCalendar bind:value captionLayout="dropdown" onValueChange={handleRangeChange} />
		</Popover.Content>
	</Popover.Root>

	<div class="text-muted-foreground px-1 text-sm">
		{#if value?.start && value?.end}
			Selected range:
			<span class="font-medium">{formatRange(value)}</span>
		{:else}
			Pick a date range.
		{/if}
	</div>
</div>
