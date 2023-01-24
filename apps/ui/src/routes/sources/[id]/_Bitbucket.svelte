<script lang="ts">
	export let source: any;
	export let settings: any;
	import { page } from '$app/stores';
	import { getAPIUrl, post } from '$lib/api';
	import { dev } from '$app/env';

	import { t } from '$lib/translations';
	import { errorNotification } from '$lib/common';
	import { addToast, appSession } from '$lib/store';
	import Explainer from '$lib/components/Explainer.svelte';
	import CopyPasswordField from '$lib/components/CopyPasswordField.svelte';
	const { id } = $page.params;

	$: selfHosted = source.htmlUrl !== 'https://bitbucket.org';
	let url = settings.fqdn ? settings.fqdn : window.location.origin;
	if (dev) url = getAPIUrl();
	let loading = false;
	if (!source.bitbucketAppId) {
		source.bitbucketApp = {
			consumerWorkspaceName: null,
			consumerKey: null,
			consumerSecret: null
		};
	}

	async function handleSubmit() {
		if (loading) return;
		loading = true;
		if (!source.bitbucketAppId) {
			try {
				const { id } = await post(`/sources/new/bitbucket`, {
					type: 'bitbucket',
					name: source.name,
					htmlUrl: source.htmlUrl.replace(/\/$/, ''),
					apiUrl: source.apiUrl.replace(/\/$/, ''),
					customPort: source.customPort,
					customUser: source.customUser,
					consumerWorkspaceName: source.bitbucketApp.consumerWorkspaceName,
					consumerKey: source.bitbucketApp.consumerKey,
					consumerSecret: source.bitbucketApp.consumerSecret
				});
				const from = $page.url.searchParams.get('from');
				if (from) {
					return window.location.assign(from);
				}
				return window.location.assign(`/sources/${id}`);
			} catch (error) {
				return errorNotification(error);
			} finally {
				loading = false;
			}
		} else {
			try {
				await post(`/sources/${id}`, {
					name: source.name,
					htmlUrl: source.htmlUrl.replace(/\/$/, ''),
					apiUrl: source.apiUrl.replace(/\/$/, ''),
					customPort: source.customPort,
					customUser: source.customUser,
					consumerWorkspaceName: source.bitbucketApp.consumerWorkspaceName,
					consumerKey: source.bitbucketApp.consumerKey,
					consumerSecret: source.bitbucketApp.consumerSecret
				});
				return addToast({
					message: 'Configuration saved.',
					type: 'success'
				});
			} catch (error) {
				return errorNotification(error);
			} finally {
				loading = false;
			}
		}
	}

	async function changeSettings() {
		const {
			htmlUrl,
			bitbucketApp: { consumerWorkspaceName }
		} = source;
		const left = screen.width / 2 - 1020 / 2;
		const top = screen.height / 2 - 1000 / 2;
		const newWindow = open(
			`${htmlUrl}/${consumerWorkspaceName}/workspace/settings/api`,
			'Bitbucket',
			'resizable=1, scrollbars=1, fullscreen=0, height=1000, width=1020,top=' +
				top +
				', left=' +
				left +
				', toolbar=0, menubar=0, status=0'
		);
		const timer = setInterval(() => {
			if (newWindow?.closed) {
				clearInterval(timer);
			}
		}, 100);
	}
	function newOAuthConsumer() {
		if (!source.workspaceName || !source.htmlUrl) {
			return addToast({
				message: 'Workspace Name missing.',
				type: 'error'
			});
		}
		if (!source.htmlUrl) {
			return addToast({
				message: 'HTML URL missing.',
				type: 'error'
			});
		}
		window.open(`${source.htmlUrl}/${source.workspaceName}/workspace/settings/api`);
	}
</script>

