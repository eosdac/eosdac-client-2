<template>
  <div class="rounded-borders shadow-4 col-md-12">
    <q-table
      ref="table"
      color="primary-light"
      :rows-per-page-options="[3, 5, 7, 10, 20, 50, 100]"
      :title="title"
      :data="serverData"
      :columns="columns"
      :filter="filter"
      row-key="name"
      :pagination.sync="serverPagination"
      :loading="loading"
      @request="request"
    >
      <q-td slot="body-cell-from" slot-scope="props" :props="props">
        <!-- {{ props.value }} -->
        <router-link class="a2" :to="{ path: '/profile/' + props.value }">{{
          props.value
        }}</router-link>
      </q-td>

      <q-td slot="body-cell-to" slot-scope="props" :props="props">
        <router-link class="a2" :to="{ path: '/profile/' + props.value }">{{
          props.value
        }}</router-link>
      </q-td>

      <q-td slot="body-cell-memo" slot-scope="props" :props="props">
        <span class="q-caption">{{ $helper.truncate(props.value, 40) }}</span>
      </q-td>

      <q-td slot="body-cell-block_time" slot-scope="props" :props="props">
        {{ props.value.relative }}
      </q-td>

      <q-td slot="body-cell-trx_id" slot-scope="props" :props="props">
        <a
          target="_blank"
          :href="$configFile.get('explorer_transaction').replace('{transaction_id}', props.value)"
        >
          {{ $helper.truncate(props.value, 10) }}
        </a>
      </q-td>

    </q-table>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import '@formatjs/intl-relativetimeformat/polyfill'
import { selectUnit } from '@formatjs/intl-utils'
var rf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' })

export default {
  name: 'TransferTable',
  data () {
    return {
      title: 'Transfers',
      serverData: [],
      filter: '',
      loading: false,
      serverPagination: {
        page: 0,
        rowsPerPage: 10,
        rowsNumber: 5 // specifying this determines pagination is server-side
      },
      columns: [
        {
          name: 'block_num',
          label: '# BLOCK',
          field: 'block_num',
          align: 'left'
          // searchable: true
        },
        {
          name: 'from',
          label: 'FROM',
          field: 'from',
          align: 'left'
          // searchable: true
        },
        {
          name: 'to',
          label: 'TO',
          field: 'to',
          align: 'left'
          // searchable: true
        },
        {
          name: 'quantity',
          label: 'QUANTITY',
          field: 'quantity',
          align: 'left'
        },
        {
          name: 'memo',
          label: 'MEMO',
          field: 'memo',
          align: 'left'
        },

        {
          name: 'block_time',
          label: 'TIME',
          field: 'block_time',
          align: 'left',
          format: val => {
            let secago = Date.now() - new Date(val).getTime()
            const diff = selectUnit(Date.now() - secago)
            let rel = rf.format(diff.value, diff.unit)
            return {
              relative: rel,
              abs: val
            }
          }
        },
        {
          name: 'trx_id',
          label: 'TRXID',
          field: 'trx_id',
          align: 'right'
          // searchable: true
        }
      ]
    }
  },
  computed: {
    ...mapGetters({
    })
  },
  methods: {
    async request (props) {
      this.loading = true
      // console.log(props);
      let res = await this.$store.dispatch('dac/fetchDACTokenTransfers', {
        limit: props.pagination.rowsPerPage,
        skip: props.pagination.rowsPerPage * (props.pagination.page - 1)
      })
      console.log(res)
      this.serverPagination = props.pagination
      this.serverPagination.rowsNumber = res.count
      this.serverData = res.results.map(r => {
        let transfer = r.action.data
        transfer.block_time = r.block_timestamp.split('.')[0] + '.000+00:00'
        transfer.block_num = r.block_num
        transfer.trx_id = r.trx_id
        return transfer
      })
      this.loading = false
    }
  },
  mounted: async function () {
    this.request({
      pagination: this.serverPagination,
      filter: this.filter
    })
  }
}
</script>

<style>
.q-table-top {
  border-top: 0px !important;
}
</style>
