<script lang="ts">
	import type { PageData } from './$types';
	import * as Table from '$lib/components/ui/table';
	import { Button } from '$lib/components/ui/button';
	import * as Select from '$lib/components/ui/select';
	import * as Sheet from '$lib/components/ui/sheet';
	import { goto } from '$app/navigation';
	import { page } from '$app/state';
	import { t } from '$lib/translations';
	import { api } from '$lib/api.client';
	import EmailFolderTree from '$lib/components/EmailFolderTree.svelte';
	import type { EmailFolder, ArchivedEmail } from '@open-archiver/types';
	import { onMount } from 'svelte';
	import { Menu } from 'lucide-svelte';

	let { data }: { data: PageData } = $props();

	let ingestionSources = $derived(data.ingestionSources);
	let selectedIngestionSourceId = $state(data.selectedIngestionSourceId ?? null);

	let folders = $state<EmailFolder[]>(data.folders || []);
	let archivedEmails = $state<{
		items: ArchivedEmail[];
		total: number;
		page: number;
		limit: number;
	}>(data.archivedEmails);
	let loadingFolders = $state(false);
	let loadingEmails = $state(false);

	// Sorting states
	let sortBy = $state<'sentAt' | 'senderEmail' | 'subject'>('sentAt');
	let sortOrder = $state<'asc' | 'desc'>('desc');

	const handleSourceChange = (value: string | undefined) => {
		if (value) {
			selectedPath = null; // Reset path on source change
			goto(`/dashboard/archived-emails?ingestionSourceId=${value}`);
		}
	};

	// New states for folder filtering
	let selectedPath = $state<string | null>(null);
	
	// Mobile sidebar state
	let mobileSheetOpen = $state(false);

	// New function to load folders
	async function loadFolders(sourceId: string) {
		loadingFolders = true;
		try {
			const response = await api(`/archived-emails/ingestion-source/${sourceId}/folders`);
			if (response.ok) {
				folders = await response.json();
			} else {
				folders = [];
			}
		} catch (error) {
			console.error('Failed to load folders:', error);
			folders = [];
		} finally {
			loadingFolders = false;
		}
	}

	// New function to load emails (defaults to all emails when selectedPath is null)
	async function loadEmails(page = 1) {
		if (!selectedIngestionSourceId) return;
		loadingEmails = true;
		try {
			const params = new URLSearchParams({
				page: page.toString(),
				limit: archivedEmails.limit.toString(),
				sortBy,
				sortOrder,
			});
			// Only add path parameter if a folder is explicitly selected
			if (selectedPath !== null) {
				params.append('path', selectedPath);
			}
			// If selectedPath is null, don't send path parameter to get all emails
			const response = await api(
				`/archived-emails/ingestion-source/${selectedIngestionSourceId}?${params}`
			);
			if (response.ok) {
				const responseData = await response.json();
				archivedEmails = {
					...archivedEmails,
					items: responseData.items,
					total: responseData.total,
					page: page,
				};
			}
		} catch (error) {
			console.error('Failed to load emails:', error);
		} finally {
			loadingEmails = false;
		}
	}

	// New function to handle folder selection
	function handleFolderSelect(path: string | null) {
		selectedPath = path;
		archivedEmails.page = 1; // Reset page on folder change
		loadEmails(1);
	}

	// Sorting toggle function
	function toggleSort(field: typeof sortBy) {
		if (sortBy === field) {
			sortOrder = sortOrder === 'asc' ? 'desc' : 'asc';
		} else {
			sortBy = field;
			sortOrder = 'desc';
		}
		archivedEmails.page = 1; // Reset page on sort
		loadEmails(1);
	}

	const getPaginationItems = (currentPage: number, totalPages: number, siblingCount = 1) => {
		const totalPageNumbers = siblingCount + 5;

		if (totalPages <= totalPageNumbers) {
			return Array.from({ length: totalPages }, (_, i) => i + 1);
		}

		const leftSiblingIndex = Math.max(currentPage - siblingCount, 1);
		const rightSiblingIndex = Math.min(currentPage + siblingCount, totalPages);

		const shouldShowLeftDots = leftSiblingIndex > 2;
		const shouldShowRightDots = rightSiblingIndex < totalPages - 2;

		const firstPageIndex = 1;
		const lastPageIndex = totalPages;

		if (!shouldShowLeftDots && shouldShowRightDots) {
			let leftItemCount = 3 + 2 * siblingCount;
			let leftRange = Array.from({ length: leftItemCount }, (_, i) => i + 1);
			return [...leftRange, '...', totalPages];
		}

		if (shouldShowLeftDots && !shouldShowRightDots) {
			let rightItemCount = 3 + 2 * siblingCount;
			let rightRange = Array.from(
				{ length: rightItemCount },
				(_, i) => totalPages - rightItemCount + i + 1
			);
			return [firstPageIndex, '...', ...rightRange];
		}

		if (shouldShowLeftDots && shouldShowRightDots) {
			let middleRange = Array.from(
				{ length: rightSiblingIndex - leftSiblingIndex + 1 },
				(_, i) => leftSiblingIndex + i
			);
			return [firstPageIndex, '...', ...middleRange, '...', lastPageIndex];
		}

		return [];
	};

	let paginationItems = $derived(
		getPaginationItems(
			archivedEmails.page,
			Math.ceil(archivedEmails.total / archivedEmails.limit)
		)
	);

	onMount(() => {
		if (selectedIngestionSourceId && folders.length === 0) {
			loadFolders(selectedIngestionSourceId);
		}
	});

	$effect(() => {
		const urlSource = page.url.searchParams.get('ingestionSourceId');
		const desiredSource = urlSource ?? data.selectedIngestionSourceId ?? null;

		if (desiredSource !== selectedIngestionSourceId) {
			selectedIngestionSourceId = desiredSource;
			if (desiredSource) {
				selectedPath = null; // Reset path on source change
				folders = [];
				archivedEmails = { ...archivedEmails, items: [], total: 0, page: 1 };
				loadFolders(desiredSource);
				loadEmails(1);
			} else {
				// Clear data if no source
				folders = [];
				archivedEmails = { ...archivedEmails, items: [], total: 0, page: 1 };
			}
		}
	});
