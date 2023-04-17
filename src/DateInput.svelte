<script lang="ts">
  import { fly } from 'svelte/transition'
  import { cubicInOut } from 'svelte/easing'
  import { toText } from './date-utils'
  import type { Locale } from './locale'
  import { parse, createFormat } from './parse'
  import type { FormatToken } from './parse'
  import DateTimePicker from './DatePicker.svelte'
  import { writable } from 'svelte/store'
  import { createEventDispatcher } from 'svelte'
  import { parse as dateParse, isValid } from 'date-fns';
  import { enUS } from 'date-fns/locale'
  import zonedTimeToUtc from 'date-fns-tz/zonedTimeToUtc'

  const dispatch = createEventDispatcher<{ select: Date }>()

  /** Default date to display in picker before value is assigned */
  export let defaultDate
  export let ignoreTimezones = false

  // inner date value store for preventing value updates (and also
  // text updates as a result) when date is unchanged
  const innerStore = writable(null as Date | null)
  const store = (() => {
    return {
      subscribe: innerStore.subscribe,
      set: (d: Date | null) => {
        if (d === null) {
          innerStore.set(null)
          value = d
        } else if (d.getTime() !== $innerStore?.getTime()) {
          if (ignoreTimezones) {
            d = zonedTimeToUtc(d, Intl.DateTimeFormat().resolvedOptions().timeZone)
          }
          innerStore.set(d)
          value = d
        }
      },
    }
  })()

  /** Date value */
  export let value: Date | null = defaultDate
  $: {
    store.set(value)
    dispatch('select', value)
  }

  /** The earliest value the user can select */
  export let min = new Date((defaultDate?.getFullYear() ?? new Date().getFullYear()) - 20, 0, 1)
  /** The latest value the user can select */
  export let max = new Date((defaultDate?.getFullYear() ?? new Date().getFullYear()), 11, 31, 23, 59, 59, 999)
  /** Placeholder text to show when input field is empty */
  export let placeholder = '2020-12-31 23:00:00'
  /** Whether the text is valid */
  export let valid = true
  /** Disable the input **/
  export let disabled = false
  /** Disable the calender **/
  export let showCalendar = true

  /** Format string */
  export let format = 'yyyy-MM-dd HH:mm:ss'
  let formatTokens = createFormat(format)
  $: formatTokens = createFormat(format)

  /** Locale object for internationalization */
  export let locale: Locale = {}
  export let dfLocale = enUS

  function valueUpdate(value: Date | null, formatTokens: FormatToken[]) {
    text = toText(value, formatTokens)
  }
  $: valueUpdate($store, formatTokens)

  export let text = toText($store, formatTokens)
  let textHistory = [text, text]
  $: textHistory = [textHistory[1], text]

  $: {
    if (typeof min === "string" && (min as string).length > 0) {
      let parsedDate = dateParse(min, format, new Date(), { locale: dfLocale })
      if (!isValid(parsedDate)) {
        parsedDate = dateParse(min, "yyyy-MM-dd HH:mm:ss", new Date())
      }
      if (isValid(parsedDate)) {
        min = parsedDate
      } else {
        min = new Date((defaultDate?.getFullYear() ?? new Date().getFullYear()) - 20, 0, 1)
      }
    } else if (typeof min === "string") {
      min = new Date((defaultDate?.getFullYear() ?? new Date().getFullYear()) - 20, 0, 1)
    }
    if (typeof max === "string" && (max as string).length > 0) {
      let parsedDate = dateParse(max, format, new Date(), { locale: dfLocale })
      if (!isValid(parsedDate)) {
        parsedDate = dateParse(max, "yyyy-MM-dd HH:mm:ss", new Date())
      }
      if (isValid(parsedDate)) {
        max = parsedDate
      } else {
        max = new Date((defaultDate?.getFullYear() ?? new Date().getFullYear()), 11, 31, 23, 59, 59, 999)
      }
    } else if (typeof max === "string") {
      max = new Date((defaultDate?.getFullYear() ?? new Date().getFullYear()), 11, 31, 23, 59, 59, 999)
    }
  }

  function textUpdate(text: string, formatTokens: FormatToken[]) {
    if (text.length) {
      const result = parse(text, formatTokens, $store)
      if (result.date !== null && text.length <= format.length) {
        valid = true
        store.set(result.date)
      } else {
        valid = false
      }
    } else {
      valid = true // <-- empty string is always valid
      // value resets to null if you clear the field
      if (value) {
        value = null
        store.set(null)
      }
    }
  }
  $: textUpdate(text, formatTokens)

  function input(e: unknown) {
    if (
      e instanceof InputEvent &&
      e.inputType === 'insertText' &&
      typeof e.data === 'string' &&
      text === textHistory[0] + e.data
    ) {
      // check for missing punctuation, and add if there is any
      let result = parse(textHistory[0], formatTokens, $store)
      if (result.missingPunctuation !== '' && !result.missingPunctuation.startsWith(e.data)) {
        text = textHistory[0] + result.missingPunctuation + e.data
      }
    }
  }

  /** Whether the date popup is visible */
  export let visible = false
  /** Close the date popup when a date is selected */
  export let closeOnSelection = false
  /** Wait with updating the date until a date is selected */
  export let browseWithoutSelecting = false

  // handle on:focusout for parent element. If the parent element loses
  // focus (e.g input element), visible is set to false
  function onFocusOut(e: FocusEvent) {
    if (
      e?.currentTarget instanceof HTMLElement &&
      e.relatedTarget &&
      e.relatedTarget instanceof Node &&
      e.currentTarget.contains(e.relatedTarget)
    ) {
      return
    } else {
      visible = false
    }
  }
  function keydown(e: KeyboardEvent) {
    if (e.key === 'Escape' && visible) {
      visible = false
      e.preventDefault()
      // When the date picker is open, we prevent 'Escape' from propagating,
      // so for example a parent modal won't be closed
      e.stopPropagation()
    } else if (e.key === 'Enter') {
      visible = !visible
      e.preventDefault()
    }
  }

  function onSelect() {
    if (closeOnSelection) {
      visible = false
    }
  }
