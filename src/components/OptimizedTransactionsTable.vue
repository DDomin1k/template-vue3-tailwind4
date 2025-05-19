<script>
	// Rozwiązałem problem wydajnościowy przez dodanie wirtualizacji wierszy oraz zastąpienie .slice().reverse() -> [].unshift()
	// Pętla v-for renderuje wycinek widocznych wierszy z całego arraya na podstawie pozycji scrolla i buforu
	// Computed nie odwraca już za każdym razem całego arraya ponieważ dane są do niego dodawane za pomocą unshift()
	// Dodatkowo aby zapewnić spójność wizualną dodałem wirtualny scrollbar odwzorowujący pozycję scrolla rzeczywistego

	let transactionCounter = 0
	function generateTransaction() {
		const now = new Date()
		const time = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}:${now.getSeconds().toString().padStart(2, '0')}.${now.getMilliseconds().toString().padStart(3, '0')}`
		const price = (Math.random() * 100 + 100).toFixed(2)
		const volume = Math.floor(Math.random() * 500) + 1
		const value = (price * volume).toFixed(2)
		return {
			n: transactionCounter++,
			t: time,
			p: price,
			v: volume,
			u: value,
		}
	}

	export default {
		data() {
			return {
				transactions: [],
				isLoading: true,
				updateInterval: null,
				initialBatchSize: 10000,
				updateIntervalMs: 200,

				rowHeight: 45, // Wysokość pojedynczego wiersza w pikselach zgodnie z obecnym layoutem
				visibleRows: 60, // Bufor widocznych wierszy, które są renderowane
				scrolledRows: 0,
			}
		},
		computed: {
			totalHeight() {
				// Konieczne aby odzwierciedlić rzeczywisty rozmiar scrollbara
				return this.transactions.length * this.rowHeight
			},
			paddingTop() {
				// Symulacja scrollowania przez przesuwanie zawartości tabeli
				return Math.max(0, this.scrolledRows - this.visibleRows / 2) * this.rowHeight
			},
			displayedTransactions() {
				const start = Math.max(0, this.scrolledRows - this.visibleRows / 2)
				const end = Math.min(this.transactions.length, start + this.visibleRows)
				return this.transactions.slice(start, end)
			},
		},
		methods: {
			loadInitialData() {
				console.log(`Generating ${this.initialBatchSize} initial transactions...`)
				const initialBatch = []
				for (let i = 0; i < this.initialBatchSize; i++) {
					initialBatch.unshift(generateTransaction())
				}
				this.transactions = initialBatch
				this.isLoading = false
				console.log('Initial data loaded.')
				this.startStreamingUpdates()
			},
			startStreamingUpdates() {
				if (this.updateInterval) {
					clearInterval(this.updateInterval)
				}
				console.log(
					`Starting streaming updates (1 transaction every ${this.updateIntervalMs}ms)...`
				)
				this.updateInterval = setInterval(() => {
					const newTransaction = generateTransaction()
					this.transactions.unshift(newTransaction)
				}, this.updateIntervalMs)
			},
			stopStreamingUpdates() {
				if (this.updateInterval) {
					clearInterval(this.updateInterval)
					this.updateInterval = null
					console.log('Stopped streaming updates.')
				}
			},
			onScroll(e) {
				this.scrolledRows = Math.floor(e.target.scrollTop / this.rowHeight)
			},
		},
		mounted() {
			this.loadInitialData()
		},
		beforeUnmount() {
			this.stopStreamingUpdates()
		},
	}
</script>

<template>
	<h1>Task: Table Performance</h1>

	<div class="candidate-table-container">
		<h3>Real-Time Transactions</h3>
		<div class="table-wrapper" @scroll="onScroll">
			<div :style="{ height: totalHeight + 'px', position: 'relative' }">
				<table
					:style="{
						position: 'absolute',
						top: paddingTop + 'px',
						left: 0,
						right: 0,
					}"
				>
					<thead>
						<tr>
							<th>Time</th>
							<th>Price</th>
							<th>Volume</th>
							<th>Value</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="tx in displayedTransactions" :key="tx.n">
							<td>{{ tx.t }}</td>
							<td>{{ tx.p }}</td>
							<td>{{ tx.v }}</td>
							<td>{{ tx.u }}</td>
						</tr>
					</tbody>
				</table>
			</div>
			<div v-if="isLoading" class="loading-overlay">Loading initial data...</div>
		</div>
		<div class="stats">Total Transactions: {{ transactions.length }}</div>
	</div>
</template>

<style>
	body {
		font-family: Arial, sans-serif;
		background-color: #f4f4f4;
		margin: 0;
		padding: 20px;
	}

	h1 {
		text-align: center;
	}

	.candidate-table-container {
		max-width: 800px;
		margin: auto;
		background: white;
		padding: 20px;
		border-radius: 8px;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
	}

	table {
		width: 100%;
		border-collapse: collapse;
	}

	th,
	td {
		padding: 10px;
		text-align: left;
		border-bottom: 1px solid #ddd;
	}

	th {
		background-color: #f2f2f2;
	}

	td {
		font-family: monospace;
	}

	tr:hover {
		background-color: #f1f1f1;
	}

	.table-wrapper {
		max-height: 400px;
		overflow-y: auto;
		position: relative;
	}

	.stats {
		margin-top: 10px;
		font-size: 14px;
		color: #555;
	}

	.loading-overlay {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(255, 255, 255, 0.8);
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 18px;
		color: #333;
	}
</style>
