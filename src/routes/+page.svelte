<script>
	import { base } from '$app/paths';
	import { onMount } from 'svelte';
	import { spring } from 'svelte/motion';
	import { fade, fly, slide } from 'svelte/transition';
	import { backInOut } from 'svelte/easing';

	// Import images
	import dragonlore from '$lib/assets/dragonlore.jpg';
	import gungnir from '$lib/assets/gungnir.jpg';
	import casehardened from '$lib/assets/casehardened.jpg';
	import toggleicon from '$lib/assets/toggleicon.webp';

	// Variables to store results
	let priceInCNY;
	let priceInUSD;
	let priceInNPR;
	let conversionUSD;
	let conversionNPR;

	let loadedDOM = false;
	onMount(() => {
		loadedDOM = true;
		setThemePreference();
	});

	let size = spring(25);
	let coords = spring(
		{ x: 50, y: 50 },
		{
			stiffness: 0.1,
			damping: 0.5
		}
	);

	function csvUpload() {
		const csvInput = document.getElementById('csvInput');
		const selectedFile = csvInput.files[0];
		const reader = new FileReader();

		reader.onload = async function (event) {
			const csvText = event.target.result;
			const rows = csvText.split('\n');
			let data = rows.map((row) => row.split(','));

			const yenPrices = getSalePrices(data);

			let totalAmount = 0;
			for (let i = 1; i < yenPrices.length - 1; i++) {
				let amount = yenPrices[i].toString().replace('¥ ', '');
				totalAmount = totalAmount + parseFloat(amount);
			}

			try {
				const response = await fetch(
					'https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/cny.json'
				);

				if (!response.ok) {
					throw new Error('Network response was not ok');
				}

				const currencyData = await response.json();
				let afterfees = totalAmount * 0.975;

				priceInCNY = afterfees.toFixed(2);
				priceInUSD = (afterfees * currencyData.cny.usd).toFixed(2);
				priceInNPR = (afterfees * currencyData.cny.npr).toFixed(2);
				conversionUSD = currencyData.cny.usd;
				conversionNPR = currencyData.cny.npr;

				const modal = document.getElementById('modal');
				modal.showModal();
			} catch {
				console.error('Error fetching data:', error);
			}
		};
		reader.readAsText(selectedFile);
	}

	function getSalePrices(data) {
		const yenPrices = data.map((row) => {
			return row.filter((value) => value.trim().startsWith('¥'));
		});
		return yenPrices;
	}

	function setThemePreference() {
		const userPreference = window.matchMedia('(prefers-color-scheme: dark)');
		const htmlElement = document.querySelector('html');
		const cardElement = document.querySelector('#card');

		htmlElement.className = 'transition-all';
		if (userPreference.matches) {
			htmlElement.setAttribute('data-theme', 'dracula');
			cardElement.className = 'card z-10 shadow-xl w-full lg:w-1/2 bg-neutral';
		} else {
			htmlElement.setAttribute('data-theme', 'emerald');
			cardElement.className = 'card z-10 shadow-xl w-full lg:w-1/2 bg-slate-200';
		}
	}

	function toggleTheme() {
		const htmlElement = document.querySelector('html');
		const currentTheme = htmlElement.getAttribute('data-theme');
		const cardElement = document.querySelector('#card');

		if (currentTheme === 'emerald') {
			htmlElement.setAttribute('data-theme', 'dracula');
			cardElement.className = 'card z-10 shadow-xl w-full lg:w-1/2 bg-neutral';
		} else {
			htmlElement.setAttribute('data-theme', 'emerald');
			cardElement.className = 'card z-10 shadow-xl w-full lg:w-1/2 bg-slate-200';
		}
	}
</script>