</script>

<div class="pickerContainer" on:focusout={onFocusOut} >
  <div class="spectrum-Textfield spectrum-Textfield-input spectrum-InputGroup-input" on:keydown={keydown}>
    <input
      class:invalid={!valid}
      type="text"
      bind:value={text}
      {placeholder}
      {disabled}
      on:focus={() => (visible = true)}
      on:mousedown={() => (visible = true)}
      on:input={input}
    />
  </div>
  {#if visible && !disabled}
    <div class="picker" class:visible transition:fly={{ duration: 80, easing: cubicInOut, y: -5 }}>
      <DateTimePicker
        on:select={onSelect}
        bind:value={$store}
        {min}
        {max}
        {locale}
        {browseWithoutSelecting}
        {showCalendar}
        parsedPlaceholder={dateParse(placeholder, format, new Date(), { locale: dfLocale })}
      />
    </div>
  {/if}
</div>

<style lang="sass">
  .pickerContainer
    height: 32px
    overflow: visible
  .spectrum-Textfield
    width: 100%
  input
    color: var(--date-picker-foreground, #000000)
    background: var(--date-picker-background, #ffffff)
    min-width: 0px
    box-sizing: border-box
    border: none
    outline: none
    transition: all 80ms cubic-bezier(0.4, 0.0, 0.2, 1)
    &:focus
      border-color: var(--date-picker-highlight-border, #0269f7)
      box-shadow: 0px 0px 0px 2px var(--date-picker-highlight-shadow, rgba(#0269f7, 0.4))
    &:disabled
      opacity: 0.5
  .invalid
    border: 1px solid rgba(#f92f72, 0.5)
    background-color: rgba(#f92f72, 0.1)
    &:focus
      border-color: #f92f72
      box-shadow: 0px 0px 0px 2px rgba(#f92f72, 0.5)
  .picker
    display: none
    position: sticky
    z-index: 999
    &.visible
      display: block
</style>
