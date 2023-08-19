<script>
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
	let priceInUSD;
	let priceInNPR;

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

				priceInUSD = (afterfees * currencyData.cny.usd).toFixed(2);
				priceInNPR = (afterfees * currencyData.cny.npr).toFixed(2);

				const modal = document.getElementById('modal');
				modal.showModal();

				console.log('CNY TO USD CONVERSION RATE: ' + currencyData.cny.usd);
				console.log('CNY TO NPR CONVERSION RATE: ' + currencyData.cny.npr);
				console.log('TOTAL AMOUNT: CNY ' + totalAmount.toFixed(2));
				console.log('TOTAL AMOUNT AFTER FEES: CNY ' + afterfees.toFixed(2));
				console.log('TOTAL SALES AMOUNT: USD ' + priceInUSD);
				console.log('TOTAL SALES AMOUNT: NPR ' + priceInNPR);
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
		htmlElement.className = 'transition-all';
		if (userPreference.matches) {
			htmlElement.setAttribute('data-theme', 'dracula');
		} else {
			htmlElement.setAttribute('data-theme', 'emerald');
		}
	}

	function toggleTheme() {
		const htmlElement = document.querySelector('html');
		const currentTheme = htmlElement.getAttribute('data-theme');

		if (currentTheme === 'emerald') {
			htmlElement.setAttribute('data-theme', 'dracula');
		} else {
			htmlElement.setAttribute('data-theme', 'emerald');
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
			class="h-9 absolute z-10 top-14 left-14 mix-blend-overlay opacity-0"
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
						in:slide={{ duration: 1000, axis: 'y', easing: backInOut }}
						class="card z-10 shadow-xl w-full lg:w-1/2 bg-slate-700"
					>
						<div
							in:fade={{
								duration: 500
							}}
							class="card-body flex justify-center"
						>
							<div class="flex flex-col justify-center items-start md:items-center text-white">
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
			<h3 class="font-bold text-lg">Stats</h3>
			<p>Total Sales (USD): {priceInUSD}</p>
			<p>Total Sales (NPR): {priceInNPR}</p>
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
