<script setup lang="ts">
import {
  QResizeObserver,
  QScrollAreaProps,
  QTable,
  QTableColumn,
  QTableProps,
} from 'quasar';

const report = ref<{
  height: number;
  width: number;
} | null>(null);

const onResize: QResizeObserver['onResize'] = (size) => {
  report.value = size;
};
const columns: QTableColumn<Row>[] = [
  {
    name: 'id',
    required: true,
    label: 'ID',
    align: 'left',
    field: (row) => row.name,
    format: (val) => `${val}`,
    sortable: true,
  },
  { name: 'name', align: 'left', label: 'Name', field: 'name', sortable: true },
  {
    name: 'description',
    label: 'Description',
    field: 'description',
    sortable: true,
  },
];

type Row = { id: number; name: string; description: string };
const originalRows: Row[] = [
  { id: 1, name: 'Frozen Yogurt', description: 'Frozen Yogurt' },
  { id: 2, name: 'Ice cream sandwich', description: 'Ice cream sandwich' },
  { id: 3, name: 'Eclair', description: 'Frozen Yogurt' },
  { id: 4, name: 'Cupcake', description: 'Frozen Yogurt' },
  { id: 5, name: 'Gingerbread', description: 'Frozen Yogurt' },
  { id: 6, name: 'Jelly bean', description: 'Frozen Yogurt' },
  { id: 7, name: 'Lollipop', description: 'Frozen Yogurt' },
  { id: 8, name: 'Honeycomb', description: 'Frozen Yogurt' },
  { id: 9, name: 'Donut', description: 'Frozen Yogurt' },
  { id: 10, name: 'KitKat', description: 'Frozen Yogurt' },
  { id: 11, name: 'Frozen Yogurt-1', description: 'Frozen Yogurt' },
  { id: 12, name: 'Ice cream sandwich-1', description: 'Ice cream sandwich-1' },
  { id: 13, name: 'Eclair-1', description: 'Eclair-1' },
  { id: 14, name: 'Cupcake-1', description: 'Cupcake-1' },
  { id: 15, name: 'Gingerbread-1', description: 'Gingerbread-1' },
  { id: 16, name: 'Jelly bean-1', description: 'Jelly bean-1' },
  { id: 17, name: 'Lollipop-1', description: 'Lollipop-1' },
  { id: 18, name: 'Honeycomb-1', description: 'Honeycomb-1' },
  { id: 19, name: 'Donut-1', description: 'Donut-1' },
  { id: 20, name: 'KitKat-1', description: 'KitKat-1' },
  { id: 21, name: 'Frozen Yogurt-2', description: 'Frozen Yogurt-2' },
  { id: 22, name: 'Ice cream sandwich-2', description: 'Ice cream sandwich-2' },
  { id: 23, name: 'Eclair-2', description: 'Eclair-2' },
  { id: 24, name: 'Cupcake-2', description: 'Cupcake-2' },
  { id: 25, name: 'Gingerbread-2', description: 'Gingerbread-2' },
  { id: 26, name: 'Jelly bean-2', description: 'Jelly bean-2' },
  { id: 27, name: 'Lollipop-2', description: 'Lollipop-2' },
  { id: 28, name: 'Honeycomb-2', description: 'Honeycomb-2' },
  { id: 29, name: 'Donut-2', description: 'Donut-2' },
  { id: 30, name: 'KitKat-2', description: 'KitKat-2' },
  { id: 31, name: 'Frozen Yogurt-3', description: 'Frozen Yogurt-3' },
  { id: 32, name: 'Ice cream sandwich-3', description: 'Ice cream sandwich-3' },
  { id: 33, name: 'Eclair-3', description: 'Eclair-3' },
  { id: 34, name: 'Cupcake-3', description: 'Cupcake-3' },
  { id: 35, name: 'Gingerbread-3', description: 'Gingerbread-3' },
  { id: 36, name: 'Jelly bean-3', description: 'Jelly bean-3' },
  { id: 37, name: 'Lollipop-3', description: 'Lollipop-3' },
  { id: 38, name: 'Honeycomb-3', description: 'Honeycomb-3' },
  { id: 39, name: 'Donut-3', description: 'Donut-3' },
  { id: 40, name: 'KitKat-3', description: 'KitKat-3' },
];

import { ref, onMounted } from 'vue';

const splitterModel = ref(50);

const tableRef = ref<QTable | null>(null);
const rows = ref<Row[]>([]);
const filter = ref('');
const loading = ref(false);
const pagination = ref({
  sortBy: 'desc',
  descending: false,
  page: 1,
  rowsPerPage: 10,
  rowsNumber: 0,
});

