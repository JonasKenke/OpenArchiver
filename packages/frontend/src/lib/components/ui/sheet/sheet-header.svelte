<script lang="ts">
	import { cn } from "$lib/utils.js";
	import type { HTMLAttributes } from "svelte/elements";
	import type { Snippet } from "svelte";

	let {
		class: className,
		children,
		...restProps
	}: HTMLAttributes<HTMLDivElement> & { children?: Snippet } = $props();
</script>

<div
	data-slot="sheet-header"
	class={cn("flex flex-col gap-y-2 text-center sm:text-left", className)}
	{...restProps}
>
	{@render children?.()}
</div>
