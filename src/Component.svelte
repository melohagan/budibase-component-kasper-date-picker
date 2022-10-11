<script>
  import { getContext, onDestroy } from "svelte"
  import DateInput from "./DateInput.svelte";
  import { parse, isValid } from 'date-fns';
  import { enUS, enGB, zhCN, ru, fr, es, de, ptBR, it, ja, bn, da } from 'date-fns/locale';

  export let field
  export let label
  export let defaultValue = null
  export let disabled = false
  export let validation
  export let date
  export let showCalendar = true
  export let min
  export let max
  export let locale = "enUS"
  export let format = 'yyyy-MM-dd HH:mm:ss'
  export let placeholder

  const formContext = getContext("form")
  const formStepContext = getContext("form-step");

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  const formApi = formContext?.formApi;
  $: formStep = formStepContext ? $formStepContext || 1 : 1
  $: formField = formApi?.registerField(field, "datetime", defaultValue, disabled, validation, formStep)

  let fieldApi;
  let fieldState; 

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  const fieldGroupContext = getContext("field-group")
  const labelPos = fieldGroupContext?.labelPosition || "above"
  $: labelClass = labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  $: {
    if (typeof defaultValue === "string") {
      const parsedDate = parse(defaultValue, format, new Date());
      if (isValid(parsedDate)) {
        defaultValue = parsedDate
      } else {
        defaultValue = null
      }
    }
  }

  let dateLocale = enUS
  $: {
    if (locale === "enUS") {
      dateLocale = enUS
    }
    else if (locale === "enGB") {
      dateLocale = enGB
    }
    else if (locale === "bn") {
      dateLocale = bn
    }
    else if (locale === "da") {
      dateLocale = da
    }
    else if (locale === "de") {
      dateLocale = de
    }
    else if (locale === "es") {
      dateLocale = es
    }
    else if (locale === "fr") {
      dateLocale = fr
    }
    else if (locale === "it") {
      dateLocale = it
    }
    else if (locale === "ja") {
      dateLocale = ja
    }
    else if (locale === "ptBR") {
      dateLocale = ptBR
    }
    else if (locale === "ru") {
      dateLocale = ru
    }
    else if (locale === "zhCN") {
      dateLocale = zhCN
    }
  }
</script>

<div use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form.</div>
  {:else}
  <div class="spectrum-Form-item">
    <label
        class:hidden={!label}
        for={fieldState?.fieldId}
        class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
      >
        {label || " "}
    </label>
  </div>
    <DateInput 
      bind:value={date} 
      on:select={() => fieldApi?.setValue(date)} 
      {showCalendar}
      {min}
      {max}
      {placeholder}
      defaultDate={defaultValue}
      {disabled}
      locale={dateLocale}
    />
    {#if fieldState?.error}
      <div class="error">{fieldState.error}</div>
    {/if}
  {/if}
</div>

<style>
  label {
    white-space: nowrap;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
  .error {
    color: var(--spectrum-semantic-negative-color-default, 
               var(--spectrum-global-color-red-500));
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
</style>