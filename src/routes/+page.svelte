<script lang="ts">
	import json from '$lib/ikea.json';
	import { onMount } from 'svelte';
	import { stores, availabilities } from 'ikea-availability-checker';

	let currentTime = new Date().toLocaleTimeString();
	setInterval(() => {
		currentTime = new Date().toLocaleTimeString();
	}, 1000);

	let availabilityList: storeAvailability[] = [];

	interface storeAvailability {
		storename: string;
		products: {
			name: string;
			id: string;
			productType: string;
			quantity: number;
		}[];
	}

	onMount(async () => {
		const storeObjects = stores.findById(json.storeIds);
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
						quantity: availability.stock
					});
				}
			}
			list.push(storeAvailability);
		}

		availabilityList = list;
	});

	function getProductRowClass(product: { name: string; quantity: number }) {
		if (product.quantity < 2) {
			return 'out-of-stock';
		} else if (product.quantity < 5) {
			return 'low-stock';
		} else {
			return 'in-stock';
		}
	}
</script>

<div class="container">
	<div class="title">Ikea Stock Checker</div>
	<div class="time">{currentTime}</div>
	<div class="divider" />
	{#each availabilityList as store}
		<div class="store">
			<h2 class="store-name">{store.storename}</h2>
			<table class="availability-table">
				<thead>
					<tr>
						<th>Namn</th>
						<th>Kvantitet</th>
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
		height: 100vh;
	}
	.title {
		font-size: 2em;
		margin-bottom: 20px;
	}
	.time {
		font-size: 1.5em;
	}
	.store {
		margin-bottom: 20px;
		display: flex;
		flex-direction: column;
		align-items: center;
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
</style>
