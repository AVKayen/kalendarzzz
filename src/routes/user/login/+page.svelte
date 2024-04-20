<script>
	let name = '';
	let password = '';
	function login() {
		let success = fetch('http://localhost:8921/login', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				login: name,
				password: password
			})
		})
			.then((response) => response.json())
			.then((result) => {
				setCookie('ARL', result.resource_key, 1);
				window.location.href = '/';
			})
			.catch((error) => {
				console.error('Error:', error);
			});
	}

	function setCookie(name, value, days) {
		const date = new Date();
		date.setTime(date.getTime() + days * 24 * 60 * 60 * 1000);
		const expires = 'expires=' + date.toUTCString();
		document.cookie = name + '=' + value + ';' + expires + ';path=/';
	}


</script>

<div class="form">
	<h1>Login</h1>
	<input type="name" bind:value={name} id="name" name="name" placeholder="Name" required />
	<input
		type="password"
		bind:value={password}
		id="password"
		name="password"
		placeholder="Password"
		required
	/>
	<button on:click={login}>Login</button>
</div>

<style>
	* {
		font-family: 'Lato', sans-serif;
	}
	.form {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		background-color: #c7ccdb;
		padding: 2rem;
		border-radius: 1rem;
	}
	h1 {
		text-align: center;
		color: #2a324b;
		font-size: 1.5rem;
	}
	input {
		padding: 0.5rem;
		border: 1px solid #2a324b;
		border-radius: 0.5rem;
		background-color: #e1e5ee;
	}

	button {
		padding: 0.5rem;
		border: 1px solid #2a324b;
		border-radius: 0.5rem;
		background-color: #e1e5ee;
		cursor: pointer;
	}
</style>
