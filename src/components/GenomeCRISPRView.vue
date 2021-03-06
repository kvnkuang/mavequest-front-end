<template>
  <div class="card has-table-padding in-paragraph in-list">
    <b-table
      :data="genomeCRISPRData"
      paginated
      per-page="10"
      pagination-simple
      hoverable
      narrowed
      detailed
    >
      <template slot="bottom-left">
        Click&nbsp;
        <b-icon 
          pack="fas" type="is-link" size="is-small" icon="chevron-right">
        </b-icon>&nbsp;to show the percentage of records that are hits.
      </template>

      <template slot-scope="props">
        <b-table-column field="source" label="Pubmed" width="100">
          <a
            :href="'https://www.ncbi.nlm.nih.gov/pubmed/' + 
                              props.row.source"
            target="_blank"
          >
            <span>{{props.row.source}}</span>
          </a>
        </b-table-column>

        <b-table-column field="condition" label="Condition" width="300">{{props.row.condition}}</b-table-column>

        <b-table-column field="screen" label="Screen" width="150">{{props.row.screen}}</b-table-column>

        <b-table-column field="cellline" label="Cell Lines">
          <ExpandableRow :elements="props.row.cellline" preview_items="5"></ExpandableRow>
        </b-table-column>
      </template>

      <template slot="detail" slot-scope="props">
        <VueApexCharts type="bar" :series="getSeries(props)" 
          :height="getHeight(props)" :options="chartOptions">
        </VueApexCharts>
      </template>
    </b-table>
  </div>
</template>

<script>
import ExpandableRow from "@/components/ExpandableRow.vue";
import VueApexCharts from "vue-apexcharts";

export default {
  name: "genome-crispr-view",
  components: {
    ExpandableRow,
    VueApexCharts
  },
  props: {
    genomeCRISPRData: Array,
    genomeCRISPRStats: Object
  },
  data() {
    return {
      chartOptions: {
        chart: {
          toolbar: { show: false },
        },
        dataLabels: { enabled: false },
        plotOptions: {
          bar: {
            horizontal: true
          }
        },
        grid: {
          show: true,
          xaxis: { lines: { show: false } },
          yaxis: { lines: { show: false } },
        },
        title: {
          text: "Percentage of records that are hits",
          margin: 0,
          floating: true,
          style: {
            fontSize: "16px"
          }
        },
        tooltip: {
          x: { show: false },
          y: {
            formatter: function(value) {
              return value.toFixed(2) + "% of total records";
            },
            title: {
              formatter: () => ""
            }
          },
          marker: {show: false}
        },
        xaxis: {
          labels: {
            formatter: (value) => value.toFixed(2)
          },
          title: {
            text: "Percentage of Hits"
          }
        },
        yaxis: {
          labels: {
            offsetY: 3.5
          }
        },
        legend: {show: false}
      }
    };
  },
  methods: {
    getStats(props) {
      // Construct a key
      const pubmed = props.row.source;
      const condition = props.row.condition;

      // Only select stats related to the key
      const stats = this.genomeCRISPRStats[pubmed].filter(
        e => e.condition === condition && e.num_hits > 0
      );

      return stats;
    },
    getSeries(props) {
      const stats = this.getStats(props);

      // this.chartOptions.xaxis['categories'] = 
      //   stats.map(e => e.cellline.toString());

      const series = [{
        data: stats.map(e => {
          return {
            x: e.cellline.toString(),
            y: e.percent_hits * 100
          }
        })
      }];

      return series;
    },
    getHeight(props) {
      const stats = this.getStats(props);

      const cellline = stats.map(e => e.cellline.toString());
      return 90 + cellline.length * 20;
    }
  }
};
</script>

<style scoped>
.margin-right {
  margin-right: 5px;
}
.margin-left {
  margin-left: 5px;
}
@media all and (min-width: 769px) {
  .margin-left-large {
    margin-left: 20px;
  }
}
@media all and (max-width: 769px) {
  .margin-bottom {
    margin-bottom: 5px;
  }
}
</style>
