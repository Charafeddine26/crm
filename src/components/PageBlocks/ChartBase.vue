<template>
  <wrap
    v-bind="$props"
    v-on="$listeners"
  >
    <chart-component
      v-if="chart"
      :chart="chart"
      :reporter="reporter"
    />
  </wrap>
</template>
<script>
import { mapActions } from 'vuex'
import base from './base'
import ChartComponent from '../Chart'
import { NoID } from '@cortezaproject/corteza-js'
import { evaluatePrefilter } from 'corteza-webapp-compose/src/lib/record-filter'

export default {
  i18nOptions: {
    namespaces: 'notification',
  },

  components: {
    ChartComponent,
  },
  extends: base,

  data () {
    return {
      chart: null,
    }
  },

  mounted () {
    const { chartID } = this.options

    if (chartID === NoID) {
      return
    }

    const { namespaceID } = this.namespace
    this.findChartByID({ chartID, namespaceID }).then((chart) => {
      this.chart = chart
    }).catch(this.toastErrorHandler(this.$t('chart.loadFailed')))
  },

  methods: {
    ...mapActions({
      findChartByID: 'chart/findByID',
    }),

    reporter (r) {
      const nr = { ...r }
      if (nr.filter) {
        nr.filter = evaluatePrefilter(nr.filter, {
          record: this.record,
          recordID: (this.record || {}).recordID || NoID,
          ownerID: (this.record || {}).userID || NoID,
          userID: (this.$auth.user || {}).userID || NoID,
        })
      }

      const { namespaceID } = this.namespace
      return this.$ComposeAPI.recordReport({ namespaceID, ...nr })
    },
  },
}
</script>
