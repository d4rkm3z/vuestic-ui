<template>
  <div :class="computedClass">
    <va-dropdown
      v-if="!split"
      :disabled="disabled"
      :position="position"
      :offset="$props.offset"
      :keep-anchor-width="keepAnchorWidth"
      :close-on-content-click="closeOnContentClick"
      v-model="showDropdown"
      :stateful="stateful"
    >
      <template #anchor>
        <va-button
          :size="size"
          :flat="flat"
          :outline="outline"
          :disabled="disabled"
          :color="color"
          :icon-right="computedIcon"
          @click="click"
        >
          <slot name="label">
            {{ label }}
          </slot>
        </va-button>
      </template>

      <va-dropdown-content>
        <slot />
      </va-dropdown-content>
    </va-dropdown>
    <va-button-group
      v-else
      :outline="outline"
      :flat="flat"
    >
      <va-button
        :size="size"
        :disabled="disabled || disableButton"
        :color="color"
        :to="splitTo"
        @click="mainButtonClick"
      >
        <slot name="label">
          {{ label }}
        </slot>
      </va-button>
      <va-dropdown
        :disabled="disabled || disableDropdown"
        :position="position"
        :offset="$props.offset"
        v-model="showDropdown"
        :stateful="stateful"
      >
        <template #anchor>
          <va-button
            :size="size"
            :flat="flat"
            :outline="outline"
            :disabled="disabled || disableDropdown"
            :color="color"
            :icon="computedIcon"
            @click="click"
          />
        </template>
        <va-dropdown-content>
          <slot />
        </va-dropdown-content>
      </va-dropdown>
    </va-button-group>
  </div>
</template>

<script lang="ts">
import { Options, Vue, prop, mixins } from 'vue-class-component'

import ColorMixin from '../../services/color-config/ColorMixin'
import { SizeMixin } from '../../mixins/SizeMixin'

import VaDropdown, { VaDropdownContent } from '../va-dropdown'
import VaButton from '../va-button'
import VaButtonGroup from '../va-button-group'

class ButtonDropdownProps {
  modelValue = prop<boolean>({ type: Boolean, default: false })
  color = prop<string>({ type: String, default: 'primary' })
  outline = prop<boolean>({ type: Boolean, default: false })
  disableButton = prop<boolean>({ type: Boolean, default: false })
  disableDropdown = prop<boolean>({ type: Boolean, default: false })
  flat = prop<boolean>({ type: Boolean, default: false })
  disabled = prop<boolean>({ type: Boolean, default: false })
  size = prop<string>({
    type: String,
    default: 'medium',
    validator: (value: string) => {
      return ['medium', 'small', 'large'].includes(value)
    },
  })

  keepAnchorWidth = prop({ type: Boolean, default: false })
  split = prop<boolean>({ type: Boolean })
  splitTo = prop<string>({ type: String, default: '' })
  icon = prop<string>({ type: String, default: 'expand_more' })
  openedIcon = prop<string>({ type: String, default: 'expand_less' })
  position = prop<string>({ type: String, default: 'bottom' })
  label = prop<string>({ type: String })
  offset = prop<number | number[]>({ type: [Array, Number], default: () => [] })
  closeOnContentClick = prop<boolean>({ type: Boolean, default: true })
  stateful = prop<boolean>({ type: Boolean, default: true })
}

const ButtonDropdownPropsMixin = Vue.with(ButtonDropdownProps)

@Options({
  name: 'VaButtonDropdown',
  components: { VaButtonGroup, VaButton, VaDropdown, VaDropdownContent },
  emits: ['click', 'main-button-click', 'update:modelValue'],
})
export default class VaButtonDropdown extends mixins(
  SizeMixin,
  ColorMixin,
  ButtonDropdownPropsMixin,
) {
  get computedIcon (): string {
    // @ts-ignore
    return this.showDropdown ? this.$props.openedIcon : this.$props.icon
  }

  get computedClass () {
    return {
      'va-button-dropdown': true,
      'va-button-dropdown--split': this.split,
      'va-button-dropdown--normal': this.size === 'normal',
      'va-button-dropdown--large': this.size === 'large',
      'va-button-dropdown--small': this.size === 'small',
    }
  }

  get showDropdown (): boolean {
    return this.modelValue
  }

  set showDropdown (value: boolean) {
    this.$emit('update:modelValue', value)
  }

  click (event: Event): void {
    this.$emit('click', event)
  }

  mainButtonClick (event: Event): void {
    this.$emit('main-button-click', event)
  }
}
</script>

<style lang="scss">
@import "../../styles/resources";
@import "variables";

.va-button-dropdown {
  display: inline-block;

  .va-button {
    margin: var(--va-button-dropdown-button-margin);
  }

  &--split {
    .va-dropdown {
      .va-dropdown__anchor {
        margin: 0;
      }

      .va-button {
        border-top-left-radius: 0;
        border-bottom-left-radius: 0;
        font-size: 1.5rem;
      }
    }

    &.va-button-dropdown--small {
      .va-dropdown .va-button {
        font-size: 1.6rem;
      }
    }

    &.va-button-dropdown--large {
      .va-dropdown .va-button {
        font-size: 1.7rem;
      }
    }
  }
}
</style>
