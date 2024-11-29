<script lang="ts">
	import { connection, lang, states, ripple } from '$lib/Stores';
	import Modal from '$lib/Modal/Index.svelte';
	import ConfigButtons from '$lib/Modal/ConfigButtons.svelte';
	import { getName, getSupport } from '$lib/Utils';
	import Select from '$lib/Components/Select.svelte';
	import { callService } from 'home-assistant-js-websocket';
	import Ripple from 'svelte-ripple';
	import Icon from '@iconify/svelte';

	export let isOpen: boolean;
	export let sel: any;

	$: entity = $states[sel?.entity_id];
	$: state = entity?.state;
	$: attributes = entity?.attributes;
	$: supported_features = attributes?.supported_features;

	$: supports = getSupport(supported_features, {
		TURN_ON: 1,
		TURN_OFF: 2,
		PAUSE: 4,
		STOP: 8,
		RETURN_HOME: 16,
		FAN_SPEED: 32,
		BATTERY: 64,
		STATUS: 128,
		SEND_COMMAND: 256,
		LOCATE: 512,
		CLEAN_SPOT: 1024,
		MAP: 2048,
		STATE: 4096,
		START: 8192
	});

	$: options = attributes?.fan_speed_list?.map((option: string) => ({
		id: option,
		label: $lang(option?.toLowerCase())
	}));

	// Mop Intensity Opties
	$: mop_intensity_options = attributes?.mop_intensity_list?.map((option: string) => ({
		id: option,
		label: $lang(option?.toLowerCase())
	}));	

	/**
	 * Handle click
	 */
	function handleClick(service: string) {
		callService($connection, 'vacuum', service, {
			entity_id: entity?.entity_id
		});
	}

	/**
	 * Handle change 'set_fan_speed'
	 */
	 function handleChange(selected_fan_speed: string) {
    console.log('Selected fan speed:', selected_fan_speed);

    callService($connection, 'vacuum', 'set_fan_speed', {
        entity_id: 'vacuum.roborock_q7_max',
        fan_speed: selected_fan_speed,
    });
}

	function cleanRoom(script_entity: string) {
		callService($connection, 'script', 'turn_on', {
			entity_id: script_entity,
		});
	}	
	
	function handleMopIntensityChange(mop_intensity: string) {
		callService($connection, 'select', 'select_option', {
			entity_id: 'select.roborock_q7_max_mop_intensity',
			option: mop_intensity,
		});
	}
</script>