</script>

<svelte:head>
	<title>{$t('app.archived_emails_page.title')} - OpenArchiver</title>
</svelte:head>

<!-- Mobile Sheet for folders -->
<Sheet.Root bind:open={mobileSheetOpen}>
	<Sheet.Content side="left" class="w-[280px] sm:w-[320px] overflow-y-auto">
		<Sheet.Header>
			<Sheet.Title>{$t('app.archived_emails_page.folders')}</Sheet.Title>
		</Sheet.Header>
		<div class="mt-4">
			{#if loadingFolders}
				<p>{$t('app.archived_emails_page.loading_folders')}</p>
			{:else if folders.length > 0}
				<EmailFolderTree 
					{folders} 
					bind:selectedPath 
					onSelectFolder={(path) => {
						handleFolderSelect(path);
						mobileSheetOpen = false;
					}} 
				/>
			{:else}
				<p>{$t('app.archived_emails_page.no_folders_available')}</p>
			{/if}
		</div>
	</Sheet.Content>
</Sheet.Root>

<!-- Updated layout: Responsive grid for sidebar and table -->
<div class="container mx-auto p-4 md:p-8">
	<div class="md:grid md:grid-cols-4 md:gap-6">
		<!-- Desktop sidebar for folders - hidden on mobile -->
		<aside class="hidden md:block md:col-span-1 max-h-[calc(100vh-200px)] overflow-y-auto">
			<h2 class="mb-4 text-lg font-semibold">{$t('app.archived_emails_page.folders')}</h2>
			{#if loadingFolders}
				<p>{$t('app.archived_emails_page.loading_folders')}</p>
			{:else if folders.length > 0}
				<EmailFolderTree {folders} bind:selectedPath onSelectFolder={handleFolderSelect} />
			{:else}
				<p>{$t('app.archived_emails_page.no_folders_available')}</p>
			{/if}
		</aside>

		<!-- Main content section -->
		<main class="md:col-span-3">
		<!-- Header with mobile menu button -->
		<div class="mb-4 flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
			<div class="flex items-center gap-2">
				<!-- Mobile menu button - only visible on small screens -->
				<Button
					variant="outline"
					size="icon"
					class="md:hidden"
					onclick={() => (mobileSheetOpen = true)}
				>
					<Menu class="h-5 w-5" />
					<span class="sr-only">Toggle folder menu</span>
				</Button>
				<h1 class="text-xl font-bold sm:text-2xl">{$t('app.archived_emails_page.header')}</h1>
			</div>
			{#if ingestionSources.length > 0}
				<div class="w-full sm:w-[250px]">
					<Select.Root
						type="single"
						onValueChange={handleSourceChange}
						value={selectedIngestionSourceId ?? undefined}
					>
						<Select.Trigger class="w-full">
							<span
								>{selectedIngestionSourceId
									? ingestionSources.find(
											(s) => s.id === selectedIngestionSourceId
										)?.name
									: $t('app.archived_emails_page.select_ingestion_source')}</span
							>
						</Select.Trigger>
						<Select.Content>
							{#each ingestionSources as source}
								<Select.Item value={source.id}>{source.name}</Select.Item>
							{/each}
						</Select.Content>
					</Select.Root>
				</div>
			{/if}
		</div>

		<!-- Table with horizontal scroll on mobile -->
		<div class="rounded-md border overflow-x-auto">
			<Table.Root>
				<Table.Header>
					<Table.Row>
						<Table.Head class="whitespace-nowrap">
							<button
								onclick={() => toggleSort('sentAt')}
								class="flex items-center gap-1 hover:underline"
							>
								{$t('app.archived_emails_page.date')}
								{sortBy === 'sentAt' ? (sortOrder === 'asc' ? '↑' : '↓') : ''}
							</button>
						</Table.Head>
						<Table.Head class="whitespace-nowrap">{$t('app.archived_emails_page.subject')}</Table.Head>
						<Table.Head class="whitespace-nowrap hidden sm:table-cell">{$t('app.archived_emails_page.sender')}</Table.Head>
						<Table.Head class="whitespace-nowrap hidden md:table-cell">{$t('app.archived_emails_page.inbox')}</Table.Head>
						<Table.Head class="whitespace-nowrap hidden lg:table-cell">{$t('app.archived_emails_page.path')}</Table.Head>
						<Table.Head class="text-right whitespace-nowrap"
							>{$t('app.archived_emails_page.actions')}</Table.Head
						>
					</Table.Row>
				</Table.Header>
				<Table.Body class="text-sm">
					{#if loadingEmails}
						<Table.Row>
							<Table.Cell colspan={6} class="text-center"
								>Loading emails...</Table.Cell
							>
						</Table.Row>
					{:else if archivedEmails.items.length > 0}
						{#each archivedEmails.items as email (email.id)}
							<Table.Row>
								<Table.Cell class="whitespace-nowrap text-xs sm:text-sm">
									<span class="hidden sm:inline">{new Date(email.sentAt).toLocaleString()}</span>
									<span class="sm:hidden">{new Date(email.sentAt).toLocaleDateString()}</span>
								</Table.Cell>

								<Table.Cell class="min-w-[150px] sm:min-w-[200px]">
									<div class="max-w-[150px] sm:max-w-[300px] truncate">
										<a
											class="link text-sm"
											href={`/dashboard/archived-emails/${email.id}`}
										>
											{email.subject}
										</a>
									</div>
									<!-- Show sender on mobile as subtitle -->
									<div class="text-xs text-muted-foreground mt-1 sm:hidden truncate">
										{email.senderEmail || email.senderName}
									</div>
								</Table.Cell>
								<Table.Cell class="hidden sm:table-cell text-sm">
									{email.senderEmail || email.senderName}
								</Table.Cell>
								<Table.Cell class="hidden md:table-cell text-sm">{email.userEmail}</Table.Cell>
								<Table.Cell class="hidden lg:table-cell">
									{#if email.path}
										<span class="bg-muted truncate rounded p-1.5 text-xs"
											>{email.path}</span
										>
									{:else}
										<span class="text-muted-foreground text-xs"
											>All folders</span
										>
									{/if}
								</Table.Cell>
								<Table.Cell class="text-right">
									<a href={`/dashboard/archived-emails/${email.id}`}>
										<Button variant="outline" size="sm"
											>{$t('app.archived_emails_page.view')}</Button
										>
									</a>
								</Table.Cell>
							</Table.Row>
						{/each}
					{:else}
						<Table.Row>
							<Table.Cell colspan={6} class="text-center"
								>{$t('app.archived_emails_page.no_emails_found')}</Table.Cell
							>
						</Table.Row>
					{/if}
				</Table.Body>
			</Table.Root>
		</div>

		{#if archivedEmails.total > archivedEmails.limit}
			<div class="mt-8 flex flex-row flex-wrap items-center justify-center gap-2">
				<Button
					variant="outline"
					size="sm"
					disabled={archivedEmails.page === 1 || loadingEmails}
					onclick={() => loadEmails(archivedEmails.page - 1)}
				>
					{$t('app.archived_emails_page.prev')}
				</Button>

				<div class="flex flex-row flex-wrap items-center justify-center gap-2">
					{#each paginationItems as item}
						{#if typeof item === 'number'}
							<Button
								variant={item === archivedEmails.page ? 'default' : 'outline'}
								size="sm"
								disabled={loadingEmails}
								onclick={() => loadEmails(item)}
							>
								{item}
							</Button>
						{:else}
							<span class="px-2 py-1 text-sm">...</span>
						{/if}
					{/each}
				</div>

				<Button
					variant="outline"
					size="sm"
					disabled={archivedEmails.page * archivedEmails.limit >= archivedEmails.total ||
						loadingEmails}
					onclick={() => loadEmails(archivedEmails.page + 1)}
				>
					{$t('app.archived_emails_page.next')}
				</Button>
			</div>
		{/if}
	</main>
	</div>
</div>
