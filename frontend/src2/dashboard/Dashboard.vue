<script setup lang="ts">
import { Breadcrumbs } from 'frappe-ui'
import { RefreshCcw } from 'lucide-vue-next'
import { provide, ref } from 'vue'
import { useRouter } from 'vue-router'
import { downloadImage, waitUntil } from '../helpers'
import useWorkbook from '../workbook/workbook'
import useDashboard from './dashboard'
import DashboardItem from './DashboardItem.vue'
import useDashboardStore from './dashboards'
import VueGridLayout from './VueGridLayout.vue'

const props = defineProps<{ name: string }>()

const store = useDashboardStore()
const workbookName = await store.fetchWorkbookName(props.name)

const workbook = useWorkbook(workbookName)
await waitUntil(() => workbook.doc.dashboards.length > 0)

const dashboard = useDashboard(
	workbook.doc.dashboards.find((dashboard) => dashboard.name === props.name)!
)
provide('dashboard', dashboard)
dashboard.refresh()

const router = useRouter()
function openWorkbook() {
	router.push(`/workbook/${workbook.doc.name}`)
}

const dashboardContainer = ref<HTMLElement | null>(null)
async function downloadDashboardImage() {
	if (!dashboardContainer.value) return
	await downloadImage(dashboardContainer.value, `${dashboard.doc.title}.png`)
}
</script>

<template>
	<header class="flex h-12 items-center justify-between border-b py-2.5 pl-5 pr-2">
		<Breadcrumbs
			:items="[
				{ label: 'Dashboards', route: '/dashboards' },
				{ label: dashboard.doc.title, route: `/dashboards/${dashboard.doc.name}` },
			]"
		/>
		<div class="flex items-center gap-2">
			<Button variant="outline" @click="() => dashboard.refresh()" label="Refresh">
				<template #prefix>
					<RefreshCcw class="h-4 w-4 text-gray-700" stroke-width="1.5" />
				</template>
			</Button>
			<Dropdown
				placement="left"
				:button="{ icon: 'more-vertical', variant: 'outline' }"
				:options="[
					{
						label: 'Export as PNG',
						variant: 'outline',
						icon: 'download',
						onClick: downloadDashboardImage,
					},
					{
						label: 'Open Workbook',
						variant: 'outline',
						icon: 'external-link',
						onClick: openWorkbook,
					},
				]"
			/>
		</div>
	</header>

	<div class="relative flex h-full w-full overflow-hidden">
		<div ref="dashboardContainer" class="flex-1 overflow-y-auto p-4">
			<VueGridLayout
				v-if="dashboard.doc.items.length > 0"
				class="h-fit w-full"
				:cols="20"
				:disabled="true"
				:modelValue="dashboard.doc.items.map((item) => item.layout)"
			>
				<template #item="{ index }">
					<DashboardItem :index="index" :item="dashboard.doc.items[index]" />
				</template>
			</VueGridLayout>
		</div>
	</div>
</template>
