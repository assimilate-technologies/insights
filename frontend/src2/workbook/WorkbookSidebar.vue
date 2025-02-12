<script setup lang="ts">
import { Braces, LayoutPanelTop, ScrollText, Table2 } from 'lucide-vue-next'
import { computed, inject } from 'vue'
import { useRoute } from 'vue-router'
import ChartIcon from '../charts/components/ChartIcon.vue'
import WorkbookSidebarListSection from './WorkbookSidebarListSection.vue'
import { workbookKey } from './workbook'

const workbook = inject(workbookKey)!
const route = useRoute()

const activeQueryName = computed(() => {
	if (route.name === 'WorkbookQuery') {
		const index = parseInt(route.params.index as string)
		return workbook?.doc.queries[index].name
	}
})
</script>

<template>
	<div
		v-if="workbook"
		class="relative z-[1] flex h-full w-[17rem] flex-shrink-0 flex-col overflow-y-auto bg-white"
	>
		<WorkbookSidebarListSection
			v-bind="{
				title: 'Queries',
				emptyMessage: 'No queries',
				items: workbook.doc.queries,
				itemKey: 'name',
				isActive: (idx: number) => workbook.isActiveTab('query', idx),
				add: workbook.addQuery,
				remove: (query) => workbook.removeQuery(query.name),
				route: (idx: number) => `/workbook/${workbook.name}/query/${idx}`,
			}"
		>
			<template #item-icon="{ item }">
				<ScrollText
					v-if="item.is_native_query"
					class="h-4 w-4 text-gray-700"
					stroke-width="1.5"
				/>
				<Braces
					v-else-if="item.is_script_query"
					class="h-4 w-4 text-gray-700"
					stroke-width="1.5"
				/>
				<Table2 v-else class="h-4 w-4 text-gray-700" stroke-width="1.5" />
			</template>
		</WorkbookSidebarListSection>

		<WorkbookSidebarListSection
			v-bind="{
				title: 'Charts',
				emptyMessage: 'No charts',
				items: workbook.doc.charts,
				itemKey: 'name',
				isActive: (idx: number) => workbook.isActiveTab('chart', idx),
				add: () => workbook.addChart(activeQueryName),
				remove: (chart) => workbook.removeChart(chart.name),
				route: (idx: number) => `/workbook/${workbook.name}/chart/${idx}`,
			}"
		>
			<template #item-icon="{ item }">
				<ChartIcon :chart-type="item.chart_type" />
			</template>
		</WorkbookSidebarListSection>

		<WorkbookSidebarListSection
			v-bind="{
				title: 'Dashboards',
				emptyMessage: 'No dashboards',
				items: workbook.doc.dashboards,
				itemKey: 'name',
				isActive: (idx: number) => workbook.isActiveTab('dashboard', idx),
				add: workbook.addDashboard,
				remove: (dashboard) => workbook.removeDashboard(dashboard.name),
				route: (idx: number) => `/workbook/${workbook.name}/dashboard/${idx}`,
			}"
		>
			<template #item-icon>
				<LayoutPanelTop class="h-4 w-4 text-gray-700" stroke-width="1.5" />
			</template>
		</WorkbookSidebarListSection>
	</div>
</template>
