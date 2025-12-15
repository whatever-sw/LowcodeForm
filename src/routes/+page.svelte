<script>
	import NodeGraphEditor from '$lib/components/NodeGraphEditor.svelte';
	import FormDisplay from '$lib/components/FormDisplay.svelte';

	let nodes = $state([]);
	let connections = $state([]);
	let formData = $state({});
</script>

<div class="min-h-screen bg-gray-50">
	<header class="bg-white shadow-sm border-b border-gray-200">
		<div class="max-w-7xl mx-auto px-4 py-4">
			<h1 class="text-2xl font-bold text-gray-900">Low-Code Form Builder</h1>
			<p class="text-sm text-gray-600 mt-1">
				Create custom forms with node-based visual programming
			</p>
		</div>
	</header>

	<main class="max-w-7xl mx-auto px-4 py-6">
		<div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
			<!-- Node Graph Editor -->
			<div class="bg-white rounded-lg shadow-md overflow-hidden">
				<div class="bg-gray-800 text-white px-4 py-3 font-semibold">
					Node Graph Editor
				</div>
				<NodeGraphEditor bind:nodes bind:connections />
			</div>

			<!-- Form Display -->
			<div class="bg-white rounded-lg shadow-md overflow-hidden">
				<div class="bg-blue-600 text-white px-4 py-3 font-semibold">Form Preview</div>
				<FormDisplay {nodes} {connections} bind:formData />
			</div>
		</div>
	</main>
</div>
