<script lang="ts">
	import json from '$lib/ikea.json';
	import { onMount } from 'svelte';
	import { stores, availabilities } from 'ikea-availability-checker';

	let availabilityList: storeAvailability[] = [];
	let lastFetchTime: string | undefined;

	interface storeAvailability {
		storename: string;
		products: {
			name: string;
			id: string;
			productType: string;
			quantity: number;
			rare?: boolean;
		}[];
	}

	async function fetchAvailability() {
		const storeObjects = stores.findById(json.storeIds);
		storeObjects.sort((a, b) => json.storeIds.indexOf(a.buCode) - json.storeIds.indexOf(b.buCode));

		const availabilityObjects = await availabilities(
			storeObjects,
			json.products.map((p) => p.id)
		);

		const list = [];
		for (const store of storeObjects) {
			const storeAvailability: storeAvailability = {
				storename: store.name,
				products: []
			};
			const availabilities = availabilityObjects.filter((a) => a.buCode === store.buCode);
			for (const product of json.products) {
				const availability = availabilities.find((a) => a.productId === product.id);
				if (availability) {
					storeAvailability.products.push({
						name: product.name,
						id: product.id,
						productType: product.productType,
						quantity: availability.stock,
						rare: product.rare
					});
				}
			}
			list.push(storeAvailability);
			lastFetchTime = new Date().toLocaleTimeString('sv-SE', {
				hour: '2-digit',
				minute: '2-digit'
			});
		}
		return list;
	}

	onMount(async () => {
		availabilityList = await fetchAvailability();
	});

	setInterval(
		async () => {
			availabilityList = await fetchAvailability();
		},
		10 * 60 * 1000
	);

	function getProductRowClass(product: { name: string; quantity: number; rare?: boolean }) {
		if (product.quantity === 0) {
			return 'out-of-stock';
		} else if (product.rare) {
			return 'rare-in-stock';
		} else if (product.quantity < 5) {
			return 'low-stock';
		} else {
			return 'in-stock';
		}
	}
</script>

<div class="container">
	<div class="title">Ikea Stock Checker</div>
	{#if lastFetchTime}
		<div>Data fetched: {lastFetchTime}</div>
	{/if}
	{#each availabilityList as store}
		<div class="store">
			<h2 class="store-name">{store.storename}</h2>
			<table class="availability-table">
				<thead>
					<tr>
						<th>Namn</th>
						<th>Antal</th>
						<th>Produkttyp</th>
					</tr>
				</thead>
				<tbody>
					{#each store.products as product}
						<tr class={getProductRowClass(product)}>
							<td
								><a href="https://www.ikea.com/se/sv/p/-{product.id}" target="_blank"
									>{product.name}</a
								></td
							>
							<td>{product.quantity}</td>
							<td>{product.productType}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	{/each}
</div>

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 100%;
		overflow-y: auto;
	}
	.title {
		font-size: 1.8em;
		margin-bottom: 10px;
	}
	.time {
		font-size: 1.5em;
	}
	.store {
		margin-bottom: 20px;
		display: flex;
		flex-direction: column;
		align-items: center;
		max-width: 100%;
	}
	.availability-table {
		border-collapse: collapse;
		margin: 0 auto; /* Center the table */
		width: fit-content;
	}
	.availability-table th,
	.availability-table td {
		border: 1px solid green; /* Add gridlines */
		padding: 8px;
		text-align: left;
		text-overflow: ellipsis;
		word-wrap: break-word;
		overflow: hidden;
		text-overflow: ellipsis;
	}
	.divider {
		width: 100%;
		margin: 20px 0;
	}
	.store-name {
		font-size: 1.5em;
		margin-bottom: 10px;
	}
	.in-stock {
		color: lightgreen;
	}
	.low-stock {
		color: yellow;
	}
	.out-of-stock {
		color: red;
	}
	a {
		color: inherit;
	}
	.rare-in-stock {
		animation: rainbow 3s linear;
		animation-iteration-count: infinite;
		color: white;
	}

	@keyframes rainbow {
		100%,
		0% {
			background-color: rgb(255, 0, 0);
		}
		16% {
			background-color: rgb(255, 255, 0);
		}
		33% {
			background-color: rgb(0, 255, 0);
		}
		50% {
			background-color: rgb(0, 255, 255);
		}
		66% {
			background-color: rgb(0, 0, 255);
		}
		83% {
			background-color: rgb(255, 0, 255);
		}
	}
</style>
