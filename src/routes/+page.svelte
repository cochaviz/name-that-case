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

<div
	class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-[#181818] via-[#232323] to-[#101c10] text-[#6aff6a] font-mono px-4"
>
	<h1 class="text-6xl md:text-7xl font-bold tracking-widest mb-4 animate-fadein drop-shadow-lg">
		name-that-case
	</h1>
	<p
		class="text-2xl md:text-3xl text-[#eaeaea] mb-16 animate-fadein2 flex items-center gap-3 relative"
	>
		A tiny city for your <span class="text-[#6aff6a]">DFIR</span> case.
		<span class="relative group z-0">
			<span
				class="inline-flex w-8 h-8 rounded-full border border-[#6aff6a] text-[#181818] bg-[#6aff6a] items-center justify-center text-xl font-black cursor-help select-none shadow-md transition-all duration-200 group-hover:bg-[#aaffaa] group-hover:text-[#181818] group-hover:shadow-lg group-hover:scale-110 animate-glow-symbol leading-none text-center"
			>
				?
			</span>
			<span
				class="absolute left-1/2 translate-y-8 -translate-x-1/2 mt-4 w-80 bg-[#181818] text-[#eaeaea] border border-[#6aff6a] rounded-lg px-4 py-3 text-base shadow-2xl z-20 opacity-0 pointer-events-none group-hover:opacity-100 group-hover:pointer-events-auto transition-opacity duration-200 text-center"
			>
				<span
					class="absolute top-[-10px] left-1/2 -translate-x-1/2 w-0 h-0 border-l-8 border-l-transparent border-r-8 border-r-transparent border-b-8 border-b-[#181818] z-20"
				></span>
				We like to name our DFIR cases after small cities. This is something we picked up on early in
				our careers from other incident responders. We thought we might share this little inside joke,
				and make it an outside joke.
			</span>
		</span>
	</p>
	<div
		class="border border-[#6aff6a] rounded-2xl p-12 w-full max-w-3xl flex flex-col gap-10 bg-[#101c10]/90 shadow-2xl animate-popin transition-transform duration-300 hover:scale-105 hover:shadow-3xl backdrop-blur-md z-10"
	>
		<div class="flex flex-row justify-between items-center gap-4 text-[#eaeaea] mb-2">
			<span class="text-xl whitespace-nowrap">Select a country</span>
			<div
				class="relative w-80"
				bind:this={dropdownRef}
				tabindex="0"
				on:keydown={handleDropdownKeydown}
			>
				<button
					class="w-full flex justify-between items-center bg-[#181818] border border-[#6aff6a] rounded-lg px-6 py-3 text-[#6aff6a] text-lg focus:outline-none focus:ring-2 focus:ring-[#6aff6a] transition-all custom-dropdown-btn"
					type="button"
					aria-haspopup="listbox"
					aria-expanded={dropdownOpen}
					on:click={() => (dropdownOpen = !dropdownOpen)}
				>
					<span>{selectedCountry || 'Select...'}</span>
					<span class="ml-2 text-[#6aff6a] text-xl">▼</span>
				</button>
				{#if dropdownOpen}
					<ul
						class="absolute z-10 left-0 mt-2 w-full bg-[#181818] border border-[#6aff6a] rounded-lg shadow-2xl max-h-60 overflow-auto animate-popin"
						role="listbox"
					>
						{#each countries as country, i}
							<li
								class="px-6 py-3 cursor-pointer hover:bg-[#232323] text-[#6aff6a] text-lg transition-colors"
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
		<div class="flex flex-col items-center gap-4 py-10 min-h-[110px] justify-center w-full">
			{#if loading}
				<span class="text-2xl text-[#eaeaea] text-center w-full animate-fadein2">Loading...</span>
			{:else if city}
				<span class="text-5xl md:text-6xl font-extrabold tracking-widest text-center w-full"
					>{city.name}</span
				>
				<span class="text-2xl text-[#eaeaea] text-center w-full"
					>Population ~{city.population} - {city.country}</span
				>
			{/if}
		</div>
		<div class="flex gap-8 mt-6 justify-center">
			<button
				class="border border-[#6aff6a] px-8 py-3 rounded-lg text-lg font-semibold hover:bg-[#6aff6a] hover:text-[#181818] transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-[#6aff6a] shadow-md"
				on:click={tryAnother}
				disabled={loading || !selectedCountry}
			>
				TRY ANOTHER
			</button>
			<button
				class="border border-[#6aff6a] px-8 py-3 rounded-lg text-lg font-semibold hover:bg-[#6aff6a] hover:text-[#181818] transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-[#6aff6a] relative shadow-md"
				on:click={copyToClipboard}
				disabled={!city}
			>
				COPY TO CLIPBOARD
				{#if showCopied}
					<span
						class="absolute -top-8 left-1/2 -translate-x-1/2 text-sm bg-[#222] px-3 py-1 rounded text-[#6aff6a] border border-[#6aff6a] animate-fadein2 shadow-lg"
						>Copied!</span
					>
				{/if}
			</button>
		</div>
	</div>
</div>

<style>
	@keyframes fadein {
		from {
			opacity: 0;
			transform: translateY(-20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	@keyframes fadein2 {
		from {
			opacity: 0;
			transform: translateY(20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
	@keyframes popin {
		from {
			opacity: 0;
			transform: scale(0.95);
		}
		to {
			opacity: 1;
			transform: scale(1);
		}
	}
	@keyframes glow {
		0%,
		100% {
			text-shadow:
				0 0 8px #6aff6a,
				0 0 4px #6aff6a;
		}
		50% {
			text-shadow:
				0 0 8px #6aff6a,
				0 0 2px #6aff6a;
		}
	}
	.animate-fadein {
		animation: fadein 0.35s cubic-bezier(0.4, 0, 0.2, 1) both;
	}
	.animate-fadein2 {
		animation: fadein2 0.45s cubic-bezier(0.4, 0, 0.2, 1) both;
	}
	.animate-popin {
		animation: popin 0.25s cubic-bezier(0.4, 0, 0.2, 1) both;
	}
	.animate-glow {
		animation: glow 1.2s infinite alternate;
	}
	.shadow-2xl {
		box-shadow:
			0 8px 40px 0 #6aff6a33,
			0 1.5px 8px 0 #000a;
	}
	.shadow-3xl {
		box-shadow:
			0 16px 64px 0 #6aff6a55,
			0 2px 16px 0 #000c;
	}
	.drop-shadow-lg {
		filter: drop-shadow(0 2px 8px #6aff6a88);
	}
	.drop-shadow-xl {
		filter: drop-shadow(0 4px 16px #6aff6a88);
	}
	.backdrop-blur-md {
		backdrop-filter: blur(8px);
	}
	.custom-dropdown-btn {
		background-image: none;
		outline: none;
		box-shadow: none;
		position: relative;
		z-index: 20;
		cursor: pointer;
	}
	ul[role='listbox'] {
		scrollbar-width: thin;
		scrollbar-color: #6aff6a #232323;
	}
	ul[role='listbox']::-webkit-scrollbar {
		width: 8px;
		background: #232323;
	}
	ul[role='listbox']::-webkit-scrollbar-thumb {
		background: #6aff6a;
		border-radius: 8px;
	}
	li.selected {
		background: #232323;
		font-weight: bold;
	}
	li.highlighted {
		background: #6aff6a33;
		color: #181818;
	}
	li[role='option'] {
		cursor: pointer;
	}
	button,
	.custom-dropdown-btn {
		cursor: pointer;
	}
	.flex.gap-8.mt-6.justify-center > button {
		cursor: pointer;
	}
	.animate-glow-symbol {
		box-shadow:
			0 0 8px #6aff6a88,
			0 0 2px #6aff6a44;
		transition:
			box-shadow 0.2s,
			background 0.2s,
			color 0.2s,
			transform 0.2s;
	}
	.group:hover .animate-glow-symbol {
		box-shadow:
			0 0 24px #6aff6a,
			0 0 8px #6aff6a;
	}
</style>
