<script lang="ts">
	import * as NavigationMenu from '$lib/components/ui/navigation-menu/index.js';
	import * as Sheet from '$lib/components/ui/sheet';
	import Button from '$lib/components/ui/button/button.svelte';
	import { authStore } from '$lib/stores/auth.store';
	import { goto } from '$app/navigation';
	import { page } from '$app/state';
	import ThemeSwitcher from '$lib/components/custom/ThemeSwitcher.svelte';
	import { t } from '$lib/translations';
	import { Menu } from 'lucide-svelte';
	const navItems: {
		href?: string;
		label: string;
		subMenu?: {
			href: string;
			label: string;
		}[];
	}[] = [
		{ href: '/dashboard', label: $t('app.layout.dashboard') },
		{ href: '/dashboard/ingestions', label: $t('app.layout.ingestions') },
		{ href: '/dashboard/archived-emails', label: $t('app.layout.archived_emails') },
		{ href: '/dashboard/search', label: $t('app.layout.search') },
		{
			label: $t('app.layout.settings'),
			subMenu: [
				{
					href: '/dashboard/settings/system',
					label: $t('app.layout.system'),
				},
				{
					href: '/dashboard/settings/users',
					label: $t('app.layout.users'),
				},
				{
					href: '/dashboard/settings/roles',
					label: $t('app.layout.roles'),
				},
				{
					href: '/dashboard/settings/api-keys',
					label: $t('app.layout.api_keys'),
				},
			],
		},
	];
	let { children } = $props();
	
	let mobileMenuOpen = $state(false);
	
	function handleLogout() {
		authStore.logout();
		goto('/signin');
	}
</script>

<!-- Mobile Navigation Sheet -->
<Sheet.Root bind:open={mobileMenuOpen}>
	<Sheet.Content side="left" class="w-[280px] overflow-y-auto">
		<Sheet.Header>
			<Sheet.Title>Menu</Sheet.Title>
		</Sheet.Header>
		<nav class="mt-6 flex flex-col gap-2">
			{#each navItems as item}
				{#if item.subMenu && item.subMenu.length > 0}
					<div class="flex flex-col gap-1">
						<div class="px-3 py-2 text-sm font-semibold">{item.label}</div>
						{#each item.subMenu as subItem}
							<a
								href={subItem.href}
								class="px-6 py-2 text-sm hover:bg-accent rounded-md transition-colors"
								class:bg-accent={page.url.pathname === subItem.href}
								onclick={() => (mobileMenuOpen = false)}
							>
								{subItem.label}
							</a>
						{/each}
					</div>
				{:else if item.href}
					<a
						href={item.href}
						class="px-3 py-2 text-sm hover:bg-accent rounded-md transition-colors"
						class:bg-accent={page.url.pathname === item.href}
						onclick={() => (mobileMenuOpen = false)}
					>
						{item.label}
					</a>
				{/if}
			{/each}
			<div class="mt-4 border-t pt-4">
				<Button onclick={handleLogout} variant="outline" class="w-full">
					{$t('app.layout.logout')}
				</Button>
			</div>
		</nav>
	</Sheet.Content>
</Sheet.Root>

<header class="bg-background sticky top-0 z-40 border-b">
	<div class="container mx-auto flex h-16 flex-row items-center justify-between px-4">
		<div class="flex items-center gap-2">
			<!-- Mobile menu button -->
			<Button
				variant="ghost"
				size="icon"
				class="lg:hidden"
				onclick={() => (mobileMenuOpen = true)}
			>
				<Menu class="h-5 w-5" />
				<span class="sr-only">Toggle menu</span>
			</Button>
			<a href="/dashboard" class="flex flex-row items-center gap-2 font-bold">
				<img src="/logos/logo-sq.svg" alt="OpenArchiver Logo" class="h-8 w-8" />
				<span class="hidden sm:inline">Open Archiver</span>
			</a>
		</div>
		
		<!-- Desktop Navigation - hidden on mobile -->
		<NavigationMenu.Root viewport={false} class="hidden lg:flex">
			<NavigationMenu.List class="flex items-center space-x-4">
				{#each navItems as item}
					{#if item.subMenu && item.subMenu.length > 0}
						<NavigationMenu.Item
							class={item.subMenu.some((sub) =>
								page.url.pathname.startsWith(
									sub.href.substring(0, sub.href.lastIndexOf('/'))
								)
							)
								? 'bg-accent rounded-md'
								: ''}
						>
							<NavigationMenu.Trigger class="cursor-pointer font-normal">
								{item.label}
							</NavigationMenu.Trigger>
							<NavigationMenu.Content>
								<ul class="grid w-fit min-w-28 gap-1 p-1">
									{#each item.subMenu as subItem}
										<li>
											<NavigationMenu.Link href={subItem.href}>
												{subItem.label}
											</NavigationMenu.Link>
										</li>
									{/each}
								</ul>
							</NavigationMenu.Content>
						</NavigationMenu.Item>
					{:else if item.href}
						<NavigationMenu.Item
							class={page.url.pathname === item.href ? 'bg-accent rounded-md' : ''}
						>
							<NavigationMenu.Link href={item.href}>
								{item.label}
							</NavigationMenu.Link>
						</NavigationMenu.Item>
					{/if}
				{/each}
			</NavigationMenu.List>
		</NavigationMenu.Root>
		
		<div class="flex items-center gap-2 sm:gap-4">
			<ThemeSwitcher />
			<Button onclick={handleLogout} variant="outline" size="sm" class="hidden lg:flex">
				{$t('app.layout.logout')}
			</Button>
		</div>
	</div>
</header>

<main class="container mx-auto my-4 sm:my-6 md:my-10 px-4 sm:px-6">
	{@render children()}
</main>