{#if isOpen}
	<Modal>
		<h1 slot="title">{getName(sel, entity)}</h1>

		<h2>{$lang('state')}</h2>

		{#if entity}
			{$lang(state)}
			<br />

			{#if supports?.BATTERY}
				<h2>{$lang('battery')}</h2>
				{attributes?.battery_level} %
			{/if}
		{/if}
		<!-- Fan Speed and Mop Intensity Options -->
		{#if supports?.FAN_SPEED}
			<div class="options-container">
				<div class="option">
					<h2>{$lang('fan_speed')}</h2>
					<Select
						{options}
						placeholder={$lang('options')}
						bind:value={attributes.fan_speed}
						on:change={(event) => handleChange(event?.detail)}
					/>
				</div>

				{#if attributes?.mopForbiddenEnable === 1 && mop_intensity_options}
					<div class="option">
						<h2>{$lang('mop_intensity')}</h2>
						<Select
							options={mop_intensity_options}
							placeholder={$lang('mop_intensity')}
							bind:value={attributes.mop_intensity}
							on:change={(event) => handleMopIntensityChange(event?.detail)}
						/>
					</div>
				{/if}
			</div>
		{/if}

		{#if supports?.TURN_ON || supports?.TURN_OFF || supports?.START || supports?.PAUSE || supports?.STOP || supports?.LOCATE || supports?.RETURN_HOME}
			<h2>{$lang('vacuum_commands')?.replace(':', '')}</h2>
		{/if}

		<div class="button-container">
			{#if supports?.TURN_ON}
				<button
					title={$lang('on')}
					class:selected={entity?.state === 'on'}
					on:click={() => handleClick('turn_on')}
					use:Ripple={$ripple}
				>
					<div class="icon" style="transform: scale(0.7);">
						<Icon icon="mdi:power-on" height="none" />
					</div>
				</button>
			{/if}

			{#if supports?.TURN_OFF}
				<button
					title={$lang('off')}
					class:selected={entity?.state === 'off'}
					on:click={() => handleClick('turn_off')}
					use:Ripple={$ripple}
				>
					<div class="icon" style="transform: scale(0.7);">
						<Icon icon="mdi:power-off" height="none" />
					</div>
				</button>
			{/if}

			{#if supports?.START}
				<button
					title={$lang('start')}
					class:selected={entity?.state === 'cleaning'}
					on:click={() => handleClick('start')}
					use:Ripple={$ripple}
				>
					<div class="icon">
						<Icon icon="ic:round-play-arrow" height="none" />
					</div>
				</button>
			{/if}

			{#if supports?.PAUSE}
				<button
					title={$lang('pause')}
					class:selected={entity?.state === 'paused'}
					on:click={() => handleClick('pause')}
					use:Ripple={$ripple}
				>
					<div class="icon">
						<Icon icon="ic:round-pause" height="none" />
					</div>
				</button>
			{/if}

			{#if supports?.STOP}
				<button
					title={$lang('stop')}
					class:selected={entity?.state === 'idle'}
					on:click={() => handleClick('stop')}
					use:Ripple={$ripple}
				>
					<div class="icon">
						<Icon icon="ic:round-stop" height="none" />
					</div>
				</button>
			{/if}

			{#if supports?.LOCATE}
				<button title={$lang('locate')} on:click={() => handleClick('locate')} use:Ripple={$ripple}>
					<div class="icon" style="transform: scale(0.65);">
						<Icon icon="fa:search" height="none" />
					</div>
				</button>
			{/if}

			{#if supports?.RETURN_HOME}
				<button
					title={$lang('return_home')}
					class:selected={entity?.state === 'returning'}
					on:click={() => handleClick('return_to_base')}
					use:Ripple={$ripple}
				>
					<div class="icon" style="transform: scale(0.85);">
						<Icon icon="ic:round-home" height="none" />
					</div>
				</button>
			{/if}
			<button title="Legen" on:click={() => cleanRoom('script.opvangbak_legen')} use:Ripple={$ripple}>
				<div class="icon" style="transform: scale(0.8);">
					<Icon icon="mdi:trash" height="none" />
				</div>
			</button>			
		</div>

		<!-- Room-specific cleaning buttons-->
		<h2>Selecteer ruimte</h2>
		<div class="button-container">
			<button title="Hal" on:click={() => cleanRoom('script.hal_stofzuigen')} use:Ripple={$ripple}>
				<div class="icon" style="transform: scale(0.8);">
					<Icon icon="mdi:door-open" height="none" />
				</div>
			</button>
			<button title="Keuken" on:click={() => cleanRoom('script.keuken_stofzuigen')} use:Ripple={$ripple}>
				<div class="icon" style="transform: scale(0.8);">
					<Icon icon="mdi:silverware-fork-knife" height="none" />
				</div>
			</button>
			<button title="Woonkamer" on:click={() => cleanRoom('script.woonkamer_stofzuigen')} use:Ripple={$ripple}>
				<div class="icon" style="transform: scale(0.8);">
					<Icon icon="mdi:sofa" height="none" />
				</div>
			</button>
			<button title="Bijkeuken" on:click={() => cleanRoom('script.bijkeuken_stofzuigen')} use:Ripple={$ripple}>
				<div class="icon" style="transform: scale(0.8);">
					<Icon icon="mdi:garage" height="none" />
				</div>
			</button>
		</div>

		<ConfigButtons />
	</Modal>
{/if}

<style>
	.options-container {
		display: flex;
		gap: 1rem;
	}

	.option {
		flex: 1;
	}

	.button-container > button {
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.icon {
		width: 1.6rem;
		height: 1.6rem;
	}
</style>