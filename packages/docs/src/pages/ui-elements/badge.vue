<template>
  <docs-content :config="configComputed" />
</template>

<script lang="ts">
import { Vue, Options } from 'vue-class-component'
import DocsContent from '../../components/DocsContent.vue'
import badgeConfig from '../../components/page-configs/ui-elements/va-badge/page-config'

@Options({
  name: 'DocsBadge',
  components: {
    DocsContent,
  },
})
export default class Badge extends Vue {
  get configComputed () {
    return badgeConfig
  }
}
</script>
