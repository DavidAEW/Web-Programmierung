<script>
	// Importieren Sie die notwendigen Funktionen
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';

	// Deklarieren Sie die Variablen
	let data = [];
	let deck;
	let options = [];
	let deckId = $page.params.deckId;
	let front;
	let back;
	let cardStatus;
	let newDeckId;
	let cardId;

	// Deklarieren Sie die Konstanten für die Legende der möglichen Statuswerte
	const status = [
		{ value: 0, label: 'neu' },
		{ value: 1, label: 'etwas gelernt' },
		{ value: 2, label: 'kann ich teilweise' },
		{ value: 3, label: 'kann ich' }
	];

	//Funktion zum Abrufen der Decks anhand der User-ID
	async function getOptions() {
		const API_URL = 'http://localhost:3001/Deck'; 
		try {
			const response = await fetch(API_URL,

				{
					method: 'GET',
					headers: {
						'Content-Type': 'application/json'
					},
					credentials: 'include'
				});
			const data = await response.json();
			options = data.map((item) => ({
				value: item.deckId,
				label: item.deckName
			}));
			return data;
		} catch (error) {
			console.error('Fehler beim Laden der Daten:', error);
		}
	}
	//Funktion zum Ausführen der Funktion, um die Decks anhand der User-ID zu laden und anzuzeigen
	async function loadOptionsAndSetValue() {
		await getOptions();
		deckId = $page.params.deckId;
		const selectElement = document.querySelector('select');
		selectElement.value = deckId;
	}

	//Funktion zum Abrufen der Karteikarten anhand der Deck-ID
	async function getAll(selectedOption) {
		const API_URL = `http://localhost:3001/Card_Deck/${selectedOption}`;
		const options = {
			method: 'GET',
			credentials: 'include',
			headers: {
				'Content-Type': 'application/json'
			},
		};

		const response = await fetch(API_URL, options);
		const data = await response.json();
		return data;
	}

	//Funktion zum Ausführen der Funktionen beim Seitenladen
	onMount(async () => {
		loadOptionsAndSetValue();
		try {
			data = await getAll(deckId);
			deck = await getOptions();
		} catch (error) {
			console.error('Fehler beim Laden der Daten:', error);
		}
	});


	//Funktion zum Aktualisieren der Karteikarten nach der Änderung
	async function updateCard(cardid, row) {

		cardId = cardid;
		front = row.front;
		back = row.back;
		cardStatus = row.cardStatus;
		newDeckId = row.deckId;

		try {
			const response = await fetch('http://localhost:3001/Card', {
				method: 'PUT',
				headers: {
					'Content-Type': 'application/json'
				},
				credentials: 'include',
				body: JSON.stringify({
					cardId,
					front,
					back,
					cardStatus,
					deckId: newDeckId
				})
			});

			if (response.ok) {
				getAll(deckId).then((result) => {
					data = result;
				});
			} else {
				console.error('Fehler beim Aktualisieren des Datensatzes:', response.statusText);
			}
		} catch (error) {
			console.error('Fehler bei der Fetch-Anfrage:', error);

		}
	}

	//Funktion zum Löschen der Karteikarten anhand der Card-ID
	async function deleteCard(row) {
		const cardId = row.cardId;
		const isConfirmed = confirm('Bist du sicher, dass du die Karte löschen möchtest?');
    
    if (!isConfirmed) {
        return;
    }

		try {
			const response = await fetch('http://localhost:3001/Card', {
				method: 'DELETE',
				headers: {
					'Content-Type': 'application/json'
				},
				credentials: 'include',
				body: JSON.stringify({
					cardId
				})
			});

			if (response.ok) {
				getAll(deckId).then((result) => {
					data = result;
				});

			} else {
				console.error('Fehler beim Aktualisieren des Datensatzes:', response.statusText);
			}
		} catch (error) {
			console.error('Fehler bei der Fetch-Anfrage:', error);
		}
	}

	//Funktion zum Navigieren zum neuen Deck
	function handleChange(event) {
		goto('/homePage/manageKarteikarten/deck/' + event.target.value);
		deckId = event.target.value;
		getAll(deckId).then((result) => {
			data = result;
		});
	}

	//Funktion zum Navigieren zurück zur Home-Seite
	function backtopage() {
		goto('/homePage');
	}
