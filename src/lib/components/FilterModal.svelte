<script>
	import { Modal, Spinner, Select, Button, Input, Helper, Alert } from 'flowbite-svelte';
	import { goto } from '$app/navigation';

	export let open = false;
	export let onClose = () => {};

	let selectedRover = 'all';
	let selectedCamera = 'all';
	let searchType = 'latest';
	let solValue = '';
	let startDate = '';
	let endDate = '';
	let error = '';

	const MAX_DATE_RANGE_DAYS = 7; // Limit to 7 days to prevent too many requests

	const roverDates = {
		perseverance: { landed: '2021-02-18', max_date: new Date().toISOString().split('T')[0] },
		curiosity: { landed: '2012-08-06', max_date: new Date().toISOString().split('T')[0] },
		opportunity: { landed: '2004-01-25', max_date: '2018-06-11' },
		spirit: { landed: '2004-01-04', max_date: '2011-03-22' }
	};

	const rovers = [
		{ value: 'all', name: 'All Rovers' },
		{
			value: 'perseverance',
			name: `Perseverance (${formatDate(roverDates.perseverance.landed)} - Present)`
		},
		{
			value: 'curiosity',
			name: `Curiosity (${formatDate(roverDates.curiosity.landed)} - Present)`
		},
		{
			value: 'opportunity',
			name: `Opportunity (${formatDate(roverDates.opportunity.landed)} - ${formatDate(roverDates.opportunity.max_date)})`
		},
		{
			value: 'spirit',
			name: `Spirit (${formatDate(roverDates.spirit.landed)} - ${formatDate(roverDates.spirit.max_date)})`
		}
	];

	const roverCameras = {
		perseverance: [
			{ value: 'EDL_RDCAM', name: 'Rover Down-Look Camera' },
			{ value: 'EDL_DDCAM', name: 'Descent Stage Down-Look Camera' },
			{ value: 'EDL_PUCAM1', name: 'Parachute Up-Look Camera A' },
			{ value: 'EDL_PUCAM2', name: 'Parachute Up-Look Camera B' },
			{ value: 'FHAZ', name: 'Front Hazard Avoidance Camera' },
			{ value: 'RHAZ', name: 'Rear Hazard Avoidance Camera' },
			{ value: 'NAVCAM_LEFT', name: 'Navigation Camera - Left' },
			{ value: 'NAVCAM_RIGHT', name: 'Navigation Camera - Right' },
			{ value: 'MCZ_LEFT', name: 'Mastcam-Z Left' },
			{ value: 'MCZ_RIGHT', name: 'Mastcam-Z Right' },
			{ value: 'SKYCAM', name: 'MEDA Skycam' },
			{ value: 'SHERLOC_WATSON', name: 'SHERLOC WATSON Camera' },
			{ value: 'SUPERCAM_RMI', name: 'SuperCam Remote Micro-Imager' }
		],
		curiosity: [
			{ value: 'FHAZ', name: 'Front Hazard Avoidance Camera' },
			{ value: 'RHAZ', name: 'Rear Hazard Avoidance Camera' },
			{ value: 'MAST', name: 'Mast Camera' },
			{ value: 'CHEMCAM_RMI', name: 'Chemistry and Camera Complex (RMI)' },
			{ value: 'MAHLI', name: 'Mars Hand Lens Imager' },
			{ value: 'MARDI', name: 'Mars Descent Imager' },
			{ value: 'NAVCAM', name: 'Navigation Camera' },
			{ value: 'NAV_LEFT_A', name: 'Navigation Camera - Left A' },
			{ value: 'NAV_LEFT_B', name: 'Navigation Camera - Left B' },
			{ value: 'NAV_RIGHT_A', name: 'Navigation Camera - Right A' },
			{ value: 'NAV_RIGHT_B', name: 'Navigation Camera - Right B' }
		],
		opportunity: [
			{ value: 'FHAZ', name: 'Front Hazard Avoidance Camera' },
			{ value: 'RHAZ', name: 'Rear Hazard Avoidance Camera' },
			{ value: 'NAVCAM', name: 'Navigation Camera' },
			{ value: 'PANCAM', name: 'Panoramic Camera' },
			{ value: 'MINITES', name: 'Miniature Thermal Emission Spectrometer' }
		],
		spirit: [
			{ value: 'FHAZ', name: 'Front Hazard Avoidance Camera' },
			{ value: 'RHAZ', name: 'Rear Hazard Avoidance Camera' },
			{ value: 'NAVCAM', name: 'Navigation Camera' },
			{ value: 'PANCAM', name: 'Panoramic Camera' },
			{ value: 'MINITES', name: 'Miniature Thermal Emission Spectrometer' }
		]
	};

	const searchTypes = [
		{
			value: 'earth_date',
			name: 'Earth Date Range',
			description: 'Search for photos between two Earth dates'
		},
		{
			value: 'sol',
			name: 'Martian Sol',
			description: 'Search by Martian day (Sol 0 = landing day)'
		}
	];

	// Base date constraints from rover dates
	$: dateConstraints =
		selectedRover === 'all'
			? { min: '2004-01-04', max: new Date().toISOString().split('T')[0] }
			: selectedRover !== ''
				? { min: roverDates[selectedRover].landed, max: roverDates[selectedRover].max_date }
				: { min: '', max: '' };

	// Only constrain the end date based on start date selection
	$: endDateConstraints = startDate
		? {
				min: startDate,
				max: new Date(new Date(startDate).getTime() + MAX_DATE_RANGE_DAYS * 24 * 60 * 60 * 1000)
					.toISOString()
					.split('T')[0]
			}
		: dateConstraints;

	// Reactive statement to generate available cameras based on selected rover
	$: availableCameras =
		selectedRover === 'all'
			? [{ value: 'all', name: 'All Cameras' }].concat(
					Object.entries(roverCameras)
						.reduce((allCameras, [rover, cameras]) => {
							cameras.forEach((camera) => {
								const existing = allCameras.find((c) => c.value === camera.value);
								if (existing) {
									existing.rovers = [...existing.rovers, rover];
								} else {
									allCameras.push({
										...camera,
										rovers: [rover],
										name: `${camera.name} (${rover})`
									});
								}
							});
							return allCameras;
						}, [])
						.map((camera) => ({
							...camera,
							name: `${camera.name.split(' (')[0]} (${camera.rovers.map((r) => r.charAt(0).toUpperCase() + r.slice(1)).join(', ')})`
						}))
				)
			: [{ value: 'all', name: 'All Cameras' }].concat(roverCameras[selectedRover] || []);

	// Reset camera selection when rover changes
	$: if (selectedRover) {
		const cameraExists = availableCameras.some((cam) => cam.value === selectedCamera);
		if (!cameraExists) {
			selectedCamera = 'all';
		}
	}

	function formatDate(dateString) {
		if (!dateString) return '';
		const date = new Date(dateString);
		return new Intl.DateTimeFormat('en-US', {
			month: '2-digit',
			day: '2-digit',
			year: 'numeric'
		}).format(date);
	}

	// Clear end date when start date changes
	$: if (startDate) {
		// Only clear end date if it's outside the valid range
		const endDateValue = new Date(endDate);
		const maxDate = new Date(endDateConstraints.max);
		const minDate = new Date(endDateConstraints.min);
		if (endDateValue > maxDate || endDateValue < minDate) {
			endDate = '';
		}
	}

	function validateSearch() {
		error = '';

		if (searchType === 'earth_date' && startDate && !endDate) {
			error = 'Please select an end date';
			return false;
		}

		if (searchType === 'sol' && solValue !== '' && parseInt(solValue) < 0) {
			error = 'Sol number must be 0 or greater';
			return false;
		}

		if (searchType === 'earth_date') {
			if (!startDate && !endDate) {
				return true;
			}
			if (!startDate || !endDate) {
				error = 'Please select both start and end dates';
				return false;
			}
			if (startDate > endDate) {
				error = 'Start date must be before end date';
				return false;
			}

			// Check date range
			const start = new Date(startDate);
			const end = new Date(endDate);
			const daysDiff = Math.ceil((end - start) / (1000 * 60 * 60 * 24));

			if (daysDiff > MAX_DATE_RANGE_DAYS) {
				error = `Please select a date range of ${MAX_DATE_RANGE_DAYS} days or less`;
				return false;
			}
		}

		return true;
	}

	function handleSearch() {
		if (!validateSearch()) return;

		const params = new URLSearchParams();
		if (selectedRover !== 'all') {
			params.set('rover', selectedRover.toLowerCase());
		}
		if (selectedCamera !== 'all') {
			params.set('camera', selectedCamera);
		}
		if (searchType === 'sol' && solValue !== '') {
			params.set('sol', solValue);
		} else if (searchType === 'earth_date' && startDate && endDate) {
			params.set('start_date', startDate);
			params.set('end_date', endDate);
		}

		goto(`/photos?${params.toString()}`);
		onClose();
	}
