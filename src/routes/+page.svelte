<script lang="ts">
	import { onMount } from 'svelte';
	import data from '$lib/data.json';

	let countries: { name: string; cities: any[] }[] = data.countries;
	let selectedCountry: string = '';
	let city: { name: string; population: number; country: string } | null = null;
	let showCopied = false;
	let loading = false;
	let dropdownOpen = false;
	let dropdownIndex = -1;
	let dropdownRef: HTMLDivElement | null = null;

	function initializeData() {
		if (countries.length > 0) {
			selectedCountry = countries[0].name;
			fetchCity(selectedCountry);
		}
	}

	function fetchCity(country: string) {
		loading = true;
		const countryObj = countries.find((c) => c.name === country);
		if (!countryObj) return;

		// Get a random city from the selected country
		const cities = countryObj.cities;
		const randomIndex = Math.floor(Math.random() * cities.length);
		city = cities[randomIndex];

		loading = false;
		showCopied = false;
	}

	function tryAnother() {
		if (selectedCountry) fetchCity(selectedCountry);
	}

	function copyToClipboard() {
		if (city) {
			navigator.clipboard.writeText(`${city.name}`);
			showCopied = true;
			setTimeout(() => (showCopied = false), 1200);
		}
	}

	$: if (selectedCountry) {
		fetchCity(selectedCountry);
	}

	onMount(() => {
		initializeData();
		document.addEventListener('click', handleClickOutside);
		return () => document.removeEventListener('click', handleClickOutside);
	});

	function handleClickOutside(e: MouseEvent) {
		if (dropdownRef && !dropdownRef.contains(e.target as Node)) {
			dropdownOpen = false;
			dropdownIndex = -1;
		}
	}

	function selectCountry(country: string) {
		selectedCountry = country;
		dropdownOpen = false;
		dropdownIndex = -1;
	}

	function handleDropdownKeydown(e: KeyboardEvent) {
		if (!dropdownOpen) return;
		if (e.key === 'ArrowDown') {
			dropdownIndex = (dropdownIndex + 1) % countries.length;
			e.preventDefault();
		} else if (e.key === 'ArrowUp') {
			dropdownIndex = (dropdownIndex - 1 + countries.length) % countries.length;
			e.preventDefault();
		} else if (e.key === 'Enter' && dropdownIndex >= 0) {
			selectCountry(countries[dropdownIndex].name);
			e.preventDefault();
		} else if (e.key === 'Escape') {
			dropdownOpen = false;
			dropdownIndex = -1;
			e.preventDefault();
		}
	}
</script>