<div class="mx-auto max-w-6xl px-6">
	<form on:submit|preventDefault={handleSubmit} class="py-4">
		<div class="flex lg:flex-row lg:justify-between flex-col space-y-3 w-full lg:items-center">
			<h1 class="title">General</h1>
			<div class="flex flex-col lg:flex-row lg:space-x-4 lg:w-fit space-y-2 lg:space-y-0 w-full">
				{#if $appSession.isAdmin}
					<button type="submit" class="btn btn-sm bg-sources" disabled={loading}
						>{loading ? $t('forms.saving') : $t('forms.save')}</button
					>
					{#if source.bitbucketAppId}
						<button class="btn btn-sm" on:click|preventDefault={changeSettings}
							>{$t('source.change_app_settings', { name: 'Bitbucket' })}</button
						>
					{:else}
						<button class="btn btn-sm" on:click|preventDefault|stopPropagation={newOAuthConsumer}
							>Create new OAuth Consumer</button
						>
					{/if}
				{/if}
			</div>
		</div>
		<a
			href="https://docs.coollabs.io/coolify/sources#how-to-integrate-with-bitbucket"
			class="font-bold "
			target="_blank noreferrer"
			rel="noopener noreferrer">Documentation and detailed instructions.</a
		>

		<div class="grid grid-flow-row gap-2 lg:px-10">
			<div class="grid grid-flow-row gap-2">
				<div class="mt-2 grid grid-cols-2 items-center">
					<label for="name" class="text-base font-bold text-stone-100">{$t('forms.name')}</label>
					<input class="w-full" name="name" id="name" required bind:value={source.name} />
				</div>
			</div>
			<div class="grid grid-cols-2 items-center">
				<label for="htmlUrl" class="text-base font-bold text-stone-100">HTML URL</label>
				<input class="w-full" name="htmlUrl" id="htmlUrl" required bind:value={source.htmlUrl} />
			</div>
			<div class="grid grid-cols-2 items-center">
				<label for="apiUrl" class="text-base font-bold text-stone-100">API URL</label>
				<input class="w-full" name="apiUrl" id="apiUrl" required bind:value={source.apiUrl} />
			</div>
			{#if selfHosted}
				<div class="grid grid-cols-2 items-center">
					<label for="customPort" class="text-base font-bold text-stone-100"
						>Custom SSH User <Explainer
							explanation={'If you use a self-hosted version of Git, you can provide a custom SSH user for all the Git related actions.'}
						/></label
					>
					<input
						class="w-full"
						name="customUser"
						id="customUser"
						disabled={!selfHosted}
						readonly={!selfHosted}
						required
						bind:value={source.customUser}
					/>
				</div>
				<div class="grid grid-cols-2 items-center">
					<label for="customPort" class="text-base font-bold text-stone-100"
						>Custom SSH Port <Explainer
							explanation={'If you use a self-hosted version of Git, you can provide custom port for all the Git related actions.'}
						/></label
					>
					<input
						class="w-full"
						name="customPort"
						id="customPort"
						disabled={!selfHosted}
						readonly={!selfHosted}
						required
						bind:value={source.customPort}
					/>
				</div>
			{/if}
			<div class="grid grid-cols-2 items-center">
				<label for="consumerWorkspaceName" class="text-base font-bold text-stone-100"
					>OAuth Consumer Workspace Name</label
				>
				<input
					class="w-full"
					name="consumerWorkspaceName"
					id="consumerWorkspaceName"
					required
					disabled={source.bitbucketAppId}
					readonly={source.bitbucketAppId}
					bind:value={source.bitbucketApp.consumerWorkspaceName}
				/>
			</div>
			<div class="grid grid-cols-2 items-center">
				<label for="consumerKey" class="text-base font-bold text-stone-100"
					>OAuth Consumer Key</label
				>
				<input
					class="w-full"
					disabled={source.bitbucketAppId}
					readonly={source.bitbucketAppId}
					name="consumerKey"
					id="consumerKey"
					required
					bind:value={source.bitbucketApp.consumerKey}
				/>
			</div>

			<div class="grid grid-cols-2 items-center">
				<label for="consumerSecret" class="text-base font-bold text-stone-100"
					>OAuth Consumer Secret</label
				>
				<CopyPasswordField
					disabled={source.bitbucketAppId}
					readonly={source.bitbucketAppId}
					isPasswordField={true}
					name="consumerSecret"
					id="consumerSecret"
					required
					bind:value={source.bitbucketApp.consumerSecret}
				/>
			</div>
		</div>
	</form>
</div>