</script>

<Modal bind:open size="lg" outsideclose on:close={onClose}>
	<div class="space-y-6">
		<h3 class="text-xl font-medium text-gray-900 dark:text-white">Filter Mars Rover Photos</h3>

		<div class="space-y-4">
			<div>
				<label
					for="rover-select"
					class="mb-2 block text-sm font-medium text-gray-900 dark:text-white"
				>
					Select Rover
				</label>
				<Select id="rover-select" items={rovers} bind:value={selectedRover} class="capitalize" />
				{#if selectedRover !== 'all' && selectedRover !== ''}
					<Helper class="mt-2">Select a camera and search type to continue</Helper>
				{/if}
			</div>

			<div>
				<label
					for="camera-select"
					class="mb-2 block text-sm font-medium text-gray-900 dark:text-white"
				>
					Select Camera
				</label>
				<Select id="camera-select" items={availableCameras} bind:value={selectedCamera} />
			</div>

			<div>
				<label
					for="search-type"
					class="mb-2 block text-sm font-medium text-gray-900 dark:text-white"
				>
					Search Type
				</label>
				<Select id="search-type" bind:value={searchType}>
					<option value="latest">Most Recent Photos</option>
					<option value="earth_date">Date Range Search</option>
					<option value="sol">Martian Sol Search</option>
				</Select>
				<Helper>
					{#if searchType === 'latest'}
						Shows the most recent photos available from each selected rover
					{:else if searchType === 'earth_date'}
						Search for photos taken between specific Earth dates
					{:else}
						Search by Martian day (Sol 0 = landing day)
					{/if}
				</Helper>
			</div>

			{#if searchType === 'earth_date'}
				<div class="space-y-4">
					<Alert color="info" class="mb-4">
						<span class="font-medium">Note:</span>
						Select dates up to {MAX_DATE_RANGE_DAYS} days apart.
					</Alert>
					<div>
						<label
							for="start-date"
							class="mb-2 block text-sm font-medium text-gray-900 dark:text-white"
						>
							Start Date
						</label>
						<Input
							id="start-date"
							type="date"
							bind:value={startDate}
							min={dateConstraints.min}
							max={dateConstraints.max}
						/>
					</div>
					<div>
						<label
							for="end-date"
							class="mb-2 block text-sm font-medium text-gray-900 dark:text-white"
						>
							End Date
						</label>
						<Input
							id="end-date"
							type="date"
							bind:value={endDate}
							min={endDateConstraints.min}
							max={endDateConstraints.max}
							required={!!startDate}
						/>
						{#if startDate}
							<Helper>
								Select an end date within {MAX_DATE_RANGE_DAYS} days of the start date
							</Helper>
						{/if}
					</div>
				</div>
			{:else if searchType === 'sol'}
				<div>
					<label
						for="sol-input"
						class="mb-2 block text-sm font-medium text-gray-900 dark:text-white"
					>
						Martian Sol
					</label>
					<Input
						id="sol-input"
						type="number"
						bind:value={solValue}
						min="0"
						placeholder="Enter sol number (e.g., 1000)"
					/>
				</div>
			{/if}

			<div class="mt-6 rounded-lg bg-gray-50 p-4 dark:bg-gray-800">
				<p class="font-medium text-gray-900 dark:text-white">You will see:</p>
				<p class="text-gray-600 dark:text-gray-400">
					{selectedRover === 'all'
						? 'All Rovers'
						: selectedRover.charAt(0).toUpperCase() + selectedRover.slice(1)}
					• {selectedCamera === 'all' ? 'All Cameras' : `${selectedCamera} Camera`}
					{#if searchType === 'latest'}
						• Most recent available photos
					{:else if searchType === 'earth_date' && startDate && endDate}
						• Photos from {startDate} to {endDate}
					{:else if searchType === 'sol' && solValue !== ''}
						• Photos from Sol {solValue}
					{/if}
				</p>
			</div>

			{#if error}
				<p class="text-sm text-red-600 dark:text-red-500">{error}</p>
			{/if}
		</div>

		<div class="flex justify-end space-x-4">
			<Button color="alternative" on:click={onClose}>Cancel</Button>
			<Button color="primary" on:click={handleSearch}>Search Photos</Button>
		</div>
	</div>
</Modal>
