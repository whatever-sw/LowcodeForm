<script>
	let { nodes, connections, formData = $bindable({}) } = $props();

	let validationErrors = $state({});
	let computedValues = $state({});

	// Get input field nodes (text, number, email)
	const inputNodes = $derived(nodes.filter((n) => ['text', 'number', 'email'].includes(n.type)));

	// Get computed field nodes
	const computedNodes = $derived(nodes.filter((n) => n.type === 'computed'));

	// Get validation nodes
	const validationNodes = $derived(nodes.filter((n) => n.type === 'validation'));

	// Initialize form data for all input nodes
	$effect(() => {
		inputNodes.forEach((node) => {
			if (!(node.id in formData)) {
				formData[node.id] = '';
			}
		});
	});

	// Compute derived values
	$effect(() => {
		computedNodes.forEach((node) => {
			if (!node.config.expression) {
				computedValues[node.id] = '';
				return;
			}

			try {
				// Get all connected input nodes
				const connectedNodeIds = connections
					.filter((c) => c.to === node.id)
					.map((c) => c.from);

				// Create context with field names and values
				const context = {};
				connectedNodeIds.forEach((nodeId) => {
					const sourceNode = nodes.find((n) => n.id === nodeId);
					if (sourceNode) {
						const fieldName = sourceNode.label || sourceNode.id;
						context[fieldName] = formData[nodeId] || computedValues[nodeId] || '';
					}
				});

				// Evaluate the expression
				const expression = node.config.expression;
				// Create a function that evaluates the expression with the context
				const fn = new Function(...Object.keys(context), `return ${expression}`);
				const result = fn(...Object.values(context));
				computedValues[node.id] = result;
			} catch (e) {
				computedValues[node.id] = `Error: ${e.message}`;
			}
		});
	});

	// Run validations
	$effect(() => {
		const errors = {};
		validationNodes.forEach((node) => {
			if (!node.config.rule) return;

			try {
				// Get connected input node
				const connectedNode = connections.find((c) => c.to === node.id);
				if (!connectedNode) return;

				const sourceNode = nodes.find((n) => n.id === connectedNode.from);
				if (!sourceNode) return;

				const value = formData[sourceNode.id] || computedValues[sourceNode.id] || '';

				// Evaluate validation rule
				const fn = new Function('value', `return ${node.config.rule}`);
				const isValid = fn(value);

				if (!isValid) {
					errors[sourceNode.id] = node.label || 'Validation failed';
				}
			} catch (e) {
				errors[node.id] = `Validation error: ${e.message}`;
			}
		});
		validationErrors = errors;
	});

	function handleSubmit(event) {
		event.preventDefault();
		
		// Check if there are any validation errors
		if (Object.keys(validationErrors).length > 0) {
			alert('Please fix validation errors before submitting');
			return;
		}

		const submissionData = {
			...formData,
			computed: computedValues
		};
		
		console.log('Form submitted:', submissionData);
		alert('Form submitted! Check console for data.');
	}

	function resetForm() {
		formData = {};
		validationErrors = {};
		computedValues = {};
	}
</script>

<div class="p-6 h-[600px] overflow-y-auto">
	{#if inputNodes.length === 0 && computedNodes.length === 0}
		<div class="text-center text-gray-500 mt-20">
			<p class="text-lg mb-2">No form fields yet</p>
			<p class="text-sm">Add nodes in the editor to create your form</p>
		</div>
	{:else}
		<form onsubmit={handleSubmit} class="space-y-4">
			<!-- Input Fields -->
			{#each inputNodes as node}
				<div class="form-field">
					<label for={node.id} class="block text-sm font-medium text-gray-700 mb-1">
						{node.label || node.id}
						{#if node.type === 'email'}
							<span class="text-xs text-gray-500">(email)</span>
						{/if}
						{#if node.type === 'number'}
							<span class="text-xs text-gray-500">(number)</span>
						{/if}
					</label>

					{#if node.type === 'text'}
						<input
							id={node.id}
							type="text"
							bind:value={formData[node.id]}
							class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-transparent {validationErrors[
								node.id
							]
								? 'border-red-500'
								: ''}"
						/>
					{:else if node.type === 'number'}
						<input
							id={node.id}
							type="number"
							bind:value={formData[node.id]}
							class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-transparent {validationErrors[
								node.id
							]
								? 'border-red-500'
								: ''}"
						/>
					{:else if node.type === 'email'}
						<input
							id={node.id}
							type="email"
							bind:value={formData[node.id]}
							class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-transparent {validationErrors[
								node.id
							]
								? 'border-red-500'
								: ''}"
						/>
					{/if}

					{#if validationErrors[node.id]}
						<p class="text-xs text-red-600 mt-1">{validationErrors[node.id]}</p>
					{/if}
				</div>
			{/each}

			<!-- Computed Fields -->
			{#if computedNodes.length > 0}
				<div class="border-t border-gray-200 pt-4 mt-4">
					<h3 class="text-sm font-semibold text-gray-700 mb-3">Computed Fields</h3>
					{#each computedNodes as node}
						<div class="form-field mb-3">
							<label class="block text-sm font-medium text-gray-700 mb-1">
								{node.label || node.id}
								<span class="text-xs text-gray-500">(computed)</span>
							</label>
							<div
								class="w-full px-3 py-2 border border-gray-300 rounded-md bg-gray-50 text-gray-700"
							>
								{computedValues[node.id] !== undefined ? computedValues[node.id] : '—'}
							</div>
							{#if node.config.expression}
								<p class="text-xs text-gray-500 mt-1">Expression: {node.config.expression}</p>
							{/if}
						</div>
					{/each}
				</div>
			{/if}

			<!-- Form Actions -->
			<div class="flex gap-3 pt-4 border-t border-gray-200">
				<button
					type="submit"
					class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition"
				>
					Submit Form
				</button>
				<button
					type="button"
					onclick={resetForm}
					class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 focus:ring-2 focus:ring-gray-500 focus:ring-offset-2 transition"
				>
					Reset
				</button>
			</div>

			<!-- Debug Info -->
			<div class="border-t border-gray-200 pt-4 mt-4">
				<details class="text-xs">
					<summary class="cursor-pointer text-gray-600 font-medium">Debug Info</summary>
					<div class="mt-2 space-y-2">
						<div>
							<strong>Form Data:</strong>
							<pre class="bg-gray-100 p-2 rounded mt-1 overflow-x-auto">{JSON.stringify(
									formData,
									null,
									2
								)}</pre>
						</div>
						<div>
							<strong>Computed Values:</strong>
							<pre class="bg-gray-100 p-2 rounded mt-1 overflow-x-auto">{JSON.stringify(
									computedValues,
									null,
									2
								)}</pre>
						</div>
						{#if Object.keys(validationErrors).length > 0}
							<div>
								<strong>Validation Errors:</strong>
								<pre class="bg-red-50 p-2 rounded mt-1 overflow-x-auto">{JSON.stringify(
										validationErrors,
										null,
										2
									)}</pre>
							</div>
						{/if}
					</div>
				</details>
			</div>
		</form>
	{/if}
</div>