</script>

<main>
	<div class="container h-full mx-auto flex flex-row justify-center items-center mt-4">
		<div class="space-y-5">
			<h1 class="text-4xl text-center">Karteikarten verwalten</h1>
			<!-- Dropdown zur Auswahl des Decks -->
			<div class="container h-full mx-auto flex justify-center items-center mt-4">
				<div class="bg-primary-60 dark:bg-secondary-250 rounded-lg shadow-md p-4 w-5/6">
					<h2 class="text-xl font-bold mb-2 text-center text-primary-900">Deck wählen</h2>

					<select
						class="dark:bg-primary-60 block appearance-none w-full bg-primary-0 border border-gray-200 text-primary-400 py-3 px-4 pr-8 rounded leading-tight focus:outline-none focus:bg-white focus:border-gray-500"
						on:change={handleChange}
					>
						{#each options as option}
							<option value={option.value}>{option.label}</option>
						{/each}
					</select>
				</div>
			</div>
			<!-- Tabelle zur Anzeige der Karteikarten -->
			<div class="bg-primary-60 dark:bg-secondary-250 rounded-lg shadow-md p-4 mx-auto w-5/6">
				{#if data}
					<table class="text-center">
						<thead>
							<tr>
								<th class="w-32">Card-ID</th>
								<th class="w-64">Vorderseite</th>
								<th class="w-64">Rückseite</th>
								<th class="w-32">Cardstatus</th>
								<th class="w-32">Deck-ID</th>
								<th class="w-32">Update</th>
								<th class="w-32">Delete</th>
							</tr>
						</thead>
						<tbody>
							{#each data as row}
								<tr>
									<td>{row.cardId}</td>
									<td><input class="text-accent-400" bind:value={row.front} /></td>
									<td><input class="text-accent-400" bind:value={row.back} /></td>
									<td><input class="text-accent-400 w-1/2" bind:value={row.cardStatus} /></td>
									<td><input class="text-accent-400 w-1/2" bind:value={row.deckId} /></td>
									<td><button on:click={updateCard(row.cardId, row)}>✓</button> </td>
									<td><button on:click={deleteCard(row)}>x</button></td>
								</tr>
							{/each}
						</tbody>
					</table>
				{/if}

				{#await data}
					<p>Lade Daten...</p>
				{/await}
			</div>
		</div>
	</div>
	<div class="bg-primary-60 dark:bg-secondary-250 rounded-lg shadow-md p-4 mx-auto w-5/6 mt-5">
		<h2 class="text-2xl text-center">Legende</h2>
		<!-- Tabelle zur Anzeige der Legende der möglichen Decks -->
		<div class="flex flex-row justify-center gap-5 mt-4">
			{#if deck}
				<table class="text-center bg-white dark:bg-accent-50 text-accent-400 rounded">
					<thead>
						<tr>
							<th class="w-32">Deck-ID</th>
							<th class="w-64">Deckname</th>
						</tr>
					</thead>
					<tbody>
						{#each deck as row1}
							<tr>
								<td>{row1.deckId}</td>
								<td>{row1.deckName}</td>
							</tr>
						{/each}
					</tbody>
				</table>
			{/if}
			<!-- Tabelle zur Anzeige der Legende der möglichen Statuswerte -->
			{#if status}
				<table class="text-center bg-white dark:bg-accent-50 text-accent-400 rounded">
					<thead>
						<tr>
							<th class="w-32">Cardstatus</th>
							<th class="w-64">Statusname</th>
						</tr>
					</thead>
					<tbody>
						{#each status as row2}
							<tr>
								<td>{row2.value}</td>
								<td>{row2.label}</td>
							</tr>
						{/each}
					</tbody>
				</table>
			{/if}
		</div>
	</div>

	<!-- Button zum Navigieren zurück zur Home-Seite -->
	<div class="flex justify-center mx-auto mt-5 mb-5">
	<button
		class="bg-primary-60 dark:bg-accent-300 dark:hover:bg-primary-60 dark:hover:text-text-400 hover:bg-accent-300 hover:text-text-50 text-primary-400 dark:text-text-50 font-bold py-2 px-4 rounded"
		on:click={backtopage}
	>
		Zurück
	</button>
</div>

</main>
