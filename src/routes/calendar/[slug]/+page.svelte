<script>
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	let tasks_temp = [];
	let tasks = [[], [], [], [], [], [], []];
	let loaded = false;
	let days = [];
	let today = new Date();
	let week = 0;
	let day = 0;
	let taskName = '';
	let taskStart = '';
	let taskEnd = '';
	let taskRepeat = false;
	let adding = false;
	let generating_ai = false;
	let prompt = '';
	let ARLcookie = '';
	onMount(async () => {
		ARLcookie = getCookie('ARL');
		const res = await fetch(`http://localhost:8921/week/${$page.params.slug}?resource_key=${ARLcookie}`);
		const data = await res.json();
		const day_start = new Date(2024, 0, 1 + ($page.params.slug - 1) * 7);
		for (let i = 0; i < 7; i++) {
			let day = new Date(day_start);
			day.setDate(day.getDate() + i);
			let day_s = day.toLocaleDateString('en-UK', {
				weekday: 'long',
				day: 'numeric',
				month: 'long',
				year: 'numeric'
			});
			if (
				day_s ==
				today.toLocaleDateString('en-UK', {
					weekday: 'long',
					day: 'numeric',
					month: 'long',
					year: 'numeric'
				})
			) {
				days.push(
					day.toLocaleDateString('en-UK', {
						weekday: 'long',
						day: 'numeric',
						month: 'long',
						year: 'numeric'
					}) + ' (Today)'
				);
			} else {
				days.push(
					day.toLocaleDateString('en-UK', {
						weekday: 'long',
						day: 'numeric',
						month: 'long',
						year: 'numeric'
					})
				);
			}
		}

		function getCookie(name) {
			const cookies = document.cookie.split(';');
			for (let i = 0; i < cookies.length; i++) {
				const cookie = cookies[i].trim();
				if (cookie.startsWith(name + '=')) {
					return cookie.substring(name.length + 1);
				}
			}
			return null;
		}

		data.tasks.forEach((task) => {
			let start_date = new Date(Date.parse(task.task_start));
			let end_date = new Date(Date.parse(task.task_end));

			start_date.setMinutes(start_date.getMinutes() + start_date.getTimezoneOffset());
			end_date.setMinutes(end_date.getMinutes() + end_date.getTimezoneOffset());

			tasks_temp.push({
				start: start_date.getTime(),
				end: end_date.getTime(),
				title: task.task_name,
				description: task.task_description
			});
		});

		for (let i = 0; i < 7; i++) {
			let day_end = new Date(day_start);
			day_end.setDate(day_end.getDate() + 1);

			let tasks_temp_temp = tasks_temp.filter((task) => {
				return task.end >= day_start && task.start <= day_end;
			});

			tasks_temp_temp.forEach((task) => {
				let start_date = new Date(task.start);
				let end_date = new Date(task.end);

				let start_minutes = start_date.getHours() * 60 + start_date.getMinutes();
				let end_minutes = end_date.getHours() * 60 + end_date.getMinutes();

				// Adjust start and end minutes if the task doesn't start or end on the current day
				if (start_date < day_start) {
					start_minutes = 0;
				}
				if (end_date > day_end) {
					end_minutes = 1440;
				}

				console.log(start_minutes, end_minutes);
				tasks[i].push({
					start: start_minutes,
					end: end_minutes - start_minutes,
					title: task.title
				});
			});

			day_start.setDate(day_start.getDate() + 1);
		}
		fetch('http://localhost:8921/today')
			.then((response) => response.json())
			.then((data) => {
				today = new Date(2024, 0, 1 + (data.week - 1) * 7 + data.day);
				week = data.week;
				day = data.day;
			});
		console.log(today);
		loaded = true;
	});
	let addTask = () => {
		let start_date = new Date(Date.parse(taskStart));
		let end_date = new Date(Date.parse(taskEnd));
		start_date.setHours(start_date.getHours() + 2);
		end_date.setHours(end_date.getHours() + 2);
		fetch('http://localhost:8921/create-task', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				task_name: taskName,
				task_start: start_date,
				task_end: end_date,
				task_repeat: taskRepeat,
				task_importance: 0,
				resource_key: ARLcookie,
				task_description: null,
				task_status: true,
				task_physical: 0,
				task_mental: 0,
				task_flexible: false
			})
		});
		adding = false;
		// refresh page
		location.reload();
	};
	let generateAiTask = () => {
		generating_ai = true;
		fetch('http://localhost:8921/text-input', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				text: prompt,
				resource_key: ARLcookie
			})
		})
			.then((response) => response.json())
			.then((data) => {
				console.log(data);
				generating_ai = false;
				location.reload();
			})
			.catch((error) => {
				console.error('Error:', error);
				generating_ai = false;
			});
	};
</script>

