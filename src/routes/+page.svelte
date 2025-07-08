<script lang="ts">
	// --- Types ---
	type Project = {
		id: number;
		name: string;
		priority: number;
	};
	type Task = {
		name: string;
		urgency: number;
		projectId: number;
		roadblocking: boolean;
	};

	// --- State ---
	let projects: Project[] = $state([
		{ id: 1, name: 'Restore', priority: 5 },
		{ id: 2, name: 'Landing Page', priority: 2 },
		{ id: 3, name: 'Darkstar', priority: 4 }
	]);

	let newProjectName = $state('');
	let newProjectPriority = $state(3);

	let tasks: Task[] = $state([
		{ name: 'User login', urgency: 5, projectId: 1, roadblocking: true },
		{ name: 'Hero section', urgency: 3, projectId: 2, roadblocking: false }
	]);

	// --- Logic ---
function compositePriority({ urgency, projectId, roadblocking }: Task): string {
	const project = projects.find(p => p.id === projectId) ?? { priority: 1 };
	const priority = project.priority;

	// High urgency & high project priority
	if (urgency >= 4 && priority >= 4) return 'A';

	// Roadblocking AND at least moderate urgency or priority (3+)
	if (roadblocking && (urgency >= 3 || priority >= 3)) return 'A';

	// Roadblocking but both urgency & priority are low (≤2) → B
	if (roadblocking && urgency <= 2 && priority <= 2) return 'B';

	// High urgency or high project priority (but not both) → B
	if (urgency >= 4 || priority >= 4) return 'B';

	// Moderate urgency or priority (3) → C
	if (urgency === 3 || priority === 3) return 'C';

	// Low everything
	return 'D';
}


	function addTask(): void {
		const firstProject = projects[0]?.id ?? 1;
		tasks.push({ name: '', urgency: 1, projectId: firstProject, roadblocking: false });
	}

	function removeTask(i: number): void {
		tasks.splice(i, 1);
	}

	function addProject(): void {
		if (!newProjectName.trim()) return;
		const nextId = Math.max(0, ...projects.map(p => p.id)) + 1;
		projects.push({ id: nextId, name: newProjectName.trim(), priority: Number(newProjectPriority) });
		newProjectName = '';
		newProjectPriority = 3;
	}

	function removeProject(id: number): void {
		projects = projects.filter(p => p.id !== id);
		tasks = tasks.filter(t => t.projectId !== id);
	}
</script>

<!-- PROJECTS MANAGER -->
<div class="bg-white shadow-xl rounded-2xl p-6 mb-8 max-w-xl mx-auto mt-10">
	<h2 class="font-bold text-2xl mb-4 text-gray-900 flex items-center gap-2">
		<svg width="28" height="28" fill="none" viewBox="0 0 24 24"><rect width="24" height="24" rx="7" fill="#10b981"/><path d="M8 12l2 2l4-4" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
		Projects
	</h2>
	<ul class="divide-y divide-gray-200 mb-4">
		{#each projects as project (project.id)}
			<li class="flex items-center justify-between py-2">
				<div>
					<span class="font-semibold text-gray-800">{project.name}</span>
					<span class="ml-2 px-2 py-0.5 bg-emerald-100 text-emerald-700 text-xs rounded-lg">Priority: {project.priority}</span>
				</div>
				<button type="button" onclick={() => removeProject(project.id)} class="text-gray-400 hover:text-red-500 transition px-2 py-1">
					<svg width="18" height="18" viewBox="0 0 20 20" fill="none"><path d="M6 6l8 8M6 14L14 6" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>
				</button>
			</li>
		{/each}
	</ul>
	<div class="flex gap-2 mt-2">
		<input class="flex-1 border border-gray-300 rounded-lg px-3 py-2 text-sm" placeholder="New project name" bind:value={newProjectName} />
		<select class="border border-gray-300 rounded-lg px-2 py-2 text-sm" bind:value={newProjectPriority}>
			{#each [1,2,3,4,5] as prio}
				<option value={prio}>Priority {prio}</option>
			{/each}
		</select>
		<button type="button" class="bg-emerald-600 text-white px-4 py-2 rounded-lg font-bold hover:bg-emerald-700 transition" onclick={addProject}>
			Add
		</button>
	</div>
</div>

<!-- TASKS TABLE -->
<div class="bg-white shadow-2xl rounded-2xl p-8 max-w-5xl mx-auto">
	<h2 class="font-black text-3xl mb-6 text-gray-900 flex items-center gap-2">
		<svg width="30" height="30" fill="none" viewBox="0 0 24 24"><rect width="24" height="24" rx="7" fill="#6366f1"/><path d="M7 13l3 3 7-7" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
		Task Priority Chart
	</h2>
	<table class="min-w-full table-fixed rounded-lg overflow-hidden">
		<thead class="bg-gray-50 text-gray-600 uppercase text-xs tracking-wider">
			<tr>
				<th class="p-3 w-2/5">Task</th>
				<th class="p-3 w-1/5">Project</th>
				<th class="p-3 w-1/12">Urgency</th>
				<th class="p-3 w-1/12">Roadblock</th>
				<th class="p-3 w-1/12">Proj. Priority</th>
				<th class="p-3 w-1/12">Priority</th>
				<th></th>
			</tr>
		</thead>
		<tbody>
			{#each tasks as task, i}
				<tr class="hover:bg-gray-50 transition">
					<td class="p-2">
						<input type="text" bind:value={task.name} class="w-full border rounded-md px-3 py-1 text-sm focus:outline-emerald-600" placeholder="Task name..." />
					</td>
					<td class="p-2">
						<select bind:value={task.projectId} class="w-full border rounded-md px-2 py-1 text-sm">
							{#each projects as p}
								<option value={p.id}>{p.name}</option>
							{/each}
						</select>
					</td>
					<td class="p-2 text-center">
						<input type="number" min="1" max="5" bind:value={task.urgency} class="w-16 border rounded px-1 text-center" />
					</td>
					<td class="p-2 text-center">
						<input type="checkbox" bind:checked={task.roadblocking} class="accent-emerald-600" />
					</td>
					<td class="p-2 text-center">
						<span class="inline-block px-2 py-0.5 rounded-lg bg-indigo-100 text-indigo-700 text-xs font-bold">
							{projects.find(p => p.id === task.projectId)?.priority ?? 1}
						</span>
					</td>
					<td class="p-2 text-center">
						<span class={`inline-block px-2 py-0.5 rounded-lg text-xs font-bold
							${compositePriority(task) === 'A' ? 'bg-red-100 text-red-700' :
							compositePriority(task) === 'B' ? 'bg-orange-100 text-orange-700' :
							compositePriority(task) === 'C' ? 'bg-amber-100 text-amber-700' : 'bg-gray-200 text-gray-700'}`}>
							{compositePriority(task)}
						</span>
					</td>
					<td>
						<button type="button" onclick={() => removeTask(i)} class="text-gray-400 hover:text-red-500 transition px-2 py-1">
							<svg width="18" height="18" viewBox="0 0 20 20" fill="none"><path d="M6 6l8 8M6 14L14 6" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>
						</button>
					</td>
				</tr>
			{/each}
		</tbody>
	</table>
	<button type="button" onclick={addTask} class="mt-6 px-5 py-3 bg-indigo-600 text-white font-bold rounded-xl shadow-lg hover:bg-indigo-700 transition">
		+ Add Task
	</button>
</div>
