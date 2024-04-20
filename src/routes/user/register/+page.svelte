<script>
	let name = '';
	let password = '';
	let password2 = '';

	function register() {
		if (password !== password2) {
			alert('Passwords do not match');
			return;
		}
		let success = fetch('http://localhost:8921/register', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				login: name,
				password: password
			})
		}).then((response) => response.json());
		if (success) {
			console.log(success);
			if (success.error) {
				alert(success.error);
				return;
			} else {
				alert('Registered successfully');
				window.location.href = '/user/login';
			}
		}
	}
</script>

<div class="form">
	<h1>Register</h1>
	<input type="name" bind:value={name} id="name" name="name" placeholder="Name" required />
	<input
		type="password"
		bind:value={password}
		id="password"
		name="password"
		placeholder="Password"
		required
	/>
	<input
		type="password"
		bind:value={password2}
		id="password2"
		name="password2"
		placeholder="Confirm Password"
		required
	/>
	<button on:click={register}>Register</button>
</div>

<style>
	.form {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		background-color: #c7ccdb;
		padding: 2rem;
		border-radius: 1rem;
	}
	* {
		font-family: 'Lato', sans-serif;
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