{#if loaded}
	<!-- Add this line -->
	<div class="main">
		{#key $page.params.slug}
			<div class="calendar">
				<h1 class="cc">CleverCal</h1>
				<div class="header">
					{#each days as day}
						{#if day.includes('Today')}
							<div style="font-weight: bold">{day}</div>
						{:else}
							<div>{day}</div>
						{/if}
					{/each}
				</div>
				<div class="container">
					<div class="days">
						{#each Array.from({ length: 7 }, (_, i) => i + 1) as i}
							<div class="day">
								<div class="hours-container">
									{#if i == 1}
										{#each Array.from({ length: 24 }, (_, d) => d) as d}
											<div class="hours">
												{d.toLocaleString('en-US', {
													minimumIntegerDigits: 2,
													useGrouping: false
												})}:00
											</div>
										{/each}
									{/if}
									{#if i != 1}
										{#each Array.from({ length: 24 }, (_, d) => d) as d}
											<div class="hours" style="color: #e1e5ee;">
												{d.toLocaleString('en-US', {
													minimumIntegerDigits: 2,
													useGrouping: false
												})}:00
											</div>
										{/each}
									{/if}
								</div>
								<div class="tasks">
									{#each tasks[i - 1] as task}
										<div
											class="task"
											style="top: calc({(task.start / (24 * 60)) * 100}%); height: calc(
											{(task.end / (24 * 60)) * 100}%);"
											title={task.description}>
										
											{task.title}
										</div>
									{/each}
								</div>
							</div>
						{/each}
					</div>
				</div>
				<div class="buttons">
					<div class="prev">
						<a href={`/calendar/${Number($page.params.slug) - 1}`} data-sveltekit-reload
							>Previous Week</a
						>
					</div>
					<div class="next">
						<a href={`/calendar/${Number($page.params.slug) + 1}`} data-sveltekit-reload
							>Next Week</a
						>
					</div>
					<div class="add">
						<button
							on:click={() => {
								adding = !adding;
							}}>Add Task Manually</button
						>
					</div>
				</div>
				{#if adding}
					<div class="add-task">
						<form on:submit|preventDefault={addTask}>
							<input type="text" bind:value={taskName} placeholder="Task Name" />
							<input type="datetime-local" bind:value={taskStart} />
							<input type="datetime-local" bind:value={taskEnd} />
							<input type="checkbox" bind:checked={taskRepeat} />
							<button type="submit">Add Task</button>
						</form>
					</div>
				{/if}
			</div>
		{/key}
		<div class="textbox">
			<h1>AI Text Input</h1>
			<p>
				Enter your prompt here and the AI will generate a task for you. You might want to use
				concise and on-point language to avoid any misunderstandings
			</p>
			<textarea bind:value={prompt}></textarea>
			{#if generating_ai}
				<p>Generating...</p>
			{:else}
				<button on:click={generateAiTask}>Generate Task</button>
			{/if}
		</div>
	</div>
{/if}

<!-- Add this line -->

<style>
	* {
		font-family: 'Lato', sans-serif;
	}
	.cc {
		font-size: 2rem;
		padding-left: 2rem;
	}
	.main {
		padding: 1rem;
		display: flex;
		gap: 1rem;
	}
	.container {
		position: relative;
	}
	.calendar {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		background-color: #c7ccdb;
		padding: 1rem;
		padding-top: 1rem;
		border-radius: 1rem;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
	}

	.header {
		display: grid;
		grid-template-columns: repeat(7, 0.7fr);
		gap: 1rem;
	}
	.header div {
		width: 70%;
	}
	.days {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		border-radius: 1rem;
	}

	.hours-container {
		display: grid;
		grid-template-rows: repeat(24, 1fr);
	}

	.hours {
		display: flex;
		justify-content: left;
		align-items: flex-start;
		background-color: #e1e5ee;
		border-top: 1px solid #c7ccdb;
		border-right: 3px solid #c7ccdb;
		padding-bottom: 0.4rem;
		padding-top: 0;
		padding-left: 0.3rem;
		font-weight: 300;
	}
	.tasks {
		position: absolute;
		top: 0;
		height: 100%;
		width: calc(100% / 13.4);
	}
	.task {
		position: absolute;
		background-color: #f7c59f;
		width: 100%;
		overflow: hidden;
		font-style: italic;
		display: flex;
		justify-content: center;
		align-items: flex-end;
	}
	.buttons {
		display: flex;
		justify-content: flex-start;
		gap: 1rem;
		align-items: center;
	}
	a {
		all: unset;
		background-color: #e1e5ee;
		padding: 0.5rem;
		border-radius: 0.5rem;
		cursor: pointer;
	}
	button {
		all: unset;
		background-color: #e1e5ee;
		padding: 0.5rem;
		border-radius: 0.5rem;
		cursor: pointer;
	}
	a:hover {
		text-decoration: underline;
	}
	.textbox {
		background-color: #c7ccdb;
		padding: 1rem;
		border-radius: 1rem;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
	}
	textarea {
		width: 100%;
		height: 10rem;
	}
	.add-task {
		background-color: #e1e5ee;
		padding: 1rem;
		border-radius: 1rem;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
	}
</style>
