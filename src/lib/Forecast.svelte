<script lang="ts">
	import days from "./static/days.json";
	import data from "./static/data.json";

	let output = "";
	let hoveredDay = 0;
	let hoveredEmployee = null;
</script>

<div class="grid overflow-scroll rounded shadow">
	<div class="grid grid-flow-col bg-gray-100">
		<div class="bg-gray-200 w-40 h-10 sticky left-0" />

		<div class="grid grid-flow-col">
			{#each days.list as day}
				<span
					data-day={day}
					class="h-10 w-16 border-l border-gray-200 text-sm flex items-center justify-center text-gray-500 font-mono"
					class:bg-blue-600={hoveredDay === day}
					class:border-blue-600={hoveredDay === day}
					class:font-medium={hoveredDay === day}
					class:text-blue-50={hoveredDay === day}>{day}</span
				>
			{/each}
		</div>
	</div>

	{#each data as forecast}
		<div class="grid grid-flow-col">
			<div
				class="flex items-center w-40 h-16 p-2 sticky left-0 z-10"
				class:bg-gray-100={hoveredEmployee !== forecast.employee.id}
				class:text-gray-800={hoveredEmployee !== forecast.employee.id}
				class:bg-blue-600={hoveredEmployee === forecast.employee.id}
				class:text-white={hoveredEmployee === forecast.employee.id}
			>
				<span>{forecast.employee.name}</span>
			</div>

			<div class="grid grid-flow-col">
				{#each days.list as day}
					{#if forecast.records.some((r) => r.series.includes(day))}
						{@const record = forecast.records.find((r) => r.series.includes(day))}

						{#if record}
							{@const [hue, saturation, lightness] = record.style.background_color_hsl}
							{@const lightness_hover = lightness < 50 ? lightness + 15 : lightness - 15}
							{@const is_first_day = record.series.indexOf(day) === 0}
							{@const is_last_day = record.series.indexOf(day) === record.series.length - 1}

							<button
								data-employee={forecast.employee.name}
								data-day={day}
								style="--cell-bg: {`hsl(${hue}deg,${saturation}%,${lightness}%)`}; --cell-bg-hover: {`hsl(${hue}deg,${saturation}%,${lightness_hover}%)`}"
								class="cell h-16 w-16 border-l border-b"
								class:relative={is_first_day}
								on:mouseenter={() => {
									hoveredDay = day;
									hoveredEmployee = forecast.employee.id;

									output = JSON.stringify(
										{
											title: record.title,
											series: record.series,
											employee: forecast.employee,
											day,
											type: is_first_day ? "first_day" : is_last_day ? "last_day" : "middle_day",
										},
										null,
										4
									);
								}}
								on:mouseleave={() => {
									hoveredDay = 0;
									hoveredEmployee = null;

									output = "";
								}}
							/>
						{/if}
					{:else}
						<button
							data-employee={forecast.employee.name}
							data-day={day}
							class={`h-16 w-16 hover:bg-gray-100 border-b border-gray-100`}
							on:mouseenter={() => {
								hoveredDay = day;
								hoveredEmployee = forecast.employee.id;

								output = JSON.stringify({ employee: forecast.employee, day }, null, 4);
							}}
							on:mouseleave={() => {
								hoveredDay = 0;
								hoveredEmployee = null;

								output = "";
							}}
						/>
					{/if}
				{/each}
			</div>
		</div>
	{/each}
</div>

<div class="flex my-10 text-sm">
	<pre>
		<code>
			{output}
		</code>
	</pre>
</div>

<style>
	.cell {
		background-color: var(--cell-bg);
		border-color: var(--cell-bg);
	}
	.cell:hover {
		background-color: var(--cell-bg-hover);
		border-color: var(--cell-bg-hover);
	}
</style>
