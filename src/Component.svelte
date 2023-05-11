<script>
  import { getContext, onDestroy } from "svelte"
  import DateInput from "./DateInput.svelte";
  import { localeFromDateFnsLocale } from './locale'
  import { parse, isValid } from 'date-fns';
  import { enUS, enGB, zhCN, ru, fr, es, de, ptBR, it, ja, bn, da } from 'date-fns/locale';

  export let field
  export let label
  export let defaultValue = null
  export let disabled = false
  export let validation
  export let showCalendar = true
  export let min
  export let max
  export let locale = "enUS"
  export let format = "MM/dd/yyyy"
  export let customFormat = false
  export let closeOnSelection = false
  export let ignoreTimezones = false
  export let placeholder
  export let onChange

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
    if (!customFormat && defaultFormat) {
      format = defaultFormat
    }
  }

  let kasperLocale = enUS
  let dfLocale = enUS
  let defaultFormat = "MM/dd/yyyy"
  $: {
    if (locale === "enUS") {
      kasperLocale = localeFromDateFnsLocale(enUS)
      dfLocale = enUS
      defaultFormat = "MM/dd/yyyy"
    }
    else if (locale === "enGB") {
      kasperLocale = localeFromDateFnsLocale(enGB)
      dfLocale = enGB
      defaultFormat = "dd/MM/yyyy"
    }
    else if (locale === "bn") {
      kasperLocale = localeFromDateFnsLocale(bn)
      dfLocale = bn
      defaultFormat = "dd-MM-yyyy"
    }
    else if (locale === "da") {
      kasperLocale = localeFromDateFnsLocale(da)
      dfLocale = da
      defaultFormat = "dd-MM-yyyy"
    }
    else if (locale === "de") {
      kasperLocale = localeFromDateFnsLocale(de)
      dfLocale = de
      defaultFormat = "dd.MM.yyyy"
    }
    else if (locale === "es") {
      kasperLocale = localeFromDateFnsLocale(es)
      dfLocale = es
      defaultFormat = "dd/MM/yyyy"
    }
    else if (locale === "fr") {
      kasperLocale = localeFromDateFnsLocale(fr)
      dfLocale = fr
      defaultFormat = "dd/MM/yyyy"
    }
    else if (locale === "it") {
      kasperLocale = localeFromDateFnsLocale(it)
      dfLocale = it
      defaultFormat = "dd/MM/yyyy"
    }
    else if (locale === "ja") {
      kasperLocale = localeFromDateFnsLocale(ja)
      dfLocale = ja
      defaultFormat = "yyyy/MM/dd"
    }
    else if (locale === "ptBR") {
      kasperLocale = localeFromDateFnsLocale(ptBR)
      dfLocale = ptBR
      defaultFormat = "dd/MM/yyyy"
    }
    else if (locale === "ru") {
      kasperLocale = localeFromDateFnsLocale(ru)
      dfLocale = ru
      defaultFormat = "dd.MM.yyyy"
    }
    else if (locale === "zhCN") {
      kasperLocale = localeFromDateFnsLocale(zhCN)
      dfLocale = zhCN
      defaultFormat = "yyyy/MM/dd"
    }
    if (!customFormat) {
      format = defaultFormat
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
      value={fieldState.value} 
      on:select={e => {
        const changed = fieldApi?.setValue(e.detail)
        if (onChange && changed) {
          onChange({ value: e.detail })
        }
      }}
      on:change={onChange}
      {showCalendar}
      {min}
      {max}
      {placeholder}
      defaultDate={defaultValue}
      disabled={fieldState?.disabled ?? disabled}
      locale={kasperLocale}
      {dfLocale}
      {format}
      {closeOnSelection}
      {ignoreTimezones}
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
