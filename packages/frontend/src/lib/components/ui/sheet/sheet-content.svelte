<script lang="ts">
	import { Dialog as SheetPrimitive } from "bits-ui";
	import XIcon from "@lucide/svelte/icons/x";
	import type { Snippet } from "svelte";
	import * as Sheet from "./index.js";
	import { cn, type WithoutChildrenOrChild } from "$lib/utils.js";

	let {
		ref = $bindable(null),
		class: className,
		side = "left",
		portalProps,
		children,
		showCloseButton = true,
		...restProps
	}: WithoutChildrenOrChild<SheetPrimitive.ContentProps> & {
		portalProps?: SheetPrimitive.PortalProps;
		children: Snippet;
		side?: "left" | "right" | "top" | "bottom";
		showCloseButton?: boolean;
	} = $props();

	const sideClasses = {
		left: "data-[state=closed]:slide-out-to-left data-[state=open]:slide-in-from-left inset-y-0 left-0 h-full w-3/4 border-r sm:max-w-sm",
		right: "data-[state=closed]:slide-out-to-right data-[state=open]:slide-in-from-right inset-y-0 right-0 h-full w-3/4 border-l sm:max-w-sm",
		top: "data-[state=closed]:slide-out-to-top data-[state=open]:slide-in-from-top inset-x-0 top-0 border-b",
		bottom: "data-[state=closed]:slide-out-to-bottom data-[state=open]:slide-in-from-bottom inset-x-0 bottom-0 border-t",
	};
</script>

<Sheet.Portal {...portalProps}>
	<Sheet.Overlay />
	<SheetPrimitive.Content
		bind:ref
		data-slot="sheet-content"
		class={cn(
			"bg-background data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 fixed z-50 gap-4 p-6 shadow-lg duration-300",
			sideClasses[side],
			className
		)}
		{...restProps}
	>
		{@render children?.()}
		{#if showCloseButton}
			<SheetPrimitive.Close
				class="ring-offset-background focus:ring-ring rounded-xs focus:outline-hidden absolute right-4 top-4 opacity-70 transition-opacity hover:opacity-100 focus:ring-2 focus:ring-offset-2 disabled:pointer-events-none [&_svg:not([class*='size-'])]:size-4 [&_svg]:pointer-events-none [&_svg]:shrink-0"
			>
				<XIcon />
				<span class="sr-only">Close</span>
			</SheetPrimitive.Close>
		{/if}
	</SheetPrimitive.Content>
</Sheet.Portal>
