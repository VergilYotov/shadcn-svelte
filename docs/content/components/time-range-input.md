---
title: Time Range Input
description: A time range input component with natural language parsing and calendar picker.
component: true
---

<script>
	import ComponentPreview from "$lib/components/component-preview.svelte";
	import Callout from "$lib/components/callout.svelte";
	import PMAddComp from "$lib/components/pm-add-comp.svelte";
</script>

<ComponentPreview name="time-range-input-demo">

<div></div>

</ComponentPreview>

## Installation

<PMAddComp name="time-range-input" />

## About

The `<TimeRangeInput />` component combines natural language date parsing with a shared range calendar picker. It provides two input fields (from/to) that:

- Accept natural language input (e.g., "Today", "next Monday", "in 3 days")
- Open a shared range calendar when either calendar icon is clicked
- Format dates as "YYYY-MM-DD 00:00:00" for start and "YYYY-MM-DD 23:59:59" for end

This component uses the `chrono-node` library for natural language date parsing and `@internationalized/date` for date handling.

## Usage

```svelte showLineNumbers title="lib/components/example-time-range-input.svelte"
<script lang="ts">
	import TimeRangeInput from "$lib/components/blocks/time-range-input.svelte";

	let fromValue = $state("");
	let toValue = $state("");
</script>

<TimeRangeInput
	id="my-range"
	bind:fromValue
	bind:toValue
/>

{#if fromValue && toValue}
	<p>
		Selected range: {fromValue} to {toValue}
	</p>
{/if}
```

## API

### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `id` | `string` | **required** | Unique identifier for the component |
| `fromValue` | `string` | `""` | The from date value (format: "YYYY-MM-DD 00:00:00") |
| `toValue` | `string` | `""` | The to date value (format: "YYYY-MM-DD 23:59:59") |

### Examples

#### With Initial Values

<ComponentPreview name="time-range-input-demo">

<div></div>

</ComponentPreview>

#### Controlled Component

```svelte showLineNumbers title="lib/components/example-controlled.svelte"
<script lang="ts">
	import TimeRangeInput from "$lib/components/blocks/time-range-input.svelte";

	let fromValue = $state("2026-01-01 00:00:00");
	let toValue = $state("2026-01-31 23:59:59");

	function handleSubmit() {
		console.log("Exporting from", fromValue, "to", toValue);
	}
</script>

<TimeRangeInput
	id="controlled-range"
	bind:fromValue
	bind:toValue
/>

<button onclick={handleSubmit}>Submit Range</button>
```

<Callout type="info" title="Natural Language Parsing">

The component uses `chrono-node` to parse natural language dates. Try typing:
- "Today" or "tomorrow"
- "Next Monday" or "last Friday"
- "In 3 days" or "2 weeks ago"
- Any standard date format like "2026-01-15"

</Callout>

## Accessibility

- Keyboard navigation: Press `ArrowDown` in either input to open the calendar
- Calendar icons have screen reader labels
- Proper form labels are associated with each input
- All interactive elements are keyboard accessible
