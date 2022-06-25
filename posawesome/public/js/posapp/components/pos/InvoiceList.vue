<template>
  <v-row justify="center">
    <v-dialog v-model="invoicesDialog" max-width="800px" min-width="800px">
      <v-card>
        <v-card-title>
          <span class="headline indigo--text">{{__('Invoices')}}</span>
        </v-card-title>
        <v-container>
          <v-row class="mb-4">
          </v-row>
          <v-row>
            <v-col cols="12" class="pa-1" v-if="dialog_data">
              <template>
                <v-data-table
                  :headers="headers"
                  :items="dialog_data"
                  item-key="name"
                  class="elevation-1"
                  :single-select="singleSelect"
                  show-select
                  v-model="selected"
                >
                  <template v-slot:item.grand_total="{ item }">{{
                    formtCurrency(item.grand_total)
                  }}</template>
                </v-data-table>
              </template>
            </v-col>
          </v-row>
        </v-container>
        <v-card-actions class="mt-4">
          <v-spacer></v-spacer>
          <v-btn color="error mx-2" dark @click="close_dialog">Close</v-btn>
          <v-btn
            v-if="selected.length"
            color="primary"
            dark
            @click="submit_dialog"
            >{{__('View')}}</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import { evntBus } from '../../bus';
export default {
  data: () => ({
    invoicesDialog: false,
    singleSelect: true,
    selected: [],
    dialog_data: '',
    company: '',
    invoice_name: '',
    headers: [
      {
        text: __('Customer'),
        value: 'customer_name',
        align: 'start',
        sortable: true,
      },
      {
        text: __('Entregar a'),
        value: 'posa_notes',
        align: 'start',
        sortable: true,
      },
      {
        text: __('Orden'),
        value: 'po_no',
        align: 'start',
        sortable: true,
      },
      {
        text: __('Date'),
        align: 'start',
        sortable: true,
        value: 'posting_date',
      },
      {
        text: __('Invoice'),
        value: 'name',
        align: 'start',
        sortable: true,
      },
      {
        text: __('Amount'),
        value: 'grand_total',
        align: 'start',
        sortable: false,
      },
    ],
  }),
  watch: {},
  methods: {
    close_dialog() {
      this.invoicesDialog = false;
    },
    search_invoices() {
      const vm = this;
      frappe.call({
        method: 'posawesome.posawesome.api.posapp.get_today_invoices',
        args: {},
        async: true,
        callback: function (r) {
          if (r.message) {
            vm.dialog_data = r.message;
          }
        },
      });
    },
    submit_dialog() {
      if (this.selected.length > 0) {
        const return_doc = this.selected[0];
        const data = { invoice_doc: return_doc, return_doc };
        evntBus.$emit('load_invoice', data);
        evntBus.$emit('toggle_view_only');
        this.invoicesDialog = false;
      }
    },
    formtCurrency(value) {
      value = parseFloat(value); 
      return value.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,');
    },
  },
  created: function () {
    evntBus.$on('open_invoices', (data) => {
      this.invoicesDialog = true;
      this.search_invoices()
      this.company = data;
      this.invoice_name = '';
      this.dialog_data = '';
      this.selected = [];
    });
  },
};
</script>