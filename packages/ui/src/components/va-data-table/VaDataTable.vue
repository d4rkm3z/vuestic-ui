<template>
  <va-inner-loading
    class="va-data-table"
    :loading="loading"
    :color="loadingColor"
  >
    <table
      class="va-data-table__table"
      :class="{
        striped,
        selectable,
        hoverable,
      }"
      v-bind="$attrs"
    >
      <colgroup v-if="'colgroup' in slots">
        <slot name="colgroup" v-bind="columnsModel" />
      </colgroup>

      <thead class="va-data-table__table-thead">
        <slot name="headerPrepend" />

        <tr
          v-if="!hideDefaultHeader"
          class="va-data-table__table-tr"
        >
          <th
            v-if="selectable"
            class="va-data-table__table-th"
          >
            <va-checkbox
              v-if="selectMode === 'multiple'"
              :model-value="severalRowsSelected ? 'idl' : allRowsSelected"
              :true-value="true"
              :false-value="false"
              indeterminate-value="idl"
              indeterminate
              @update:model-value="toggleBulkSelection"
              :color="selectedColor"
            />
          </th>

          <th
            v-for="column in columnsModel"
            :key="column.key"
            :title="column.headerTitle"
            @click.exact="column.sortable ? toggleSorting(column): () => {}"
            :style="getHeaderCSSVariables(column)"
            class="va-data-table__table-th"
          >
            <div class="va-data-table__table-th-wrapper">
              <span v-if="`header(${column.key})` in slots">
                <slot :name="`header(${column.key})`" v-bind="column" />
              </span>

              <slot v-else name="header" v-bind="column">
                <span>{{ column.label }}</span>
              </slot>

              <div
                v-if="column.sortable"
                class="va-data-table__table-th-sorting"
                @selectstart.prevent
              >
                <va-icon
                  :name="sortingOrderProxy === 'asc' ? 'expand_less' : 'expand_more'"
                  size="small"
                  class="va-data-table__table-th-sorting-icon"
                  :class="{ active: sortByProxy === column.key && sortingOrderProxy !== null }"
                />
              </div>
            </div>
          </th>
        </tr>

        <slot name="headerAppend" />
      </thead>

      <tbody class="va-data-table__table-tbody">
        <slot name="bodyPrepend" />

        <tr v-if="showNoDataHtml" key="showNoDataHtml">
          <td
            :colspan="columnsModel.length + (selectable ? 1 : 0)"
            v-html="noDataHtml"
            class="no-data"
          />
        </tr>

        <tr v-if="showNoDataFilteredHtml" key="showNoDataFilteredHtml">
          <td
            :colspan="columnsModel.length + (selectable ? 1 : 0)"
            v-html="noDataFilteredHtml"
            class="no-data"
          />
        </tr>

        <transition-group
          :name="animated ? 'table-transition' : null"
          appear
        >
          <tr
            v-for="(row, index) in rows"
            :key="row.initialIndex"
            class="va-data-table__table-tr"
            :class="{
              selectable,
              hoverable,
              selected: isRowSelected(row),
            }"
            :style="rowCSSVariables"
          >
            <template v-if="perPage ? (index >= perPage * (currentPage - 1)) && (index < perPage * currentPage) : true">
              <td
                v-if="selectable"
                class="va-data-table__table-td"
                @selectstart.prevent
              >
                <va-checkbox
                  :model-value="isRowSelected(row)"
                  @click.shift.exact="shiftSelectRows(row)"
                  @click.ctrl.exact="ctrlSelectRow(row)"
                  @click.exact="ctrlSelectRow(row)"
                  :color="selectedColor"
                />
              </td>

              <td
                v-for="cell in row.cells"
                :key="cell.column.key + cell.rowIndex"
                :style="getCellCSSVariables(cell)"
                class="va-data-table__table-td"
              >
                <slot
                  v-if="`cell(${cell.column.key})` in slots"
                  :name="`cell(${cell.column.key})`"
                  v-bind="cell"
                />

                <slot v-else name="cell" v-bind="cell">
                  {{ cell.value }}
                </slot>
              </td>
            </template>
          </tr>
        </transition-group>

        <slot name="bodyAppend" />
      </tbody>

      <tfoot v-if="footerClone" class="va-data-table__table-tfoot">
        <slot name="footerPrepend" />

        <tr v-if="!hideDefaultHeader" class="va-data-table__table-tr">
          <th v-if="selectable" class="va-data-table__table-th">
            <va-checkbox
              v-if="selectMode === 'multiple'"
              :model-value="severalRowsSelected ? 'idl' : allRowsSelected"
              :true-value="true"
              :false-value="false"
              indeterminate-value="idl"
              indeterminate
              @update:model-value="toggleBulkSelection"
              :color="selectedColor"
            />
          </th>

          <th
            v-for="column in columnsModel"
            :key="column.key"
            :title="column.headerTitle"
            @click.exact="allowFooterSorting && column.sortable ? toggleSorting(column) : () => {}"
            :style="getFooterCSSVariables(column)"
            class="va-data-table__table-th"
          >
            <div class="va-data-table__table-th-wrapper">
              <span v-if="`footer(${column.key})` in slots">
                <slot :name="`footer(${column.key})`" v-bind="column" />
              </span>

              <slot v-else name="footer" v-bind="column">
                <span>{{ column.label }}</span>
              </slot>

              <div
                v-if="allowFooterSorting && column.sortable"
                class="va-data-table__table-th-sorting"
                @selectstart.prevent
              >
                <va-icon
                  :name="sortingOrderProxy === 'asc' ? 'expand_less' : 'expand_more'"
                  size="small"
                  class="va-data-table__table-th-sorting-icon"
                  :class="{ active: sortByProxy === column.key && sortingOrderProxy !== null }"
                />
              </div>
            </div>
          </th>
        </tr>

        <slot name="footerAppend" />
      </tfoot>
    </table>
  </va-inner-loading>
