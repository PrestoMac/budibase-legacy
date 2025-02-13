<script>
  import Panel from "components/design/Panel.svelte"
  import { store, selectedComponent, selectedScreen } from "builderStore"
  import { getComponentText } from "builderStore/componentUtils"
  import ComponentSettingsSection from "./ComponentSettingsSection.svelte"
  import DesignSection from "./DesignSection.svelte"
  import CustomStylesSection from "./CustomStylesSection.svelte"
  import ConditionalUISection from "./ConditionalUISection.svelte"
  import { notifications, ActionButton } from "@budibase/bbui"

  import {
    getBindableProperties,
    getComponentBindableProperties,
  } from "builderStore/dataBinding"
  import { capitalise } from "helpers"

  const onUpdateName = async value => {
    try {
      await store.actions.components.updateSetting("_instanceName", value)
    } catch (error) {
      notifications.error("Error updating component name")
    }
  }

  $: componentInstance = $selectedComponent
  $: componentDefinition = store.actions.components.getDefinition(
    $selectedComponent?._component
  )
  $: bindings = getBindableProperties(
    $selectedScreen,
    $store.selectedComponentId
  )

  $: componentBindings = getComponentBindableProperties(
    $selectedScreen,
    $store.selectedComponentId
  )
  $: isScreen = $selectedComponent?._id === $selectedScreen?.props._id
  $: title = isScreen ? "Screen" : $selectedComponent?._instanceName

  let section = "settings"
  const tabs = ["settings", "styles", "conditions"]

  $: id = $selectedComponent?._id
  $: id, (section = tabs[0])
</script>

{#if $selectedComponent}
  {#key $selectedComponent._id}
    <Panel {title} icon={componentDefinition?.icon} borderLeft wide>
      <span class="panel-title-content" slot="panel-title-content">
        <input
          class="input"
          value={title}
          {title}
          placeholder={getComponentText(componentInstance)}
          on:keypress={e => {
            if (e.key.toLowerCase() === "enter") {
              e.target.blur()
            }
          }}
          on:change={e => {
            onUpdateName(e.target.value)
          }}
        />
      </span>
      <span slot="panel-header-content">
        <div class="settings-tabs">
          {#each tabs as tab}
            <ActionButton
              size="M"
              quiet
              selected={section === tab}
              on:click={() => {
                section = tab
              }}
            >
              {capitalise(tab)}
            </ActionButton>
          {/each}
        </div>
      </span>
      {#if section == "settings"}
        <ComponentSettingsSection
          {componentInstance}
          {componentDefinition}
          {bindings}
          {componentBindings}
          {isScreen}
        />
      {/if}
      {#if section == "styles"}
        <DesignSection
          {componentInstance}
          {componentBindings}
          {componentDefinition}
          {bindings}
        />
        <CustomStylesSection
          {componentInstance}
          {componentDefinition}
          {bindings}
        />
      {/if}
      {#if section == "conditions"}
        <ConditionalUISection
          {componentInstance}
          {componentDefinition}
          {bindings}
        />
      {/if}
    </Panel>
  {/key}
{/if}

<style>
  .settings-tabs {
    display: flex;
    gap: var(--spacing-s);
    padding: 0 var(--spacing-l);
    padding-bottom: var(--spacing-l);
  }
  .input {
    color: inherit;
    font-family: inherit;
    font-size: inherit;
    background-color: transparent;
    border: none;
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .panel-title-content {
    display: contents;
  }
  .input:focus {
    outline: none;
  }
  input::placeholder {
    color: var(--spectrum-global-color-gray-600);
  }
</style>
