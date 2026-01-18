<script lang="ts">
	import { Label } from "$lib/registry/ui/label/index.js";
	import * as Popover from "$lib/registry/ui/popover/index.js";
	import { Button } from "$lib/registry/ui/button/index.js";
	import RangeCalendar from "$lib/registry/ui/range-calendar/range-calendar.svelte";
	import { Input } from "$lib/registry/ui/input/index.js";
	import CalendarIcon from "@lucide/svelte/icons/calendar";
	import { parseDate } from "chrono-node";
	import { CalendarDate, getLocalTimeZone, type DateValue } from "@internationalized/date";
	import type { DateRange } from "bits-ui";

	let { id, fromValue = $bindable<string>(""), toValue = $bindable<string>("") } = $props();

	let open = $state(false);
	let rangeValue = $state<DateRange | undefined>();

	// Raw input values (what user types)
	let fromInput = $state("");
	let toInput = $state("");

	/**
	 * Formats a Date to "YYYY-MM-DD HH:MM:SS" format
	 */
	function formatDateWithTime(date: Date): string {
		const year = date.getFullYear();
		const month = String(date.getMonth() + 1).padStart(2, "0");
		const day = String(date.getDate()).padStart(2, "0");
		const hours = String(date.getHours()).padStart(2, "0");
		const minutes = String(date.getMinutes()).padStart(2, "0");
		const seconds = String(date.getSeconds()).padStart(2, "0");
		return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
	}

	/**
	 * Formats a DateValue to the required output format with time component
	 */
	function formatDateTime(date: DateValue, isStart: boolean): string {
		const year = date.year;
		const month = String(date.month).padStart(2, "0");
		const day = String(date.day).padStart(2, "0");
		const time = isStart ? "00:00:00" : "23:59:59";
		return `${year}-${month}-${day} ${time}`;
	}

	/**
	 * Parses a date string and returns a CalendarDate and optional time string
	 */
	function parseInputToDate(input: string): { date: CalendarDate; time?: string } | undefined {
		if (!input.trim()) return undefined;

		// Try parsing as natural language first - this preserves the current time
		const parsedDate = parseDate(input, new Date(), { forwardDate: true });
		if (parsedDate) {
			return {
				date: new CalendarDate(
					parsedDate.getFullYear(),
					parsedDate.getMonth() + 1,
					parsedDate.getDate()
				),
				// Preserve the current time for natural language inputs
				time: formatDateWithTime(parsedDate).split(" ")[1]
			};
		}

		// Try parsing as "YYYY-MM-DD HH:MM:SS" format
		const dateTimeMatch = input.match(/^(\d{4})-(\d{2})-(\d{2})(?:\s+(\d{2}:\d{2}:\d{2}))?/);
		if (dateTimeMatch) {
			const [, year, month, day, time] = dateTimeMatch;
			return {
				date: new CalendarDate(parseInt(year), parseInt(month), parseInt(day)),
				time
			};
		}

		return undefined;
	}

	/**
	 * Handles range calendar selection
	 */
	function handleRangeChange(range: DateRange | undefined) {
		rangeValue = range;
		if (range?.start && range?.end) {
			fromValue = formatDateTime(range.start, true);
			toValue = formatDateTime(range.end, false);
			// Update raw inputs to match formatted values
			fromInput = fromValue;
			toInput = toValue;
			open = false;
		}
	}

	// Auto-set range when inputs change (like calendar-29 does)
	$effect(() => {
		const fromParsed = parseInputToDate(fromInput);
		const toParsed = parseInputToDate(toInput);

		if (fromParsed || toParsed) {
			if (fromParsed) {
				fromValue = fromParsed.time
					? `${fromParsed.date.year}-${String(fromParsed.date.month).padStart(2, "0")}-${String(fromParsed.date.day).padStart(2, "0")} ${fromParsed.time}`
					: formatDateTime(fromParsed.date, true);
			} else {
				fromValue = fromInput;
			}
			if (toParsed) {
				toValue = toParsed.time
					? `${toParsed.date.year}-${String(toParsed.date.month).padStart(2, "0")}-${String(toParsed.date.day).padStart(2, "0")} ${toParsed.time}`
					: formatDateTime(toParsed.date, false);
			} else {
				toValue = toInput;
			}

			// Update calendar range
			rangeValue = { start: fromParsed?.date, end: toParsed?.date };
		}
	});

	// Initialize from bound values
	$effect(() => {
		if (fromValue && !fromInput) {
			fromInput = fromValue;
		}
		if (toValue && !toInput) {
			toInput = toValue;
		}
	});
</script>

<div class="flex flex-col gap-3">
	<!-- From Date Input -->
	<div class="flex flex-col gap-2">
		<Label for="{id}-from" class="px-1">From Date</Label>
		<div class="relative flex gap-2">
			<Input
				id="{id}-from"
				bind:value={fromInput}
				placeholder="e.g., today 00:00, -5d, last week"
				class="bg-background pe-10"
				onkeydown={(e) => {
					if (e.key === "ArrowDown") {
						e.preventDefault();
						open = true;
					}
				}}
			/>
			<Popover.Root bind:open>
				<Popover.Trigger id="{id}-from-calendar">
					{#snippet child({ props })}
						<Button
							{...props}
							variant="ghost"
							class="absolute end-2 top-1/2 size-6 -translate-y-1/2"
						>
							<CalendarIcon class="size-3.5" />
							<span class="sr-only">Select date</span>
						</Button>
					{/snippet}
				</Popover.Trigger>
				<Popover.Content
					class="w-auto overflow-hidden p-0"
					align="end"
					alignOffset={-8}
					sideOffset={10}
				>
					<RangeCalendar
						bind:value={rangeValue}
						onValueChange={handleRangeChange}
						captionLayout="dropdown"
					/>
				</Popover.Content>
			</Popover.Root>
		</div>
	</div>

	<!-- To Date Input -->
	<div class="flex flex-col gap-2">
		<Label for="{id}-to" class="px-1">To Date</Label>
		<div class="relative flex gap-2">
			<Input
				id="{id}-to"
				bind:value={toInput}
				placeholder="e.g., now, today 23:59, next friday"
				class="bg-background pe-10"
				onkeydown={(e) => {
					if (e.key === "ArrowDown") {
						e.preventDefault();
						open = true;
					}
				}}
			/>
			<!-- Calendar icon that opens the same calendar -->
			<Button
				variant="ghost"
				class="absolute end-2 top-1/2 size-6 -translate-y-1/2"
				onclick={() => (open = true)}
				aria-label="Select to date"
			>
				<CalendarIcon class="size-3.5" />
				<span class="sr-only">Select date</span>
			</Button>
		</div>
	</div>
</div>
