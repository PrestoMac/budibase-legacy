<script>
  // NOTE: this is not a block - it's just named as such to avoid confusing users,
  // because it functions similarly to one
  import { getContext } from "svelte"
  import { get } from "svelte/store"
  import { Grid } from "@budibase/frontend-core"

  // table is actually any datasource, but called table for legacy compatibility
  export let table
  export let allowAddRows = true
  export let allowEditRows = true
  export let allowDeleteRows = true
  export let stripeRows = false
  export let initialFilter = null
  export let initialSortColumn = null
  export let initialSortOrder = null
  export let fixedRowHeight = null
  export let columns = null
  export let onRowClick = null
  export let buttons = null

  const context = getContext("context")
  const component = getContext("component")
  const {
    styleable,
    API,
    builderStore,
    notificationStore,
    enrichButtonActions,
    ActionTypes,
    createContextStore,
  } = getContext("sdk")

  let grid

  $: columnWhitelist = columns?.map(col => col.name)
  $: schemaOverrides = getSchemaOverrides(columns)
  $: enrichedButtons = enrichButtons(buttons)

  const getSchemaOverrides = columns => {
    let overrides = {}
    columns?.forEach(column => {
      overrides[column.name] = {
        displayName: column.displayName || column.name,
        visible: true,
      }
    })
    return overrides
  }

  const enrichButtons = buttons => {
    if (!buttons?.length) {
      return null
    }
    return buttons.map(settings => ({
      size: "M",
      text: settings.text,
      type: settings.type,
      onClick: async row => {
        // Create a fake, ephemeral context to run the buttons actions with
        const id = get(component).id
        const gridContext = createContextStore(context)
        gridContext.actions.provideData(id, row)
        gridContext.actions.provideAction(
          id,
          ActionTypes.RefreshDatasource,
          () => grid?.getContext()?.rows.actions.refreshData()
        )
        const fn = enrichButtonActions(settings.onClick, get(gridContext))
        return await fn?.({ row })
      },
    }))
  }
</script>

<div
  use:styleable={$component.styles}
  class:in-builder={$builderStore.inBuilder}
>
  <Grid
    bind:this={grid}
    datasource={table}
    {API}
    {stripeRows}
    {initialFilter}
    {initialSortColumn}
    {initialSortOrder}
    {fixedRowHeight}
    {columnWhitelist}
    {schemaOverrides}
    canAddRows={allowAddRows}
    canEditRows={allowEditRows}
    canDeleteRows={allowDeleteRows}
    canEditColumns={false}
    canExpandRows={false}
    canSaveSchema={false}
    showControls={false}
    notifySuccess={notificationStore.actions.success}
    notifyError={notificationStore.actions.error}
    buttons={enrichedButtons}
    on:rowclick={e => onRowClick?.({ row: e.detail })}
  />
</div>

<style>
  div {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    border: 1px solid var(--spectrum-global-color-gray-300);
    border-radius: 4px;
    overflow: hidden;
    min-height: 410px;
  }
  div.in-builder :global(*) {
    pointer-events: none;
  }
</style>