const fetchFromServer = (
  startRow: number,
  count: number,
  filter: string,
  sortBy: keyof Row,
  descending: boolean
) => {
  const data = filter
    ? originalRows.filter((row) => row.name.includes(filter))
    : originalRows.slice();

  // handle sortBy
  if (sortBy) {
    const sortFn =
      sortBy === 'description'
        ? descending
          ? (a: Row, b: Row) => (a.name > b.name ? -1 : a.name < b.name ? 1 : 0)
          : (a: Row, b: Row) => (a.name > b.name ? 1 : a.name < b.name ? -1 : 0)
        : descending
        ? (a: Row, b: Row) =>
            parseFloat(String(b[sortBy])) - parseFloat(String(a[sortBy]))
        : (a: Row, b: Row) =>
            parseFloat(String(a[sortBy])) - parseFloat(String(b[sortBy]));
    data.sort(sortFn);
  }

  return data.slice(startRow, startRow + count);
};

const getRowsNumberCount = (filter: string) => {
  if (!filter) {
    return originalRows.length;
  }
  let count = 0;
  originalRows.forEach((treat) => {
    if (treat.name.includes(filter)) {
      ++count;
    }
  });
  return count;
};

const onRequest: QTableProps['onRequest'] = (props) => {
  const { page, rowsPerPage, sortBy, descending } = props.pagination;
  const filter = props.filter;

  loading.value = true;

  // emulate server
  setTimeout(() => {
    // update rowsCount with appropriate value
    pagination.value.rowsNumber = getRowsNumberCount(filter);

    // get all rows if "All" (0) is selected
    const fetchCount =
      rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage;

    // calculate starting row of data
    const startRow = (page - 1) * rowsPerPage;

    // fetch data from "server"
    const returnedData = fetchFromServer(
      startRow,
      fetchCount,
      filter,
      sortBy as keyof Row,
      descending
    );

    // clear out existing data and add new
    // rows.value.splice(0, rows.value.length, ...returnedData)
    rows.value = returnedData;

    // don't forget to update local pagination object
    pagination.value.page = page;
    pagination.value.rowsPerPage = rowsPerPage;
    pagination.value.sortBy = sortBy;
    pagination.value.descending = descending;

    // ...and turn of loading indicator
    loading.value = false;
  }, 1500);
};

// const params = ref({});

onMounted(() => {
  // get initial data from server (1st page)
  tableRef.value?.requestServerInteraction();
});

const scrollAreaBindings: QScrollAreaProps = {
  thumbStyle: {
    right: '4px',
    borderRadius: '5px',
    backgroundColor: '#027be3',
    width: '5px',
    opacity: '0.75',
  },
  barStyle: {
    right: '2px',
    borderRadius: '9px',
    backgroundColor: '#027be3',
    width: '9px',
    opacity: '0.2',
  },
};
</script>
<template>
  <q-page padding>
    <q-splitter
      v-model="splitterModel"
      horizontal
      style="height: 87vh"
      after-class="after-b"
    >
      <template v-slot:before>
        <div class="q-pa-md column full-height">
          <q-scroll-area class="col" v-bind="scrollAreaBindings">
            <div class="text-h4 q-mb-md">Before</div>
            <div v-for="n in 20" :key="n" class="q-my-md">
              {{ n }}. Lorem ipsum dolor sit, amet consectetur adipisicing elit.
              Quis praesentium cumque magnam odio iure quidem, quod illum
              numquam possimus obcaecati commodi minima assumenda consectetur
              culpa fuga nulla ullam. In, libero.
            </div>
          </q-scroll-area>
        </div>
      </template>

      <template v-slot:after>
        <div class="q-pa-md">
          <q-table
            flat
            bordered
            style="height: 300px"
            ref="tableRef"
            title="Treats"
            :rows="rows"
            :columns="columns"
            row-key="id"
            v-model:pagination="pagination"
            :loading="loading"
            :filter="filter"
            binary-state-sort
            @request="onRequest"
            :style="report ? `max-height:${report.height - 32}px` : void 0"
          >
            <template v-slot:top-right>
              <q-input
                borderless
                dense
                debounce="300"
                v-model="filter"
                placeholder="Search"
              >
                <template v-slot:append>
                  <q-icon name="search" />
                </template>
              </q-input>
            </template>
          </q-table>
        </div>
        <q-resize-observer @resize="onResize" />
      </template>
    </q-splitter>
    <div v-if="report" class="q-gutter-sm">
      Reported:
      <q-badge>width: {{ report.width }}</q-badge>
      <q-badge>height: {{ report.height }}</q-badge>
    </div>
  </q-page>
</template>

<style lang="scss" scoped>
:deep(.after-b) {
  overflow-y: hidden;
}
</style>