</template>

<script lang="ts">
import { computed, defineComponent, PropType, toRefs } from 'vue'
import VaInnerLoading from '../va-inner-loading'
import VaCheckbox from '../va-checkbox'
import VaIcon from '../va-icon'
import useColumns, { TTableColumnSource } from './hooks/useColumns'
import useRows, { ITableItem } from './hooks/useRows'
import useFilterable, { TFilterMethod } from './hooks/useFilterable'
import useSortable, { TSortingOrder } from './hooks/useSortable'
import useSelectable, { TSelectMode } from './hooks/useSelectable'
import useStyleable from './hooks/useStyleable'

/*
  TODO: consider a possibility to lazy-load the hooks with dynamic imports based on respective props' values. E.G.

  if (selectable.value) {
    const { default: useSelectable } = await import("./hooks/useSelectable");
  }

  // Would be a cool feature (if possible at all).
*/

export default defineComponent({
  name: 'VaDataTable',

  components: {
    VaInnerLoading,
    VaCheckbox,
    VaIcon,
  },

  // so that the attributes could be bypassed to the <table> element rather then applied to <va-inner-loading>
  inheritAttrs: false,

  props: {
    columns: {
      type: Array as PropType<TTableColumnSource[]>,
      default: () => [] as TTableColumnSource[],
    },
    items: {
      type: Array as PropType<ITableItem[]>,
      default: () => [] as ITableItem[],
    },
    filter: {
      type: String,
      default: '',
    },
    filterMethod: {
      type: Function as PropType<TFilterMethod>,
    },
    sortBy: { // model-able
      type: String,
    },
    sortingOrder: { // model-able
      type: String as PropType<TSortingOrder>,
    },
    modelValue: { // model-able
      type: Array as PropType<ITableItem[]>,
    },
    hoverable: {
      type: Boolean,
      default: false,
    },
    animated: {
      type: Boolean,
      default: true,
    },
    selectable: {
      type: Boolean,
      default: false,
    },
    selectMode: {
      type: String as PropType<TSelectMode>,
      default: 'multiple',
    },
    selectedColor: {
      type: String,
      default: 'primary',
    },
    perPage: {
      type: Number,
    },
    currentPage: {
      type: Number,
    },
    loading: {
      type: Boolean,
      default: false,
    },
    loadingColor: {
      type: String,
      default: 'primary',
    },
    noDataHtml: {
      type: String,
      default: 'No items',
    },
    noDataFilteredHtml: {
      type: String,
      default: 'No items match the provided filtering condition',
    },
    hideDefaultHeader: {
      type: Boolean,
      default: false,
    },
    footerClone: {
      type: Boolean,
      default: false,
    },
    allowFooterSorting: {
      type: Boolean,
      default: false,
    },
    striped: {
      type: Boolean,
      default: false,
    },
  },

  emits: [
    'update:modelValue', // `modelValue` is selected items
    'update:sortBy',
    'update:sortingOrder',
    'filtered',
    'sorted',
    'selectionChange',
  ],

  setup (props, { slots, emit }) {
    const {
      columns: rawColumns,
      items: rawItems,
    } = toRefs(props)

    const {
      columns,
    } = useColumns(rawColumns, rawItems)

    const {
      rows: unfilteredRows,
    } = useRows(rawItems, columns)

    const {
      filter,
      filterMethod,
    } = toRefs(props)

    const {
      filteredRows: rows,
    } = useFilterable(unfilteredRows, filter, filterMethod, emit)

    const {
      sortBy,
      sortingOrder,
    } = toRefs(props)

    const {
      sortByProxy,
      sortingOrderProxy,
      toggleSorting,
    } = useSortable(columns, rows, sortBy, sortingOrder, emit)

    const {
      selectable,
      selectMode,
      modelValue,
    } = toRefs(props)

    const {
      toggleRowSelection,
      ctrlSelectRow,
      shiftSelectRows,
      toggleBulkSelection,
      isRowSelected,
      severalRowsSelected,
      allRowsSelected,
    } = useSelectable(rows, modelValue, selectable, selectMode, emit)

    const {
      hoverable,
      selectedColor,
      allowFooterSorting,
    } = toRefs(props)

    const {
      getHeaderCSSVariables,
      rowCSSVariables,
      getCellCSSVariables,
      getFooterCSSVariables,
    } = useStyleable(hoverable, selectable, selectedColor, allowFooterSorting)

    const showNoDataHtml = computed(() => {
      return rawItems.value.length < 1
    })

    const showNoDataFilteredHtml = computed(() => {
      return rows.value.length < 1
    })

    return {
      slots,
      columnsModel: columns,
      rows,
      toggleRowSelection,
      ctrlSelectRow,
      shiftSelectRows,
      toggleBulkSelection,
      isRowSelected,
      severalRowsSelected,
      allRowsSelected,
      sortByProxy,
      sortingOrderProxy,
      toggleSorting,
      getHeaderCSSVariables,
      rowCSSVariables,
      getCellCSSVariables,
      getFooterCSSVariables,
      showNoDataHtml,
      showNoDataFilteredHtml,
    }
  },
})
</script>

