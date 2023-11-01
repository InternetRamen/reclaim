<script lang="ts">
	interface historicalEntry {
		start: number;
		end: number;
	}

	import { onMount } from 'svelte';
	import Complete from '$lib/Complete.svelte';
	import JSConfetti from 'js-confetti';
	import { DateTime, Duration } from 'luxon';

	let isStart: boolean = true;
	let goal: string = 'loading';
	let jsConfetti: any;
	let timeBetween: number;
	let goalInput = '';
	let startTime: number = 0;
	let history: historicalEntry[] = [];
	let continueTime: number = 0;
	let previousDay = DateTime.fromMillis(Date.now()).minus({ days: 1 }).toMillis();
	$: timeLeftInDay = Duration.fromMillis(getDayEnd(Date.now()) - Date.now());
	$: timeBetween = Date.now() - startTime;
	$: percentageStyle = `--value:${
		(timeBetween / (parseFloat(goal) * 8.64e7)) * 100
	}; --size:12rem;`;

	onMount(() => {
		isStart = window.localStorage.getItem('Start Time') === null;
		jsConfetti = new JSConfetti();
		goal = window.localStorage.getItem('goal') ?? '0';
		startTime = parseInt(window.localStorage.getItem('Start Time') ?? '0');

		continueTime = parseInt(window.localStorage.getItem('Continue Time') ?? '0');
		history = JSON.parse(window.localStorage.getItem('History') ?? '[]');
		if (getDayStart(continueTime) < getDayStart(previousDay)) {
			alert('Oh no! You missed a day. Try again! You can do this.');
			handleRestartClick();
		}
		setInterval(() => {
			timeLeftInDay = Duration.fromMillis(getDayEnd(Date.now()) - Date.now());
		}, 1000);
	});

	function getDayStart(ms: number) {
		const date = DateTime.fromMillis(ms);
		return date.startOf('day').toMillis();
	}

	function getDayEnd(ms: number) {
		const date = DateTime.fromMillis(ms);
		return date.endOf('day').toMillis();
	}

	function handleStartClick() {
		window.localStorage.setItem('Start Time', Date.now().toString());
		window.localStorage.setItem('Continue Time', Date.now().toString());
		startTime = Date.now();
		continueTime = Date.now();
		isStart = false;
	}

	function handleRestartClick() {
		const hEntry = {
			start: getDayStart(startTime),
			end: getDayStart(Date.now())
		};
		history = [hEntry, ...history];
		window.localStorage.setItem('History', JSON.stringify(history));
		window.localStorage.setItem('Start Time', Date.now().toString());
		window.localStorage.setItem('Continue Time', Date.now().toString());
		startTime = Date.now();
		continueTime = Date.now();
	}

	function handleContinueClick() {
		jsConfetti.addConfetti();
		window.localStorage.setItem('Continue Time', Date.now().toString());
		continueTime = Date.now();
	}

	function openModal() {
		let modal: HTMLDialogElement = document.getElementById('my_modal_2') as HTMLDialogElement;
		modal.showModal();
	}

	function closeModal() {
		let modal: HTMLDialogElement = document.getElementById('my_modal_2') as HTMLDialogElement;
		modal.close();
	}

	function setGoal() {
		window.localStorage.setItem('goal', parseFloat(goalInput).toString());
		goal = goalInput;
		closeModal();
	}
</script>

<section class="w-screen py-24">
	<div class="mx-auto w-11/12 max-w-3xl flex flex-col items-center gap-8">
		<h1 class="text-5xl text-center uppercase font-bold text-neutral">Reclaim</h1>
		<div
			class="radial-progress text-2xl text-center font-bold text-secondary"
			style={percentageStyle}
		>
			<p>Goal: {goal} Days</p>
		</div>
		<p class="text-md font-bold">
			Time left today: {timeLeftInDay.toFormat('hh:mm:ss')} • Day {Duration.fromMillis(
				Date.now() - getDayStart(startTime)
			)
				.normalize()
				.shiftTo('day')
				.toFormat('d')}
		</p>
		<div class="space-x-4">
			{#if isStart}
				<button class="btn btn-primary" on:click={handleStartClick}>Start</button>
			{:else}
				<button class="btn btn-primary" on:click={handleRestartClick}>Restart</button>
				<button
					class={`btn ${
						continueTime < getDayEnd(Date.now()) && continueTime > getDayStart(Date.now())
							? 'btn-disabled'
							: 'btn-accent'
					}`}
					on:click={handleContinueClick}>Renew</button
				>
			{/if}
		</div>
		<button class="btn btn-secondary" on:click={openModal}>Set Goal</button>
		<dialog id="my_modal_2" class="modal">
			<div class="modal-box">
				<h3 class="font-bold text-lg">Set a goal</h3>
				<div class="space-x-2 mt-4">
					<input
						placeholder="# of days"
						class="input input-bordered w-full max-w-xs"
						type="text"
						bind:value={goalInput}
					/>
					<button class="btn btn-primary" on:click={setGoal}>Submit</button>
				</div>
			</div>
			<form method="dialog" class="modal-backdrop">
				<button>close</button>
			</form>
		</dialog>
		<div class="overflow-x-auto">
			<table class="table">
				<!-- head -->
				<thead>
					<tr>
						<th>Start Date</th>
						<th>End Date</th>
						<th>Streak</th>
					</tr>
				</thead>
				<tbody>
					{#each history as { start, end }, i}
						<tr>
							<td>{DateTime.fromMillis(start).toLocaleString()}</td>
							<td>{DateTime.fromMillis(end).toLocaleString()}</td>
							<td
								>{Duration.fromMillis(Math.floor((end - start) / 60000) * 60000)
									.shiftTo('days')
									.toHuman()}</td
							>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	</div>
</section>
