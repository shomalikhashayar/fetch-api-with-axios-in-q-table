<template>
  <div class="">
    <q-table
      :rows="rows"
      :columns="columns"
      row-key="name"
      selection="multiple"
      v-model:selected="selected"
      @selection="handleSelection"
    >
      <template v-slot:header-selection="scope">
        <q-checkbox v-model="scope.selected" />
      </template>

      <template v-slot:body-selection="scope">
        <q-checkbox
          :model-value="scope.selected"
          @update:model-value="
            (val, evt) => {
              Object.getOwnPropertyDescriptor(scope, 'selected').set(val, evt);
            }
          "
        />
      </template>
    </q-table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selected: [],
      columns: [
        {
          name: "#",
          required: true,
          label: "#",
          align: "left",
          field: (row) => row.id,
          format: (val) => `${val}`,
        },
        {
          name: "number",
          align: "left",
          label: "شماره",
          field: "number",
          sortable: true,
          align: "left",
        },
        {
          name: "customer",
          label: "مشتری",
          field: "customer",
          sortable: true,
          align: "left",
        },
        {
          name: "description",
          label: "شرح",
          field: "description",
          align: "left",
        },
        { name: "totalSum", label: "جمع کل", field: "totalSum", align: "left" },
        { name: "discount", label: "تخفیف", field: "discount", align: "left" },
        {
          name: "type",
          label: "نوع",
          field: "type",
          sortable: true,
          sort: (a, b) => parseInt(a, 10) - parseInt(b, 10),
          align: "left",
        },
        {
          name: "status",
          label: "وضعیت",
          field: "status",
          sortable: true,
          sort: (a, b) => parseInt(a, 10) - parseInt(b, 10),
          align: "left",
        },
      ],
      rows: [],
    };
  },
  mounted() {
    this.getRows();
  },

  methods: {
    getRows() {
      this.$axios
        .get("http://localhost:3000/rows")
        .then((res) => {
          this.rows = res.data;
          console.log("rows", res.data);
        })
        .catch((err) => {
          console.log(err);
        });
    },
    handleSelection({ rows, added, evt }) {
      // ignore selection change from header of not from a direct click event
      if (rows.length !== 1 || evt === void 0) {
        return;
      }

      const { oldSelectedRow } = this;
      const [newSelectedRow] = rows;
      const { ctrlKey, shiftKey } = evt;

      if (shiftKey !== true) {
        this.oldSelectedRow = newSelectedRow;
      }

      // wait for the default selection to be performed
      this.$nextTick(() => {
        if (shiftKey === true) {
          const tableRows = this.$refs.table.filteredSortedRows;
          let firstIndex = tableRows.indexOf(oldSelectedRow);
          let lastIndex = tableRows.indexOf(newSelectedRow);

          if (firstIndex < 0) {
            firstIndex = 0;
          }

          if (firstIndex > lastIndex) {
            [firstIndex, lastIndex] = [lastIndex, firstIndex];
          }

          const rangeRows = tableRows.slice(firstIndex, lastIndex + 1);

          this.selected =
            added === true
              ? this.selected.concat(
                  rangeRows.filter(
                    (row) => this.selected.includes(row) === false
                  )
                )
              : this.selected.filter(
                  (row) => rangeRows.includes(row) === false
                );
        } else if (ctrlKey !== true && added === true) {
          this.selected = [newSelectedRow];
        }
      });
    },
  },
};
</script>