<div class="container">
	<h1 class="title">name-that-case</h1>
	<p class="subtitle">
		A tiny city for your <span class="highlight">DFIR</span> case.
		<span class="help-container">
			<span class="help-icon">?</span>
			<span class="help-tooltip">
				<span class="tooltip-arrow"></span>
				We like to name our DFIR cases after small cities. This is something we picked up on early in
				our careers from other incident responders. We thought we might share this little inside joke,
				and make it an outside joke.
			</span>
		</span>
	</p>
	<div class="card">
		<div class="country-selector">
			<span class="selector-label">Select a country</span>
			<div
				class="dropdown-container"
				bind:this={dropdownRef}
				tabindex="0"
				on:keydown={handleDropdownKeydown}
			>
				<button
					class="dropdown-button"
					type="button"
					aria-haspopup="listbox"
					aria-expanded={dropdownOpen}
					on:click={() => (dropdownOpen = !dropdownOpen)}
				>
					<span class="selected-text">{selectedCountry || 'Select...'}</span>
					<span class="dropdown-arrow">▼</span>
				</button>
				{#if dropdownOpen}
					<ul class="dropdown-list" role="listbox">
						{#each countries as country, i}
							<li
								class="dropdown-item"
								class:selected={selectedCountry === country.name}
								class:highlighted={dropdownIndex === i}
								on:click={() => selectCountry(country.name)}
								on:mouseenter={() => (dropdownIndex = i)}
								role="option"
								aria-selected={selectedCountry === country.name}
								tabindex="-1"
							>
								{country.name}
							</li>
						{/each}
					</ul>
				{/if}
			</div>
		</div>
		<div class="city-display">
			{#if loading}
				<span class="loading-text">Loading...</span>
			{:else if city}
				<span class="city-name">{city.name}</span>
				<span class="city-info">Population ~{city.population} - {city.country}</span>
			{/if}
		</div>
		<div class="button-container">
			<button class="action-button" on:click={tryAnother} disabled={loading || !selectedCountry}>
				TRY ANOTHER
			</button>
			<button class="action-button copy-button" on:click={copyToClipboard} disabled={!city}>
				COPY TO CLIPBOARD
				{#if showCopied}
					<span class="copied-tooltip">Copied!</span>
				{/if}
			</button>
		</div>
	</div>
</div>

<style>
	:global(body) {
		margin: 0;
		padding: 0;
		font-family: 'IBM Plex Mono', 'Fira Mono', 'Menlo', 'Consolas', monospace;
		background: #181818;
		color: #6aff6a;
	}

	.container {
		width: 100vw;
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		background: #181818;
		color: #6aff6a;
		padding: 0;
	}

	.title {
		font-size: 3.5rem;
		font-weight: 700;
		letter-spacing: 0.08em;
		margin-bottom: 1.5rem;
		text-align: center;
		font-family: inherit;
		color: #6aff6a;
	}

	.subtitle {
		font-size: 1.25rem;
		color: #eaeaea;
		margin-bottom: 2.5rem;
		text-align: center;
		font-family: inherit;
	}

	.highlight {
		color: #6aff6a;
		font-family: inherit;
	}

	.card {
		border: 1px solid #6aff6a;
		padding: 2.5rem 2.5rem 2rem 2.5rem;
		background: #6aff6a0a;
		width: 100%;
		max-width: 600px;
		display: flex;
		flex-direction: column;
		gap: 2.5rem;
		box-shadow: none;
		border-radius: 0;
		align-items: center;
	}

	.country-selector {
		width: 100%;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 1.5rem;
		color: #eaeaea;
		font-family: inherit;
	}

	.selector-label {
		font-size: 1.25rem;
		font-family: inherit;
		color: #eaeaea;
		margin-right: 1.5rem;
		white-space: nowrap;
	}

	.dropdown-container {
		position: relative;
		width: 240px;
		font-family: inherit;
	}

	.dropdown-button {
		width: 100%;
		background: transparent;
		border: 1px solid #6aff6a;
		color: #6aff6a;
		font-size: 1.25rem;
		font-family: inherit;
		padding: 0.5rem 1.5rem 0.5rem 1rem;
		text-align: left;
		cursor: pointer;
		box-shadow: none;
		border-radius: 0;
		outline: none;
		display: flex;
		align-items: center;
		justify-content: space-between;
		transition: none;
	}

	.dropdown-arrow {
		margin-left: 1rem;
		color: #6aff6a;
		font-size: 1.5rem;
		font-family: inherit;
	}

	.dropdown-list {
		position: absolute;
		z-index: 10;
		left: 0;
		right: 0;
		width: 100%;
		min-width: 0;
		background: #181818;
		border: 1px solid #6aff6a;
		border-radius: 0;
		box-shadow: none;
		max-height: 200px;
		overflow-y: auto;
		font-family: inherit;
		/* Custom scrollbar for desktop */
		scrollbar-width: thin;
		scrollbar-color: #6aff6a #232323;
	}
	.dropdown-list::-webkit-scrollbar {
		width: 8px;
		background: #232323;
	}
	.dropdown-list::-webkit-scrollbar-thumb {
		background: #6aff6a;
		border-radius: 0;
	}
	.dropdown-list::-webkit-scrollbar-track {
		background: #232323;
	}

	.dropdown-item {
		padding: 0.5rem 1rem;
		cursor: pointer;
		color: #6aff6a;
		font-size: 1.1rem;
		font-family: inherit;
		background: transparent;
		border: none;
		transition: none;
	}
	.dropdown-item.selected,
	.dropdown-item.highlighted,
	.dropdown-item:hover {
		background: #181818;
		color: #eaeaea;
	}

	.city-display {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
		padding: 1.5rem 0 1rem 0;
		min-height: 110px;
		justify-content: center;
		width: 100%;
	}

	.loading-text {
		font-size: 1.5rem;
		color: #eaeaea;
		text-align: center;
		width: 100%;
		font-family: inherit;
	}

	.city-name {
		font-size: 2.5rem;
		font-weight: 700;
		letter-spacing: 0.08em;
		text-align: center;
		width: 100%;
		color: #6aff6a;
		font-family: inherit;
	}

	.city-info {
		font-size: 1.25rem;
		color: #eaeaea;
		text-align: center;
		width: 100%;
		font-family: inherit;
	}

	.button-container {
		display: flex;
		flex-direction: row;
		gap: 2rem;
		margin-top: 1.5rem;
		justify-content: center;
		width: 100%;
	}

	.action-button {
		border: 1px solid #6aff6a;
		color: #6aff6a;
		padding: 0.75rem 2rem;
		border-radius: 0;
		font-size: 1.25rem;
		font-family: inherit;
		font-weight: 400;
		background: #fff00000;
		cursor: pointer;
		transition: none;
		box-shadow: none;
		width: 100%;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.action-button:hover:not(:disabled),
	.action-button:focus:not(:disabled) {
		background: #6aff6a20;
		border-color: #6aff6a;
	}

	.action-button:active:not(:disabled) {
		background: #6aff6a40;
		border-color: #6aff6a;
	}

	.action-button:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	.copy-button {
		position: relative;
	}

	.copied-tooltip {
		position: absolute;
		top: -2rem;
		left: 50%;
		transform: translateX(-50%);
		font-size: 1rem;
		background: #181818;
		padding: 0.25rem 0.75rem;
		border: 1px solid #6aff6a;
		color: #6aff6a;
		font-family: inherit;
		border-radius: 0;
		box-shadow: none;
		white-space: nowrap;
	}

	.help-container {
		display: none;
	}

	@media (max-width: 700px) {
		.card {
			padding: 1.5rem 0.5rem 1rem 0.5rem;
			max-width: 98vw;
		}
		.country-selector {
			flex-direction: column;
			align-items: flex-start;
			gap: 0.5rem;
			width: 100%;
		}
		.button-container {
			flex-direction: column;
			gap: 1rem;
			width: 100%;
		}
		.dropdown-container {
			width: 100%;
			max-width: 100vw;
		}
		.dropdown-button {
			font-size: 1.1rem;
			padding: 0.75rem 1rem;
			width: 100%;
		}
		.dropdown-list {
			width: 100%;
			left: 0;
			right: 0;
			min-width: 0;
			box-sizing: border-box;
		}
		.action-button {
			font-size: 1.1rem;
			padding: 1rem 0.5rem;
			width: 100%;
		}
		.city-name {
			font-size: 2rem;
		}
		.city-info {
			font-size: 1rem;
		}
		.card,
		.container {
			box-sizing: border-box;
		}
		.container {
			padding: 0.5rem;
		}
	}
</style>
