<template>
  <component
    :is="tagComputed"
    class="va-button"
    :class="computedClass"
    :style="computedStyle"
    :disabled="disabled"
    :type="computedType"
    :href="hrefComputed"
    :target="target"
    :to="to"
    :replace="replace"
    :append="append"
    :active-class="activeClass"
    :exact="exact"
    :exact-active-class="exactActiveClass"
    @focus="updateFocusState(true)"
    @blur="updateFocusState(false)"
    :tabindex="loading ? -1 : 0"
    @mouseenter="updateHoverState(true)"
    @mouseleave="updateHoverState(false)"
    v-on="inputListeners"
  >
    <div
      class="va-button__content"
      :class="{'va-button__content--loading': loading}"
    >
      <va-icon
        v-if="icon"
        :name="icon"
        :size="size"
        class="va-button__left-icon"
      />
      <slot />
      <va-icon
        v-if="iconRight"
        :name="iconRight"
        :size="size"
        class="va-button__right-icon"
      />
    </div>
    <va-progress-circle
      v-if="loading"
      class="va-button__loader"
      indeterminate
      :size="loaderSize"
      :color="computedStyle.color"
      :thickness="0.15"
    />
  </component>
</template>

<script lang="ts">
import { watchEffect, watch } from 'vue'
import { mixins, Vue, prop, Options, setup } from 'vue-class-component'
import {
  getGradientBackground,
  getFocusColor,
  getHoverColor,
  getBoxShadowColor,
  shiftHSLAColor,
} from '../../services/color-config/color-functions'
import ColorMixin from '../../services/color-config/ColorMixin'
import { RouterLinkMixin } from '../../mixins/RouterLinkMixin/RouterLinkMixin'
import { SizeMixin } from '../../mixins/SizeMixin'
import { LoadingMixin } from '../../mixins/LoadingMixin/LoadingMixin'
import VaIcon from '../va-icon'
import { VaProgressCircle } from '../va-progress-bar'

class ButtonProps {
  color = prop<string>({ type: String, default: undefined })
  textColor = prop<string>({ type: String, default: undefined })
  tag = prop<string>({ type: String, default: 'button' })
  outline = prop<boolean>({ type: Boolean, default: undefined })
  gradient = prop<boolean>({ type: Boolean, default: undefined })
  flat = prop<boolean>({ type: Boolean, default: undefined })
  type = prop<string>({ type: String, default: 'button' })
  disabled = prop<boolean>({ type: Boolean, default: false })
  block = prop<boolean>({ type: Boolean, default: false })
  rounded = prop<boolean>({ type: Boolean, default: true })
  round = prop<boolean>({ type: Boolean, default: undefined })
  spaceBetweenItems = prop<boolean>({ type: Boolean, default: undefined })
  icon = prop<string>({ type: String, default: undefined })
  iconRight = prop<string>({ type: String, default: undefined })
  size = prop<string>({
    type: String,
    default: 'medium',
    validator: (value: string) => {
      return ['medium', 'small', 'large'].includes(value)
    },
  })
}

const ButtonPropsMixin = Vue.with(ButtonProps)

