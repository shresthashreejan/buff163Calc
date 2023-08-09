<script>
	function csvUpload() {
		const csvInput = document.getElementById('csvInput');
		const selectedFile = csvInput.files[0];

		let data;
		const reader = new FileReader();

		reader.onload = async function (event) {
			const csvText = event.target.result;
			const rows = csvText.split('\n');
			data = rows.map((row) => row.split(','));

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
				console.log('CNY TO USD CONVERSION RATE: ' + currencyData.cny.usd);
				console.log('CNY TO NPR CONVERSION RATE: ' + currencyData.cny.npr);

				const afterfees = totalAmount * 0.975;
				console.log('TOTAL AMOUNT: CNY ' + totalAmount.toFixed(2));
				console.log('TOTAL AMOUNT AFTER FEES: CNY ' + afterfees.toFixed(2));

				const priceInUSD = afterfees * currencyData.cny.usd;
				const priceInNPR = afterfees * currencyData.cny.npr;
				console.log('TOTAL SALES AMOUNT: USD ' + priceInUSD.toFixed(2));
				console.log('TOTAL SALES AMOUNT: NPR ' + priceInNPR.toFixed(2));
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

<div class="flex justify-center items-center min-h-screen">
	<div class="card w-auto bg-base-100 shadow-xl">
		<div class="card-body items-center text-center">
			<h2 class="text-4xl uppercase py-4">BUFF163 Sales Calculator</h2>
			<p class="text-lg">Upload your csv file!</p>
			<div class="card-actions flex justify-center items-center">
				<input type="file" id="csvInput" class="file-input file-input-bordered w-full max-w-xs" />
				<button class="btn btn-primary" on:click={csvUpload}>Upload</button>
			</div>
		</div>
	</div>
</div>
