<script>
	import { onMount } from 'svelte';
	import { spring } from 'svelte/motion';
	import { fade, fly, slide } from 'svelte/transition';
	import { backInOut } from 'svelte/easing';

	// Variables to store results
	let priceInUSD;
	let priceInNPR;

	let loadedDOM = false;
	onMount(() => {
		loadedDOM = true;
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
</script>

{#if loadedDOM}
	<div class="flex justify-center min-h-screen">
		<div class="flex-col justify-center items-center">
			<div
				in:fly={{ y: 500, duration: 1000, opacity: -10 }}
				out:fade
				class="uppercase py-10 my-10 text-3xl md:text-7xl lg:text-8xl flex justify-center italic"
			>
				BUFF163 Sales Calculator
			</div>
			<div class="flex justify-center items-center h-1/4 my-10">
				<div
					in:slide={{ duration: 1000, axis: 'y', easing: backInOut }}
					class="card z-10 shadow-xl w-full md:w-1/2 bg-slate-100"
				>
					<div
						in:fade={{
							duration: 500
						}}
						class="card-body flex justify-center"
					>
						<div class="flex flex-col justify-center items-center">
							<div class="py-2">
								Login to your <a href="https://buff.163.com/"><strong>BUFF163</strong></a> account
							</div>
							<div class="pb-2">Navigate to 'Sale' tab followed by 'Sale History' section</div>
							<div class="pb-2">Click the 'Export Records' button</div>
							<div class="pb-2">Choose 'sell order' type and define the date range</div>
							<div class="pb-2">Confirm and download 'CSV' file</div>
						</div>

						<div class="card-actions flex justify-center items-center my-5">
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
		}}
	>
		<circle class="fill-secondary" cx={$coords.x} cy={$coords.y} r={$size} />
	</svg>
{/if}
