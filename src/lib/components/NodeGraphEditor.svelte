<script>
	let { nodes = $bindable([]), connections = $bindable([]) } = $props();

	let canvas = $state(null);
	let selectedNode = $state(null);
	let connectingFrom = $state(null);
	let draggedNode = $state(null);
	let dragOffset = $state({ x: 0, y: 0 });
	let nextNodeId = $state(1);

	const nodeTypes = [
		{ type: 'text', label: 'Text Input', color: 'bg-green-100 border-green-400' },
		{ type: 'number', label: 'Number Input', color: 'bg-blue-100 border-blue-400' },
		{ type: 'email', label: 'Email Input', color: 'bg-purple-100 border-purple-400' },
		{ type: 'computed', label: 'Computed Field', color: 'bg-yellow-100 border-yellow-400' },
		{ type: 'validation', label: 'Validation', color: 'bg-red-100 border-red-400' }
	];

	function addNode(type) {
		const newNode = {
			id: `node-${nextNodeId++}`,
			type,
			x: 100,
			y: 50 + nodes.length * 80,
			label: '',
			config: type === 'computed' ? { expression: '' } : type === 'validation' ? { rule: '' } : {}
		};
		nodes = [...nodes, newNode];
		selectedNode = newNode;
	}

	function startDrag(event, node) {
		if (event.button !== 0) return; // Only left click
		draggedNode = node;
		const rect = canvas.getBoundingClientRect();
		dragOffset = {
			x: event.clientX - rect.left - node.x,
			y: event.clientY - rect.top - node.y
		};
		event.preventDefault();
	}

	function handleMouseMove(event) {
		if (!draggedNode) return;
		const rect = canvas.getBoundingClientRect();
		const newX = event.clientX - rect.left - dragOffset.x;
		const newY = event.clientY - rect.top - dragOffset.y;
		
		nodes = nodes.map((n) =>
			n.id === draggedNode.id ? { ...n, x: Math.max(0, newX), y: Math.max(0, newY) } : n
		);
	}

	function handleMouseUp() {
		draggedNode = null;
	}

	function startConnection(node) {
		connectingFrom = node;
	}

	function completeConnection(toNode) {
		if (connectingFrom && connectingFrom.id !== toNode.id) {
			const newConnection = {
				from: connectingFrom.id,
				to: toNode.id
			};
			// Check if connection already exists
			const exists = connections.some(
				(c) => c.from === newConnection.from && c.to === newConnection.to
			);
			if (!exists) {
				connections = [...connections, newConnection];
			}
		}
		connectingFrom = null;
	}

	function deleteNode(nodeId) {
		nodes = nodes.filter((n) => n.id !== nodeId);
		connections = connections.filter((c) => c.from !== nodeId && c.to !== nodeId);
		if (selectedNode?.id === nodeId) {
			selectedNode = null;
		}
	}

	function deleteConnection(connection) {
		connections = connections.filter(
			(c) => !(c.from === connection.from && c.to === connection.to)
		);
	}

	function getNodeColor(type) {
		return nodeTypes.find((nt) => nt.type === type)?.color || 'bg-gray-100 border-gray-400';
	}

	function getNodePosition(nodeId) {
		return nodes.find((n) => n.id === nodeId);
	}
</script>

