<template>
  <b-card>
    <div class="d-flex justify-content-between align-items-center mb-1">
      <h4 class="card-title">
        Delegators
      </h4>
      <small>Total: {{ totalDelegators }}</small>
    </div>

    <b-table
      :items="paginatedDelegators"
      :fields="fields"
      striped
      responsive
      small
      head-variant="light"
      :busy="loading"
    >
      <!-- Sıra numarası -->
      <template #cell(index)="data">
        {{ (currentPage - 1) * perPage + data.index + 1 }}
      </template>

      <!-- Delegator address -->
      <template #cell(delegator)="data">
        <span :style="highlightStyle(data)">
          <span v-if="getMedal(data)">
            {{ getMedal(data) }}
          </span>
          <router-link :to="`../account/${data.item.delegator}`">
            {{ formatAddress(data.item.delegator) }}
          </router-link>
        </span>
      </template>

      <!-- Amount -->
      <template #cell(amount)="data">
        <span :style="highlightStyle(data)">
          {{ tokenFormatter(data.item.amount) }}
        </span>
      </template>
    </b-table>

    <b-pagination
      v-if="totalPages > 1"
      v-model="currentPage"
      :total-rows="totalDelegators"
      :per-page="perPage"
      align="center"
      size="sm"
      class="mt-2"
      @change="onPageChange"
    />
  </b-card>
</template>

<script>
import { BCard, BTable, BPagination } from 'bootstrap-vue'
import { formatToken, abbrAddress } from '@/libs/utils'

export default {
  components: { BCard, BTable, BPagination },

  props: {
    validatorAddress: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      delegators: [],
      totalFromApi: 0,
      loading: false,
      currentPage: 1,
      perPage: 10,
      fields: [
        { key: 'index', label: '#', sortable: false },
        { key: 'delegator', label: 'Delegator Address', sortable: true },
        { key: 'amount', label: 'Delegated Amount', sortable: true },
      ],
    }
  },

  computed: {
    totalDelegators() {
      return this.totalFromApi
    },
    totalPages() {
      return Math.ceil(this.totalDelegators / this.perPage)
    },
    sortedDelegators() {
      return this.delegators
        .slice()
        .sort((a, b) => Number(b.amount.amount || b.amount) - Number(a.amount.amount || a.amount))
    },
    paginatedDelegators() {
      const start = (this.currentPage - 1) * this.perPage
      const end = start + this.perPage
      return this.sortedDelegators.slice(start, end)
    },
  },

  watch: {
    validatorAddress: {
      immediate: true,
      handler(newVal) {
        // console.log('[Delegators] validatorAddress changed:', { oldVal, newVal })

        if (newVal) this.fetchDelegators()
      },
    },
  },

  methods: {
    fetchDelegators() {
      if (!this.validatorAddress) return
      this.loading = true

      this.$http
        .getValidatorDelegations(this.validatorAddress)
        .then(res => {
          if (!res || !res.delegation_responses) return
          this.delegators = res.delegation_responses.map(d => ({
            delegator: d.delegation.delegator_address,
            amount: d.balance,
          }))

          this.totalFromApi = Number(res.pagination?.total || this.delegators.length)
        })
        .catch(err => console.error('Delegations ERROR:', err))
        // eslint-disable-next-line no-return-assign
        .finally(() => (this.loading = false))
    },

    tokenFormatter(token) {
      return formatToken(token)
    },

    formatAddress: abbrAddress,

    highlightStyle(data) {
      const HIGHLIGHT_COUNT = 3
      const BASE_COLOR = '#ffca00'

      const globalIndex = (this.currentPage - 1) * this.perPage + data.index

      if (globalIndex >= HIGHLIGHT_COUNT) return {}

      const darknessRatio = 1 - (globalIndex / (HIGHLIGHT_COUNT - 1))

      return {
        color: BASE_COLOR,
        opacity: 0.7 + (darknessRatio * 0.9),
        fontWeight: 'bold',
      }
    },

    getMedal(data) {
      const globalIndex = (this.currentPage - 1) * this.perPage + data.index

      switch (globalIndex) {
        case 0:
          return '👑'
        case 1:
          return '🥈'
        case 2:
          return '🥉'
        default:
          return null
      }
    },

    onPageChange(page) {
      this.currentPage = page
    },

  },
}
</script>
