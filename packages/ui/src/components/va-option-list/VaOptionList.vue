<template>
  <VaMessageListWrapper
    :error="computedError"
    :error-messages="computedErrorMessages"
    :error-count="$props.errorCount"
  >
    <ul
      class="va-option-list__list"
      :id="String($props.id)"
    >
      <li
        v-for="(option, index) in $props.options"
        :key="getKey(option)"
      >
        <slot
          :props="{
            option,
            isDisabled,
            name: $props.name,
            color: $props.color,
            leftLabel: $props.leftLabel,
            getText,
            selectedValue,
            index
          }"
        >
          <va-radio
            v-if="$props.type === 'radio'"
            ref="input"
            :option="getValue(option)"
            :disabled="isDisabled(option)"
            :name="$props.name"
            :color="$props.color"
            :left-label="$props.leftLabel"
            :label="getText(option)"
            v-model="selectedValue"
            :tabindex="index"
          />
          <va-checkbox
            v-else-if="$props.type === 'checkbox'"
            ref="input"
            v-model="selectedValue"
            :label="getText(option)"
            :disabled="isDisabled(option)"
            :left-label="$props.leftLabel"
            :array-value="getValue(option)"
            :color="$props.color"
            :name="$props.name"
          />
          <va-switch
            v-else
            ref="input"
            v-model="selectedValue"
            :label="getText(option)"
            :disabled="isDisabled(option)"
            :left-label="$props.leftLabel"
            :array-value="getValue(option)"
            :color="$props.color"
            :name="$props.name"
          />
        </slot>
      </li>
    </ul>
  </VaMessageListWrapper>
</template>

<script lang="ts">
import { Options, Vue, prop, mixins } from 'vue-class-component'

import { generateUniqueId } from '../../services/utils'
import { SelectableListMixin } from '../../mixins/SelectableList/SelectableListMixin'
import { StatefulMixin } from '../../mixins/StatefulMixin/StatefulMixin'
import VaRadio from '../va-radio'
import VaCheckbox from '../va-checkbox'
import VaSwitch from '../va-switch'
import { VaMessageListWrapper } from '../va-input'

class OptionListProps {
  type = prop<string>({
    type: String,
    default: 'checkbox',
    validator: (type: any) => ['radio', 'checkbox', 'switch'].includes(type),
  })

  disabled = prop<boolean>({ type: Boolean, default: false })
  readonly = prop<boolean>({ type: Boolean, default: false })
  defaultValue = prop<string | number | Record<string, unknown> | any[]>({ type: [String, Number, Object, Array] })
  name = prop<string>({ type: String, default: generateUniqueId })
  color = prop<string>({ type: String, default: 'primary' })
  leftLabel = prop<boolean>({ type: Boolean, default: false })
  modelValue = prop<string | number | Record<string, unknown> | any[]>({ type: [String, Number, Object, Array] })
}

const OptionListPropsMixin = Vue.with(OptionListProps)

@Options({
  name: 'VaOptionList',
  components: { VaRadio, VaCheckbox, VaSwitch, VaMessageListWrapper },
})
export default class VaOptionList extends mixins(
  SelectableListMixin,
  StatefulMixin,
  OptionListPropsMixin,
) {
  get isRadio () {
    return this.$props.type === 'radio'
  }

  get selectedValue () {
    const value = this.isRadio ? null : []
    return this.valueComputed || this.$props.defaultValue || value
  }

  set selectedValue (value) {
    if (this.$props.readonly) {
      return
    }
    if (this.isRadio) {
      this.valueComputed = this.getValue(value)
    } else {
      this.valueComputed = Array.isArray(value)
        ? value.map(el => this.getValue(el))
        : [this.getValue(value)]
    }
  }

  getKey (option: any) {
    return this.getTrackBy(option)
  }

  isDisabled (option: any) {
    return this.$props.disabled || this.getDisabled(option)
  }

  reset () {
    this.valueComputed = undefined
  }

  focus () {
    const elements = (this as any).$refs.input
    const firstActiveEl = Array.isArray(elements) && elements.find(el => !el.disabled)
    if (firstActiveEl && typeof firstActiveEl.focus === 'function') {
      firstActiveEl.focus()
    }
  }

  mounted () {
    this.isSelectableListComponent = true
    if (!this.valueComputed && this.$props.defaultValue) {
      this.selectedValue = this.$props.defaultValue
    }
  }
}
</script>

<style lang="scss">
@import 'variables';

.va-option-list {
  &__list {
    line-height: var(--va-option-list-line-height);
  }
}
</style>