<div class="h-[600px] flex">
	<!-- Toolbox -->
	<div class="w-48 bg-gray-100 border-r border-gray-300 p-4 overflow-y-auto">
		<h3 class="font-semibold text-sm mb-3 text-gray-700">Node Types</h3>
		<div class="space-y-2">
			{#each nodeTypes as nodeType}
				<button
					onclick={() => addNode(nodeType.type)}
					class="w-full text-left px-3 py-2 text-sm rounded border-2 {nodeType.color} hover:shadow transition"
				>
					{nodeType.label}
				</button>
			{/each}
		</div>
	</div>

	<!-- Canvas -->
	<div
		bind:this={canvas}
		onmousemove={handleMouseMove}
		onmouseup={handleMouseUp}
		class="flex-1 relative bg-white overflow-hidden"
		role="application"
		aria-label="Node graph canvas"
	>
		<svg class="absolute inset-0 w-full h-full pointer-events-none">
			{#each connections as connection}
				{@const fromNode = getNodePosition(connection.from)}
				{@const toNode = getNodePosition(connection.to)}
				{#if fromNode && toNode}
					<g>
						<line
							x1={fromNode.x + 80}
							y1={fromNode.y + 30}
							x2={toNode.x}
							y2={toNode.y + 30}
							stroke="#4B5563"
							stroke-width="2"
							marker-end="url(#arrowhead)"
						/>
						<circle
							cx={(fromNode.x + 80 + toNode.x) / 2}
							cy={(fromNode.y + 30 + toNode.y + 30) / 2}
							r="8"
							fill="red"
							class="cursor-pointer pointer-events-auto"
							onclick={() => deleteConnection(connection)}
						>
							<title>Click to delete connection</title>
						</circle>
					</g>
				{/if}
			{/each}
			<defs>
				<marker
					id="arrowhead"
					markerWidth="10"
					markerHeight="10"
					refX="9"
					refY="3"
					orient="auto"
				>
					<polygon points="0 0, 10 3, 0 6" fill="#4B5563" />
				</marker>
			</defs>
		</svg>

		{#each nodes as node}
			<div
				class="absolute w-40 p-2 rounded border-2 shadow-md cursor-move {getNodeColor(
					node.type
				)} {selectedNode?.id === node.id ? 'ring-2 ring-blue-500' : ''}"
				style="left: {node.x}px; top: {node.y}px;"
				onmousedown={(e) => startDrag(e, node)}
				onclick={() => (selectedNode = node)}
				role="button"
				tabindex="0"
			>
				<div class="text-xs font-semibold mb-1 text-gray-700">
					{nodeTypes.find((nt) => nt.type === node.type)?.label}
				</div>
				<input
					type="text"
					bind:value={node.label}
					placeholder="Field name"
					class="w-full text-xs px-1 py-0.5 border border-gray-300 rounded"
					onclick={(e) => e.stopPropagation()}
					onmousedown={(e) => e.stopPropagation()}
				/>
				<div class="flex justify-between mt-2">
					<button
						onclick={(e) => {
							e.stopPropagation();
							startConnection(node);
						}}
						class="text-xs bg-blue-500 text-white px-2 py-0.5 rounded hover:bg-blue-600"
						title="Start connection"
					>
						→
					</button>
					<button
						onclick={(e) => {
							e.stopPropagation();
							completeConnection(node);
						}}
						class="text-xs bg-green-500 text-white px-2 py-0.5 rounded hover:bg-green-600"
						title="Receive connection"
					>
						←
					</button>
					<button
						onclick={(e) => {
							e.stopPropagation();
							deleteNode(node.id);
						}}
						class="text-xs bg-red-500 text-white px-2 py-0.5 rounded hover:bg-red-600"
						title="Delete node"
					>
						×
					</button>
				</div>
			</div>
		{/each}

		{#if connectingFrom}
			<div class="absolute top-4 left-1/2 transform -translate-x-1/2 bg-blue-500 text-white px-4 py-2 rounded shadow-lg">
				Connecting from: {connectingFrom.label || connectingFrom.id}. Click "←" on target node.
			</div>
		{/if}
	</div>

	<!-- Properties Panel -->
	<div class="w-64 bg-gray-100 border-l border-gray-300 p-4 overflow-y-auto">
		<h3 class="font-semibold text-sm mb-3 text-gray-700">Properties</h3>
		{#if selectedNode}
			<div class="space-y-3">
				<div>
					<label class="block text-xs font-medium text-gray-700 mb-1">Node ID</label>
					<div class="text-xs text-gray-600">{selectedNode.id}</div>
				</div>
				<div>
					<label class="block text-xs font-medium text-gray-700 mb-1">Type</label>
					<div class="text-xs text-gray-600">{selectedNode.type}</div>
				</div>
				<div>
					<label class="block text-xs font-medium text-gray-700 mb-1">Label</label>
					<input
						type="text"
						bind:value={selectedNode.label}
						class="w-full text-xs px-2 py-1 border border-gray-300 rounded"
					/>
				</div>

				{#if selectedNode.type === 'computed'}
					<div>
						<label class="block text-xs font-medium text-gray-700 mb-1">
							Expression
							<span class="text-gray-500">(e.g., field1 + field2)</span>
						</label>
						<textarea
							bind:value={selectedNode.config.expression}
							class="w-full text-xs px-2 py-1 border border-gray-300 rounded h-20"
							placeholder="field1 + field2"
						></textarea>
					</div>
				{/if}

				{#if selectedNode.type === 'validation'}
					<div>
						<label class="block text-xs font-medium text-gray-700 mb-1">
							Validation Rule
							<span class="text-gray-500">(e.g., value > 0)</span>
						</label>
						<textarea
							bind:value={selectedNode.config.rule}
							class="w-full text-xs px-2 py-1 border border-gray-300 rounded h-20"
							placeholder="value > 0"
						></textarea>
					</div>
				{/if}
			</div>
		{:else}
			<p class="text-xs text-gray-500">Select a node to edit its properties</p>
		{/if}
	</div>
</div>