@Options({
  name: 'VaButton',
  components: { VaIcon, VaProgressCircle },
  emits: ['click'],
})
export default class VaButton extends mixins(
  ColorMixin,
  RouterLinkMixin,
  SizeMixin,
  LoadingMixin,
  ButtonPropsMixin,
) {
  hoverState = false
  focusState = false

  context = setup(() => {
    watch(() => this.$props.loading, (loading) => {
      if (loading) {
        this.updateFocusState(false)
        this.updateHoverState(false)
      }
    })

    watchEffect(() => {
      this.updateFocusState(this.focusState)
      this.updateHoverState(this.hoverState)
    })

    return {}
  })

  get computedType () {
    // Safari issue. type===button will break styles if the button is used as a link
    switch (this.tagComputed) {
    case 'a':
    case 'router-link':
    case 'nuxt-link':
      return undefined
    default:
      return this.type
    }
  }

  get isTransparentBackground () {
    return this.outline || this.flat
  }

  get colorComputed () {
    return this.computeColor(this.color, 'primary')
  }

  get textColorComputed () {
    if (this.$props.textColor !== undefined) {
      return this.computeColor(this.textColor, '#fff')
    }

    if (this.isTransparentBackground) {
      return this.computeColor(this.colorComputed, '#fff')
    }

    return this.computeColor(this.textColor, '#fff')
  }

  get hasDefaultSlot () {
    return this.$slots.default
  }

  get computedClass () {
    return {
      'va-button--default': !this.flat && !this.outline && !this.disabled,
      'va-button--flat': this.flat,
      'va-button--outline': this.outline,
      'va-button--disabled': this.disabled,
      'va-button--hover': this.hoverState,
      'va-button--focus': this.focusState,
      'va-button--large': this.size === 'large',
      'va-button--small': this.size === 'small',
      'va-button--normal': !this.size || this.size === 'medium',
      'va-button--loading': this.loading,
      'va-button--block': this.block,
      'va-button--square': !this.rounded,
      'va-button--round': this.round || !this.hasDefaultSlot,
      'va-button--space-between-items': this.spaceBetweenItems,
    }
  }

  get loaderSize () {
    const size = /([0-9]*)(px)/.exec(this.sizeComputed) as null | [string, string, string]

    if (size) {
      return `${+size[1] / 2}${size[2]}`
    }

    return this.sizeComputed
  }

  get computedStyle () {
    const color = this.textColorComputed
    const borderColor = this.outline ? this.colorComputed : ''
    let background = this.isTransparentBackground ? '#00000000' : this.gradient ? getGradientBackground(this.colorComputed) : this.colorComputed
    if (this.hoverState) {
      const alpha = this.outline ? -0.9 : -0.8
      const lightness = 5
      const color = this.isTransparentBackground ? shiftHSLAColor(this.colorComputed, { a: alpha }) : shiftHSLAColor(this.colorComputed, { l: lightness })
      background = this.gradient ? getGradientBackground(color) : color
    }
    if (this.focusState) {
      const alpha = this.outline ? -0.8 : -0.7
      const lightness = 10
      const color = this.isTransparentBackground ? shiftHSLAColor(this.colorComputed, { a: alpha }) : shiftHSLAColor(this.colorComputed, { l: lightness })
      background = this.gradient ? getGradientBackground(color) : color
    }
    return {
      color: color,
      borderColor: borderColor,
      background: background,
    }
  }

  get inputListeners () {
    // vue3 $listeners.click -> $attrs.onClick
    return Object.assign({},
      this.$attrs,
      {
        click: (event: Event) => {
          this.$emit('click', event)
        },
      },
    )
  }

  updateHoverState (isHover: boolean) {
    this.hoverState = isHover
  }

  updateFocusState (isFocused: boolean) {
    this.focusState = isFocused
  }

  /** @public */
  focus (): void {
    (this.$el as HTMLElement).focus()
  }

  /** @public */
  blur (): void {
    (this.$el as HTMLElement).blur()
  }
}
</script>

<style lang='scss'>
@import '../../styles/resources';
@import "variables";

