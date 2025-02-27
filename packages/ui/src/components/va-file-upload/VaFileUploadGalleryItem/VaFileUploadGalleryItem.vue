<template>
  <div
    v-if="removed"
    class="va-file-upload-gallery-item"
    :class="{'va-file-upload-gallery-item--undo': removed}"
  >
    <va-file-upload-undo
      class="va-file-upload-gallery-item--undo"
      @recover="recoverImage"
    />
  </div>

  <div
    v-else
    tabindex="0"
    class="va-file-upload-gallery-item"
    :class="{
      'file-upload-gallery-item_not-image': !this.previewImage,
      'va-file-upload-gallery-item--focused': isFocused
    }"
    @focus="isFocused = true"
    @blur="isFocused = false"
  >
    <img
      v-if="previewImage"
      :src="previewImage"
      alt=""
      class="va-file-upload-gallery-item__image"
    >
    <div
      class="va-file-upload-gallery-item__overlay"
    >
      <div class="va-file-upload-gallery-item__overlay-background" :style="overlayStyles" />
      <div
        class="va-file-upload-gallery-item__name"
        :title="file.name"
      >
        {{ file.name }}
      </div>
      <va-button
        flat
        color="danger"
        icon="delete_outline"
        class="va-file-upload-gallery-item__delete"
        @click="removeImage()"
        @focus="isFocused = true"
        @blur="isFocused = false"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { watch } from 'vue'
import { Options, Vue, prop, mixins } from 'vue-class-component'

import { colorToRgba } from '../../../services/color-config/color-functions'
import VaIcon from '../../va-icon'

import VaFileUploadUndo from '../VaFileUploadUndo'

class FileUploadGalleryItemProps {
  file = prop<any>({
    type: Object,
    default: null,
  })

  color = prop<string>({
    type: String,
    default: 'success',
  })
}

const FileUploadGalleryItemPropsMixin = Vue.with(FileUploadGalleryItemProps)

@Options({
  name: 'VaFileUploadGalleryItem',
  components: {
    VaIcon,
    VaFileUploadUndo,
  },
  emits: ['remove'],
})
export default class VaFileUploadGalleryItem extends mixins(FileUploadGalleryItemPropsMixin) {
  previewImage = ''
  removed = false
  isFocused = false

  created () {
    watch(() => this.$props.file, () => {
      this.convertToImg()
    })
  }

  get overlayStyles () {
    return {
      backgroundColor: colorToRgba(this.$props.color as string, 0.7),
    }
  }

  removeImage () {
    this.removed = true
    setTimeout(() => {
      if (this.removed) {
        this.$emit('remove')
        this.removed = false
      }
    }, 2000)
  }

  recoverImage () {
    this.removed = false
  }

  convertToImg () {
    if (!this.$props.file.name) { return }

    if (this.$props.file.image && this.$props.file.image.url) {
      this.previewImage = this.$props.file.image.url
    } else {
      const reader = new FileReader()
      reader.readAsDataURL(this?.$props?.file?.image)
      reader.onload = (e: any) => {
        if (e.target.result.includes('image')) {
          this.previewImage = e.target.result
        }
      }
    }
  }

  mounted () {
    this.convertToImg()
  }
}
</script>

<style lang='scss'>
@import 'variables';
@import '../../../styles/resources';

$max-image-size: 8.5714rem;

.va-file-upload-gallery-item {
  display: flex;
  position: relative;
  margin-bottom: 1rem;
  margin-right: 0.5rem;
  flex-basis: calc(14.2857% - 0.5rem);
  max-width: calc(14.2857% - 0.5rem);
  border-radius: 0.375rem;
  overflow: hidden;
  width: 100%;
  align-items: stretch;

  @include media-breakpoint-down(md) {
    flex-basis: calc(16.667% - 0.5rem);
    max-width: calc(16.667% - 0.5rem);
  }

  @include media-breakpoint-down(sm) {
    flex-basis: calc(20% - 0.5rem);
    max-width: calc(20% - 0.5rem);
  }

  @include media-breakpoint-down(xs) {
    flex-basis: calc(50% - 0.5rem);
    max-width: calc(50% - 0.5rem);
  }

  &:last-of-type {
    margin-right: 0;
  }

  &:hover,
  &:focus,
  &--focused {
    .va-file-upload-gallery-item__overlay {
      z-index: 3;
      opacity: 1;
    }

    .va-file-upload-gallery-item {
      &__name {
        color: var(--va-file-upload-gallery-item-text-hover);
      }
    }
  }

  &__overlay {
    display: flex;
    position: absolute;
    height: 100%;
    width: 100%;
    top: 0;
    left: 0;
    flex-direction: column;
    padding: 0.5rem;
    z-index: -1;
    opacity: 0;
  }

  &__overlay-background {
    position: absolute;
    height: 100%;
    width: 100%;
    top: 0;
    left: 0;
    z-index: -1;
  }

  &__image {
    width: 100%;
    box-shadow: var(--va-box-shadow);
    object-fit: cover;
    z-index: 1;
  }

  &__name {
    color: var(--va-file-upload-gallery-item-text);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    font-size: 0.875rem;
  }

  &__delete {
    cursor: pointer;
    font-size: 1.5rem;
    margin-top: auto;
  }

  &--undo {
    box-shadow: none;

    .va-file-upload-gallery-item--undo {
      padding: 0.5rem;
      display: flex;
      flex-direction: column;
      font-size: 0.875rem;
      height: 100%;
      justify-content: space-between;
      align-items: flex-start;

      span {
        margin-right: 0.5rem;
      }

      .va-button {
        margin: 0;
      }
    }
  }
}

.file-upload-gallery-item_not-image {
  .file-upload-gallery-item__overlay {
    display: flex;
  }
}
</style>
