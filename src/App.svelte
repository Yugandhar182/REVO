<script>
	import { onMount } from 'svelte';
	import { RevoGrid } from "@revolist/svelte-datagrid";
	import { defineCustomElements } from "@revolist/revogrid/loader";
  
	let source;
	let columns;
	let sortColumn = '';
	let sortOrder = '';
	let isModalOpen = false;
	let newCandidate = {
	  FirstName: '',
	  Surname: '',
	  Email: '',
	  Mobile: ''
	};
  
	// Fetch data from API
	onMount(() => {
	  fetch("https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E")
		.then(response => response.json())
		.then(data => {
		  console.log(data);
  
		  if (Array.isArray(data.data)) {
			source = data.data.map(candidate => ({
			  ID: candidate.id,
			  FirstName: candidate.firstName,
			  Surname: candidate.surname,
			  Email: candidate.email,
			  Mobile: candidate.mobile
			}));
			columns = Object.keys(source[0]).map(key => ({
			  name: key,
			  prop: key,
			  size: 200,
			  sortable: true
			}));
		  } else {
			console.error("Fetched data is not an array:", data);
		  }
		})
		.catch(error => {
		  console.error("Error fetching data:", error);
		});
	});
  
	function customFilter(value, filterText) {
	  if (value && filterText) {
		const lowerCaseValue = value.toString().toLowerCase();
		const lowerCaseFilterText = filterText.toLowerCase();
		return lowerCaseValue.includes(lowerCaseFilterText);
	  }
	  return false;
	}
  
	function customSort(a, b) {
	  if (sortColumn === '') {
		return 0;
	  }
  
	  const prop = sortColumn;
	  const order = sortOrder;
  
	  if (prop === 'ID') {
		return (order === 'asc' ? 1 : -1) * (parseInt(a[prop]) - parseInt(b[prop]));
	  } else {
		const lowerCaseA = a[prop].toLowerCase();
		const lowerCaseB = b[prop].toLowerCase();
		if (lowerCaseA < lowerCaseB) return order === 'asc' ? -1 : 1;
		if (lowerCaseA > lowerCaseB) return order === 'asc' ? 1 : -1;
		return 0;
	  }
	}
  
	function openModal() {
	  isModalOpen = true;
	}
  
	function closeModal() {
	  isModalOpen = false;
	  // Reset newCandidate object
	  newCandidate = {
		FirstName: '',
		Surname: '',
		Email: '',
		Mobile: ''
	  };
	}
  
	async function addCandidate() {
	  try {
		const response = await fetch("https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E", {
		  method: 'POST',
		  headers: {
			'Content-Type': 'application/json'
		  },
		  body: JSON.stringify(newCandidate)
		});
  
		if (response.ok) {
		  const data = await response.json();
		  console.log("New candidate added:", data);
  
		  source = [...source, newCandidate];
		} else {
		  console.error("Failed to add new candidate:", response);
		}
	  } catch (error) {
		console.error("Error adding new candidate:", error);
	  }
  
	  closeModal();
	}
  
	function generateUniqueId() {
	  return Math.random().toString(36).substr(2, 9);
	}
  
	defineCustomElements();
  </script>
  
  <style>
	@import 'bootstrap/dist/css/bootstrap.min.css';
	/* Your existing styles */
	revo-grid {
	  width: 100%;
	  height: 100%;
	}
	main {
	  font-family: sans-serif;
	  text-align: center;
	  height: calc(80vh);
	}
	.modal {
	  display: block;
	  background-color: rgba(0, 0, 0, 0.5);
	  position: fixed;
	  top: 0;
	  bottom: 0;
	  left: 0;
	  right: 0;
	  z-index: 999;
	  overflow: auto;
	}
	.modal-dialog {
	  margin: 10vh auto;
	  max-width: 400px;
	  background-color: white;
	  padding: 20px;
	  border-radius: 4px;
	}
	/* Add Bootstrap classes to form elements */
	.modal-dialog label {
	  display: block;
	  margin-bottom: 0.5rem;
	  font-weight: bold;
	}
	.modal-dialog input[type="text"],
	.modal-dialog input[type="email"],
	.modal-dialog input[type="tel"] {
	  width: 100%;
	  padding: 0.375rem 0.75rem;
	  border-radius: 0.25rem;
	  border: 1px solid #ced4da;
	  background-color: #fff;
	  box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
	  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
	}
	.modal-dialog button[type="submit"],
	.modal-dialog button[type="button"] {
	  margin-top: 1rem;
	}
  </style>
  
  
  <main>
	{#if source && columns}
	  <RevoGrid class="grid" source={source} resize="true" columns={columns} theme="material"
		sort={customSort} filter={customFilter}
		onSort={(e) => { sortColumn = e.detail.prop; sortOrder = e.detail.order; }}
	  >
	  </RevoGrid>
	{:else}
	  <p>Loading data...</p>
	{/if}
  
	<button on:click={openModal}>Add Candidate</button>
  
	{#if isModalOpen}
	  <div class="modal">
		<div class="modal-dialog">
		  <h3>Add Candidate</h3>
		  <form on:submit|preventDefault={addCandidate}>
			<div>
			  <label for="firstName">First Name:</label>
			  <input type="text" id="firstName" bind:value={newCandidate.FirstName} required>
			</div>
			<div>
			  <label for="surname">Surname:</label>
			  <input type="text" id="surname" bind:value={newCandidate.Surname} required>
			</div>
			<div>
			  <label for="email">Email:</label>
			  <input type="email" id="email" bind:value={newCandidate.Email} required>
			</div>
			<div>
			  <label for="mobile">Mobile:</label>
			  <input type="tel" id="mobile" bind:value={newCandidate.Mobile} required>
			</div>
			<button type="submit">Save</button>
			<button type="button" on:click={closeModal}>Cancel</button>
		  </form>
		</div>
	  </div>
	{/if}
  </main>