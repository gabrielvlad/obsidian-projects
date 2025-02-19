<script lang="ts">
  import produce from "immer";
  import {
    Button,
    IconButton,
    Icon,
    Select,
    TextInput,
    NumberInput,
    Checkbox,
  } from "obsidian-svelte";
  import HorizontalGroup from "src/ui/components/HorizontalGroup/HorizontalGroup.svelte";
  import type { DataField } from "src/lib/dataframe/dataframe";
  import {
    filterOperatorTypes,
    isNumberFilterOperator,
    isStringFilterOperator,
    type FilterDefinition,
    type FilterOperator,
  } from "src/settings/settings";
  import { fieldsToSelectOptions } from "../helpers";
  import {
    addCondition,
    getFieldByName,
    getOperatorsByField,
    removeCondition,
    setField,
    setOperator,
    setValue,
  } from "./helpers";
  import { i18n } from "src/lib/stores/i18n";

  export let filter: FilterDefinition;
  export let fields: DataField[];
  export let onFilterChange: (filter: FilterDefinition) => void;

  $: fieldOptions = fieldsToSelectOptions(fields);

  const handleFieldChange =
    (i: number) =>
    ({ detail }: CustomEvent<string>) => {
      filter = setField(filter, i, detail);
      onFilterChange(filter);
    };

  const handleOperatorChange =
    (i: number) =>
    ({ detail }: CustomEvent<string>) => {
      filter = setOperator(filter, i, detail as FilterOperator);
      onFilterChange(filter);
    };

  const handleValueChange = (i: number) => (event: FocusEvent) => {
    if (event.currentTarget instanceof HTMLInputElement) {
      filter = setValue(filter, i, event.currentTarget.value);
      onFilterChange(filter);
    }
  };

  const handleStatusChange =
    (i: number) =>
    ({ detail }: CustomEvent<boolean>) => {
      filter = produce(filter, (draft) => {
        draft.conditions = draft.conditions.map((cond, idx) =>
          idx !== i ? cond : { ...cond, enabled: detail as boolean }
        );
      });
      onFilterChange(filter);
    };

  const handleConditionRemove = (i: number) => (event: MouseEvent) => {
    event.stopPropagation();
    filter = removeCondition(filter, i);
    onFilterChange(filter);
  };

  function handleConditionAdd() {
    filter = addCondition(filter, fields);
    onFilterChange(filter);
  }
</script>

<div style="display: flex; flex-direction: column; gap: 8px;">
  {#each filter.conditions as condition, i}
    {@const field = getFieldByName(fields, condition.field)}
    <HorizontalGroup>
      <div class="setting-item-name" style="width: 5ch">
        {i === 0
          ? $i18n.t("components.filter.where")
          : $i18n.t("components.filter.and")}
      </div>
      <Select
        value={condition.field}
        options={fieldOptions}
        on:change={handleFieldChange(i)}
      />
      <Select
        value={condition.operator}
        on:change={handleOperatorChange(i)}
        options={field ? getOperatorsByField(field) : []}
      />
      {#if filterOperatorTypes[condition.operator] === "binary"}
        {#if isStringFilterOperator(condition.operator)}
          <TextInput
            value={condition.value ?? ""}
            on:blur={handleValueChange(i)}
          />
        {:else if isNumberFilterOperator(condition.operator)}
          <NumberInput
            value={parseFloat(condition.value ?? "")}
            on:blur={handleValueChange(i)}
          />
        {/if}
      {/if}
      <Checkbox
        checked={condition?.enabled ?? true}
        on:check={handleStatusChange(i)}
      />
      <IconButton icon="trash" onClick={handleConditionRemove(i)} />
    </HorizontalGroup>
  {/each}
  <HorizontalGroup>
    <Button variant="plain" on:click={handleConditionAdd}
      ><Icon name="plus" />{$i18n.t("components.filter.add")}</Button
    >
  </HorizontalGroup>
</div>