{#if loadedDOM}
	<div
		id="toggle"
		class="absolute w-36 h-36 top-[-50px] left-[-50px] bg-secondary rounded-full cursor-pointer z-10"
		on:click={toggleTheme}
	>
		<img
			id="toggle-icon"
			src={toggleicon}
			class="h-9 absolute z-10 top-14 left-14 mix-blend-overlay opacity-0 select-none"
		/>
	</div>

	<div class="flex md:justify-center min-h-screen">
		<div class="flex-col justify-center items-center">
			<div
				in:fly={{ y: 500, duration: 1000, opacity: -10 }}
				out:fade
				class="uppercase p-10 text-5xl sm:text-6xl md:text-7xl lg:text-8xl flex justify-center text-center italic"
			>
				BUFF163 Sales Calculator
			</div>
			<div>
				<div class="flex justify-center items-center h-1/4">
					<div
						id="card"
						in:slide={{ duration: 1000, axis: 'y', easing: backInOut }}
						class="card z-10 shadow-xl w-full lg:w-1/2 bg-neutral"
					>
						<div
							in:fade={{
								duration: 500
							}}
							class="card-body flex justify-center"
						>
							<div class="flex flex-col justify-center items-start md:items-center">
								<div class="py-2">
									Login to your <a href="https://buff.163.com/"><strong>BUFF163</strong></a> account
								</div>
								<div class="pb-2">Navigate to 'Sale' tab followed by 'Sale History' section</div>
								<div class="pb-2">Click the 'Export Records' button</div>
								<div class="pb-2">Choose 'sell order' type and define the date range</div>
								<div class="pb-2">Confirm and download 'CSV' file</div>
							</div>

							<div class="card-actions flex justify-center items-center my-1 md:my-3">
								<input
									type="file"
									id="csvInput"
									class="file-input file-input-secondary w-full max-w-xs"
								/>
								<button
									class="btn btn-primary border-0 bg-secondary hover:bg-white text-white hover:text-secondary"
									on:click={csvUpload}>Upload</button
								>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>

	<dialog id="modal" class="modal">
		<form method="dialog" class="modal-box">
			<h1 class="font-bold text-3xl flex justify-center uppercase italic">Stats</h1>
			<hr class="my-4 opacity-20" />
			<div>
				<h3 class="text-2xl italic">Total Sales</h3>
				<p class="text-lg">CNY: {priceInCNY}</p>
				<p class="text-lg">USD: {priceInUSD}</p>
				<p class="text-lg">NPR: {priceInNPR}</p>
			</div>
			<hr class="my-4 opacity-20" />
			<div>
				<h3 class="text-2xl italic">Conversion Rate (From CNY)</h3>
				<p class="text-lg">USD: {conversionUSD}</p>
				<p class="text-lg">NPR: {conversionNPR}</p>
			</div>
			<div class="modal-action">
				<button class="btn">Close</button>
			</div>
		</form>
	</dialog>

	<svg
		class="absolute w-full h-full left-0 top-0"
		on:mousemove={(e) => {
			coords.set({ x: e.clientX, y: e.clientY });
			e.stopPropagation();
			document.getElementById('toggle').className =
				'absolute w-28 h-28 top-[-50px] left-[-50px] bg-secondary rounded-full cursor-pointer z-10 flex transition-all';
			document.getElementById('toggle-icon').className =
				'h-9 absolute z-10 top-14 left-14 mix-blend-overlay opacity-100 transition-all delay-500';
		}}
	>
		<circle class="fill-secondary invisible md:visible" cx={$coords.x} cy={$coords.y} r={$size} />
	</svg>

	<div
		in:fly={{ y: 500, duration: 1000, opacity: -10 }}
		out:fade
		class="absolute top-[60%] hidden xl:flex"
	>
		<div class="flex">
			<div class="flex slide-container">
				<img class="image" src={dragonlore} alt="Image of AWP Dragon Lore" draggable="false" />
				<img class="image" src={gungnir} alt="Image of AWP Gungnir" draggable="false" />
				<img
					class="image"
					src={casehardened}
					alt="Image of AK-47 Case Hardened"
					draggable="false"
				/>
			</div>
			<div class="flex slide-container">
				<img class="image" src={dragonlore} alt="Image of AWP Dragon Lore" draggable="false" />
				<img class="image" src={gungnir} alt="Image of AWP Gungnir" draggable="false" />
				<img
					class="image"
					src={casehardened}
					alt="Image of AK-47 Case Hardened"
					draggable="false"
				/>
			</div>
		</div>
	</div>

	<footer class="footer footer-center p-4 z-50 absolute bottom-0">
		<div>
			<p class="italic">developed by <a href="https://github.com/shreejan-shrestha">shreejan</a></p>
		</div>
	</footer>
{/if}

<style lang="postcss">
	@keyframes slide {
		from {
			transform: translateX(0);
		}
		to {
			transform: translateX(-100%);
		}
	}
	.slide-container {
		min-width: 100%;
		animation: 5s slide infinite linear;
	}
	.image {
		width: 100%;
		height: 250px;
		object-fit: cover;
	}
</style>