<style lang="scss">
@import "./variables";
// The calculated variables are taken from a respective element's `style` attribute. See the `useStyleable` hook

.va-data-table {
  overflow-x: auto;
  min-width: unset;

  .va-data-table__table {
    width: 100%;
    cursor: default;
    white-space: nowrap;

    .va-data-table__table-thead {
      border-bottom: var(--va-data-table-thead-border);
    }

    .va-data-table__table-tbody {
      .no-data {
        text-align: var(--va-data-table-no-data-text-align);
        vertical-align: var(--va-data-table-no-data-vertical-align);
      }
    }

    .va-data-table__table-tfoot {
      border-top: var(--va-data-table-thead-border);
    }

    .va-data-table__table-th {
      padding: var(--va-data-table-cell-padding);
      text-align: var(--align);
      vertical-align: var(--vertical-align);
      color: var(--va-data-table-thead-color);
      font-size: var(--va-data-table-thead-font-size);
      line-height: var(--va-data-table-thead-line-height);
      font-weight: var(--va-data-table-thead-font-weight);
      text-transform: var(--va-data-table-thead-text-transform);
      letter-spacing: var(--va-data-table-thead-letter-spacing);
      cursor: var(--cursor);

      .va-data-table__table-th-wrapper {
        display: flex;
        align-items: center;
      }

      .va-data-table__table-th-sorting {
        justify-self: end;
        line-height: 1;
      }

      .va-data-table__table-th-sorting-icon {
        opacity: 0;
        user-select: none;
        pointer-events: none;

        &.active {
          opacity: 1;
          pointer-events: initial;
        }
      }

      span {
        flex-grow: 1;
      }

      &:hover {
        .va-data-table__table-th-sorting-icon:not(.active) {
          opacity: var(--va-data-table-hover-th-opacity);
        }
      }
    }

    .va-data-table__table-td {
      padding: var(--va-data-table-cell-padding);
      text-align: var(--align);
      vertical-align: var(--vertical-align);
    }

    .va-data-table__table-tr {
      &.selectable {
        &:hover {
          background-color: var(--hover-color);
        }

        &.selected:not(:hover) {
          background-color: var(--selected-color);
        }
      }

      &.table-transition-move {
        transition: transform var(--va-data-table-transition);
      }

      &.table-transition-leave-active {
        transition: opacity var(--va-data-table-transition);
      }

      &.table-transition-enter-active {
        transition: opacity var(--va-data-table-transition) 0.3s;
      }

      &.table-transition-enter-from,
      &.table-transition-leave-to {
        opacity: 0;
      }
    }

    &.striped {
      .va-data-table__table-tbody {
        .va-data-table__table-tr:nth-child(2n) {
          background-color: var(--va-light-gray3);
        }
      }
    }

    &.hoverable:not(.selectable) {
      .va-data-table__table-tbody {
        .va-data-table__table-tr {
          &:hover {
            background-color: var(--hover-color);
          }
        }
      }
    }

    &.striped.selectable {
      .va-data-table__table-tbody {
        .va-data-table__table-tr:nth-child(2n) {
          &:hover {
            background-color: var(--hover-color);
          }

          &.selected {
            background-color: var(--selected-color);
          }
        }
      }
    }
  }
}
</style>
