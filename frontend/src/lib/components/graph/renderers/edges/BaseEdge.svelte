<script lang="ts">
	import InsertModuleButton from '$lib/components/flows/map/InsertModuleButton.svelte'
	import type { FlowModule } from '$lib/gen'
	import { getBezierPath, BaseEdge, type Position, EdgeLabelRenderer } from '@xyflow/svelte'
	import { ClipboardCopy } from 'lucide-svelte'
	import { getContext } from 'svelte'
	import type { Writable } from 'svelte/store'
	import type { GraphEventHandlers } from '../../graphBuilder'
	import { getStraightLinePath } from '../utils'
	import { twMerge } from 'tailwind-merge'

	export let sourceX: number
	export let sourceY: number
	export let sourcePosition: Position
	export let targetX: number
	export let targetY: number
	export let targetPosition: Position
	export let markerEnd: string | undefined = undefined

	export let data: {
		insertable: boolean
		modules: FlowModule[]
		moving: string | undefined
		eventHandlers: GraphEventHandlers
		index: number
		enableTrigger: boolean
		disableAi: boolean
		disableMoveIds: string[]
	}

	$: [edgePath] = getBezierPath({
		sourceX,
		sourceY: targetY - sourceY > 100 ? targetY - 100 : sourceY,
		sourcePosition,
		targetX,
		targetY,
		targetPosition,
		curvature: 0.25
	})

	$: completeEdge =
		targetY - sourceY > 100
			? `${edgePath} ${getStraightLinePath({ sourceX, sourceY, targetY })}`
			: edgePath

	const { useDataflow } = getContext<{
		useDataflow: Writable<boolean | undefined>
	}>('FlowGraphContext')
</script>

<EdgeLabelRenderer>
	{#if data?.insertable && !$useDataflow && !data?.moving}
		<div
			class={twMerge('edgeButtonContainer nodrag nopan top-0')}
			style:transform="translate(-50%, 50%) translate({sourceX}px,{sourceY + 2}px)"
		>
			<InsertModuleButton
				disableAi={data.disableAi}
				index={data.index ?? 0}
				allowTrigger={data.enableTrigger}
				modules={data?.modules ?? []}
				on:new={(e) => {
					data?.eventHandlers.insert({
						modules: data.modules,
						index: data.index,
						kind: e.detail.kind,
						inlineScript: e.detail.inlineScript
					})
				}}
				on:pickScript={(e) => {
					// console.log('pickScript', e)
					data?.eventHandlers.insert({
						modules: data.modules,
						index: data.index,
						script: e.detail,
						kind: e.detail.kind
					})
				}}
				on:pickFlow={(e) => {
					// console.log('pickFlow', e)
					data?.eventHandlers.insert({
						modules: data.modules,
						index: data.index,
						flow: e.detail
					})
				}}
			/>
		</div>
	{/if}

	{#if data?.moving}
		<div
			class="edgeButtonContainer nodrag nopan"
			style:transform="translate(-50%, 50%) translate({sourceX}px,{sourceY}px)"
		>
			{#if data.moving && !data.disableMoveIds?.includes(data.moving)}
				<button
					title="Paste module"
					on:click={() => {
						data.eventHandlers.insert({
							modules: data.modules,
							index: data.index
						})
					}}
					type="button"
					class={twMerge(
						'w-6 h-6 flex items-center justify-center',
						'border border-gray-300 dark:border-gray-500',
						'text-primary text-sm',
						'bg-surface focus:outline-none hover:bg-surface-hover focus:ring-4 focus:ring-surface-selected rounded-full '
					)}
				>
					<ClipboardCopy size={14} />
				</button>
			{/if}
		</div>
	{/if}
</EdgeLabelRenderer>

<BaseEdge path={completeEdge} {markerEnd} class={$useDataflow ? 'hidden' : ''} />

<style>
	.edgeButtonContainer {
		position: absolute;
		font-size: 12pt;
		/* everything inside EdgeLabelRenderer has no pointer events by default */
		/* if you have an interactive element, set pointer-events: all */
		pointer-events: all;
	}
</style>