.va-button {
  display: var(--va-button-display);
  align-items: var(--va-button-align-items);
  justify-content: var(--va-button-justify-content);
  background-image: var(--va-button-background-image);
  box-shadow: var(--va-button-box-shadow, var(--va-control-box-shadow));
  outline: var(--va-button-outline);
  border: var(--va-button-border, var(--va-control-border));
  font-family: var(--va-button-font-family, var(--va-font-family));
  text-decoration: none !important;
  text-transform: initial;
  cursor: pointer;
  transition: var(--va-button-transition, var(--va-swing-transition));
  background-color: var(--va-button-background-color, var(--va-white));
  vertical-align: middle;
  box-sizing: border-box;
  font-weight: var(--va-button-font-weight);
  margin: var(--va-button-margin);
  padding: var(--va-button-padding);
  position: relative;

  &__content {
    display: flex;
    align-items: center;
    height: 100%;

    &__title,
    &__icon {
      display: flex;
      justify-content: center;
      align-items: center;
      margin: auto;
      white-space: nowrap;
    }

    &--loading {
      opacity: 0;
    }
  }

  &--default {
    color: var(--va-button-background-color, var(--va-white));

    &:focus,
    &:active {
      //filter: brightness(85%);
    }

    i {
      color: var(--va-button-icon-color, var(--va-white));
    }
  }

  &--outline {
    background-color: transparent;
    border: solid var(--va-button-outline-border, var(--va-outline-border-width));
    text-decoration: none;

    .va-button__content {
      margin: calc(var(--va-button-outline-border, var(--va-outline-border-width)) * -1);
    }

    &.va-button--disabled {
      background: transparent;

      @include va-disabled;

      &.va-button--active {
        .va-button__content,
        i {
          color: var(--va-button-outline-icon-color, var(--va-white)) !important;
        }
      }
    }
  }

  &--flat {
    background: transparent;
    border: var(--va-button-flat-border, var(--va-control-border)) solid transparent;
    text-decoration: none;
  }

  &.va-button--disabled {
    @include va-disabled;
  }

  &--large {
    @include va-button(var(--va-button-lg-content-py), var(--va-button-lg-content-px), var(--va-button-lg-font-size), var(--va-button-lg-line-height), var(--va-button-lg-border-radius));

    letter-spacing: var(--va-button-lg-letter-spacing);
    min-height: var(--va-button-lg-size);
    min-width: var(--va-button-lg-size);

    .va-button__content {
      padding: var(--va-button-lg-content-py) var(--va-button-lg-content-px);
    }

    &.va-button--round {
      width: var(--va-button-lg-size);
      height: var(--va-button-lg-size);
    }

    &.va-button--outline {
      line-height: calc(var(--va-button-lg-line-height) - 2 * var(--va-button-outline-border, var(--va-outline-border-width)));
    }

    &.va-button--square {
      border-radius: var(--va-button-lg-square-border-radius);
    }

    .va-button__left-icon {
      margin-left: calc(var(--va-button-lg-content-px) / -2);
      margin-right: calc(var(--va-button-lg-space-between-items) / 2);
    }

    .va-button__right-icon {
      margin-left: calc(var(--va-button-lg-space-between-items) / 2);
      margin-right: calc(var(--va-button-lg-content-px) / -2);
    }
  }

  &--small {
    @include va-button(var(--va-button-sm-content-py), var(--va-button-sm-content-px), var(--va-button-sm-font-size), var(--va-button-sm-line-height), var(--va-button-sm-border-radius));

    letter-spacing: var(--va-button-sm-letter-spacing);
    min-height: var(--va-button-sm-size);
    min-width: var(--va-button-sm-size);

    .va-button__content {
      padding: var(--va-button-sm-content-py) var(--va-button-sm-content-px);
    }

    &.va-button--round {
      width: var(--va-button-sm-size);
      height: var(--va-button-sm-size);
    }

    &.va-button--outline {
      line-height: calc(var(--va-button-sm-line-height) - 2 * var(--va-button-outline-border, var(--va-outline-border-width)));
    }

    &.va-button--square {
      border-radius: var(--va-button-sm-square-border-radius);
    }

    .va-button__left-icon {
      margin-left: calc(var(--va-button-sm-content-px) / -2);
      margin-right: calc(var(--va-button-sm-space-between-items) / 2);
    }

    .va-button__right-icon {
      margin-left: calc(var(--va-button-sm-space-between-items) / 2);
      margin-right: calc(var(--va-button-sm-content-px) / -2);
    }
  }

  &--normal {
    @include va-button(var(--va-button-content-py), var(--va-button-content-px), var(--va-button-font-size), var(--va-button-line-height), var(--va-button-border-radius));

    letter-spacing: var(--va-button-letter-spacing, var(--va-letter-spacing));
    min-height: var(--va-button-size);
    min-width: var(--va-button-size);

    .va-button__content {
      padding: var(--va-button-content-py) var(--va-button-content-px);
      line-height: var(--va-button-line-height);
    }

    &.va-button--round {
      width: var(--va-button-size);
      height: var(--va-button-size);
    }

    &.va-button--outline {
      line-height: calc(var(--va-button-line-height) - 2 * var(--va-button-outline-border, var(--va-outline-border-width)));
    }

    &.va-button--square {
      border-radius: var(--va-button-square-border-radius);
    }

    .va-button__left-icon {
      margin-left: calc(var(--va-button-content-px) / -2);
      margin-right: calc(var(--va-button-space-between-items) / 2);
    }

    .va-button__right-icon {
      margin-left: calc(var(--va-button-space-between-items) / 2);
      margin-right: calc(var(--va-button-content-px) / -2);
    }
  }

  &--round {
    .va-button__content {
      padding: 0;
    }

    .va-button__left-icon {
      margin-left: 0;
      margin-right: 0;
    }

    .va-button__right-icon {
      margin-left: 0;
      margin-right: 0;
    }
  }

  &--space-between-items {
    .va-button__content > * {
      margin-right: calc(var(--va-button-space-between-items) / 2);
      margin-left: calc(var(--va-button-space-between-items) / 2);

      &:last-child {
        margin-right: 0;
      }

      &:first-child {
        margin-left: 0;
      }
    }
  }

  &--loading {
    pointer-events: none;
  }

  &--block {
    display: flex;
    min-width: 100%;
  }

  &--square {
    border-radius: 0.5rem;
  }

  &__loader {
    position: absolute;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }
}
</style>
