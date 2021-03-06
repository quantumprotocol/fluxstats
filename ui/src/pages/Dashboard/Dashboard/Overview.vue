<template>
  <div>
    <div>
      <loading
        :active.sync="isLoading"
        :can-cancel="false"
      />
    </div>

    <div
      v-if="!isFetching"
      class="row"
    >
      <div class="col-xl-3 col-md-6">
        <stats-card
          :title="totalNumberOfNodes.toString()"
          sub-title="Total Nodes"
        >
          <div
            slot="header"
            class="icon-success"
          >
            <i class="nc-icon nc-chart text-success" />
          </div>
          <template slot="footer">
            Cumulus + Nimbus + Stratus
          </template>
        </stats-card>
      </div>
      <div class="col-xl-3 col-md-6">
        <stats-card
          :title="totalNumberOfCumulus.toString()"
          sub-title="Cumulus"
        >
          <div
            slot="header"
            class="icon-info"
          >
            <i class="nc-icon nc-chart text-info" />
          </div>
          <template slot="footer">
            1,000 Flux
          </template>
        </stats-card>
      </div>
      <div class="col-xl-3 col-md-6">
        <stats-card
          :title="totalNumberOfNimbus.toString()"
          sub-title="Nimbus"
        >
          <div
            slot="header"
            class="icon-danger"
          >
            <i class="nc-icon nc-chart text-danger" />
          </div>
          <template slot="footer">
            12,500 Flux
          </template>
        </stats-card>
      </div>
      <div class="col-xl-3 col-md-6">
        <stats-card
          :title="totalNumberOfStratus.toString()"
          sub-title="Stratus"
        >
          <div
            slot="header"
            class="icon-warning"
          >
            <i class="nc-icon nc-chart text-warning" />
          </div>
          <template slot="footer">
            40,000 Flux
          </template>
        </stats-card>
      </div>
      <div class="col-xl-3 col-md-6">
        <stats-card
          :title="totalTBSSD.toString()"
          sub-title="TB SSD"
        >
          <div
            slot="header"
            class="icon-success"
          >
            <i class="nc-icon nc-chart-bar-32 text-success" />
          </div>
          <template slot="footer">
            Total Number of Available Storage
          </template>
        </stats-card>
      </div>
      <div class="col-xl-3 col-md-6">
        <stats-card
          :title="totalVCores.toString()"
          sub-title="VCORES"
        >
          <div
            slot="header"
            class="icon-info"
          >
            <i class="nc-icon nc-chart-bar-32 text-info" />
          </div>
          <template slot="footer">
            Total Number of Available Cores
          </template>
        </stats-card>
      </div>
      <div class="col-xl-3 col-md-6">
        <stats-card
          :title="totalTBRAM.toString()"
          sub-title="TB RAM"
        >
          <div
            slot="header"
            class="icon-danger"
          >
            <i class="nc-icon nc-chart-bar-32 text-danger" />
          </div>
          <template slot="footer">
            Total Number of Available RAM
          </template>
        </stats-card>
      </div>
    </div>

    <div
      v-if="!isFetching"
      class="row"
    >
      <div class="col-md-4">
        <chart-card
          :chart-data="pieChart.data"
          chart-type="Pie"
        >
          <template slot="header">
            <h4 class="card-title">
              Nodes Statistics
            </h4>
          </template>
          <template slot="footer">
            <div class="legend">
              <i class="fa fa-circle text-info" /> Cumulus
              <i class="fa fa-circle text-danger" /> Nimbus
              <i class="fa fa-circle text-warning" /> Stratus
            </div>
            <hr>
          </template>
        </chart-card>
      </div>
      <div class="col-md-8">
        <chart-card
          :chart-data="lineChart.data"
          :chart-options="lineChart.options"
          :responsive-options="lineChart.responsiveOptions"
        >
          <template slot="header">
            <h4 class="card-title">
              Nodes History
            </h4>
            <p class="card-category">
              Last 5 Round Time
            </p>
          </template>
          <template slot="footer">
            <div class="legend">
              <i class="fa fa-circle text-info" /> Cumulus
              <i class="fa fa-circle text-danger" /> Nimbus
              <i class="fa fa-circle text-warning" /> Stratus
            </div>
            <hr>
            <div class="stats" />
          </template>
        </chart-card>
      </div>
    </div>

    <div
      v-if="!isFetching"
      class="row"
    >
      <div class="col-md-6">
        <chart-card
          :chart-data="barChart.data"
          :chart-options="barChart.options"
          :chart-responsive-options="barChart.responsiveOptions"
          chart-type="Bar"
        >
          <template slot="header">
            <h4 class="card-title">
              Top 10 Node Location
            </h4>
            <p class="card-category">
              Countries With Highest Node Count
            </p>
          </template>
          <template slot="footer">
            <div class="legend">
              <i class="fa fa-circle text-info" /> Cumulus
              <i class="fa fa-circle text-danger" /> Nimbus
              <i class="fa fa-circle text-warning" /> Stratus
              <i class="fa fa-circle text-success" /> Total Nodes
            </div>
            <hr>
            <div class="stats" />
          </template>
        </chart-card>
      </div>
      <div class="col-md-6">
        <card
          class="card-tasks"
          title="Top 5 Organizations"
          sub-title="Organizations With Highest Node Count"
        >
          <l-table :data="tableData2.data">
            <template slot-scope="{row}">
              <td>{{ row.title }}</td>
              <td class="td-actions d-flex justify-content-end" />
            </template>
          </l-table>
          <div
            slot="footer"
            class="stats"
          />
        </card>
      </div>
    </div>
  </div>
</template>
<script>
import {
  ChartCard, StatsCard, Card, Table as LTable,
} from 'src/components/index';
import axios from 'axios';
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';

export default {
  components: {
    ChartCard,
    StatsCard,
    Loading,
    LTable,
    Card,
  },
  data() {
    return {
      pieChart: {
        data: {
          labels: [],
          series: [],
        },
      },
      lineChart: {
        data: {
          labels: [],
          series: [],
        },
        options: {
          low: 0,
          high: 10000,
          showArea: false,
          height: '245px',
          axisX: {
            showGrid: false,
          },
          lineSmooth: true,
          showLine: true,
          showPoint: true,
          fullWidth: true,
          chartPadding: {
            right: 50,
          },
        },
        responsiveOptions: [
          ['screen and (max-width: 640px)', {
            axisX: {
              labelInterpolationFnc(value) {
                return value[0];
              },
            },
          }],
        ],
      },
      barChart: {
        data: {
          labels: [],
          series: [],
        },
        options: {
          seriesBarDistance: 10,
          axisX: {
            showGrid: false,
          },
          height: '245px',
        },
        responsiveOptions: [
          ['screen and (max-width: 640px)', {
            seriesBarDistance: 5,
            axisX: {
              labelInterpolationFnc(value) {
                return value[0];
              },
            },
          }],
        ],
      },
      tableData: [],
      tableData1: [],
      tableData2: {
        data: [
        ],
      },
      totalNumberOfNodes: 0,
      totalNumberOfCumulus: 0,
      totalNumberOfNimbus: 0,
      totalNumberOfStratus: 0,
      totalTBSSD: 0,
      totalVCores: 0,
      totalTBRAM: 0,
      isLoading: true,
      statsLength: 0,
      isFetching: true,
    };
  },
  async created() {
    this.isLoading = true;
    const map = new Map();
    const mapCumulus = new Map();
    const mapNimbus = new Map();
    const mapStratus = new Map();
    const mapOrganizations = new Map();

    axios
      .get('https://stats.runonflux.io/fluxinfo?projection=ip,tier,geolocation,benchmark')
      .then((response) => {
        this.totalNumberOfNodes = Object.keys(response.data.data).length;
        this.totalNumberOfCumulus = 0;
        this.totalNumberOfNimbus = 0;
        this.totalNumberOfStratus = 0;
        this.tableData = response.data.data;
        this.tableData.forEach((data) => {
          if (data.tier === 'CUMULUS') {
            this.totalNumberOfCumulus += 1;
          } else if (data.tier === 'NIMBUS') {
            this.totalNumberOfNimbus += 1;
          } else if (data.tier === 'STRATUS') {
            this.totalNumberOfStratus += 1;
          }

          this.totalVCores += data.benchmark.bench.cores;
          this.totalTBSSD += data.benchmark.bench.ssd;
          this.totalTBRAM += data.benchmark.bench.ram;

          if (mapOrganizations.has(data.geolocation.org)) {
            mapOrganizations.set(data.geolocation.org, mapOrganizations.get(data.geolocation.org) + 1);
          } else {
            mapOrganizations.set(data.geolocation.org, 1);
          }

          if (map.has(data.geolocation.country)) {
            map.set(data.geolocation.country, map.get(data.geolocation.country) + 1);
          } else {
            map.set(data.geolocation.country, 1);
            mapCumulus.set(data.geolocation.country, 0);
            mapNimbus.set(data.geolocation.country, 0);
            mapStratus.set(data.geolocation.country, 0);
          }

          if (data.tier === 'CUMULUS') {
            mapCumulus.set(data.geolocation.country, mapCumulus.get(data.geolocation.country) + 1);
          } else if (data.tier === 'NIMBUS') {
            mapNimbus.set(data.geolocation.country, mapNimbus.get(data.geolocation.country) + 1);
          } else if (data.tier === 'STRATUS') {
            mapStratus.set(data.geolocation.country, mapStratus.get(data.geolocation.country) + 1);
          }
        });

        this.totalTBSSD = Number(this.totalTBSSD / 1000).toFixed(2);
        this.totalTBRAM = Number(this.totalTBRAM / 1000).toFixed(2);

        const pieChartPercentageCumulus = ((this.totalNumberOfCumulus / this.totalNumberOfNodes) * 100).toFixed(2);
        const pieChartPercentageNimbus = ((this.totalNumberOfNimbus / this.totalNumberOfNodes) * 100).toFixed(2);
        const pieChartPercentageStratus = ((this.totalNumberOfStratus / this.totalNumberOfNodes) * 100).toFixed(2);
        this.pieChart.data.labels = [`${pieChartPercentageCumulus} %`, `${pieChartPercentageNimbus} %`, `${pieChartPercentageStratus} %`];
        this.pieChart.data.series = [pieChartPercentageCumulus, pieChartPercentageNimbus, pieChartPercentageStratus];

        let idx = 0;
        let ent = [];
        for (const entry of new Map([...map.entries()].sort((a, b) => b[1] - a[1])).entries()) {
          const key = entry[0];
          const value = entry[1];

          ent.push({
            name: key,
            total: value,
          });

          if (idx < 9) {
            idx += 1;
          } else {
            break;
          }
        }

        this.barChart.data.labels = [ent[0].name, ent[1].name, ent[2].name, ent[3].name, ent[4].name,
          ent[5].name, ent[6].name, ent[7].name, ent[8].name, ent[9].name];

        this.barChart.data.series = [
          [mapCumulus.get(ent[0].name), mapCumulus.get(ent[1].name), mapCumulus.get(ent[2].name), mapCumulus.get(ent[3].name), mapCumulus.get(ent[4].name),
            mapCumulus.get(ent[5].name), mapCumulus.get(ent[6].name), mapCumulus.get(ent[7].name), mapCumulus.get(ent[8].name), mapCumulus.get(ent[9].name)],
          [mapNimbus.get(ent[0].name), mapNimbus.get(ent[1].name), mapNimbus.get(ent[2].name), mapNimbus.get(ent[3].name), mapNimbus.get(ent[4].name),
            mapNimbus.get(ent[5].name), mapNimbus.get(ent[6].name), mapNimbus.get(ent[7].name), mapNimbus.get(ent[8].name), mapNimbus.get(ent[9].name)],
          [mapStratus.get(ent[0].name), mapStratus.get(ent[1].name), mapStratus.get(ent[2].name), mapStratus.get(ent[3].name), mapStratus.get(ent[4].name),
            mapStratus.get(ent[5].name), mapStratus.get(ent[6].name), mapStratus.get(ent[7].name), mapStratus.get(ent[8].name), mapStratus.get(ent[9].name)],
          [ent[0].total, ent[1].total, ent[2].total, ent[3].total, ent[4].total, ent[5].total, ent[6].total, ent[7].total, ent[8].total, ent[9].total],
        ];

        idx = 0;
        ent = [];
        for (const entry of new Map([...mapOrganizations.entries()].sort((a, b) => b[1] - a[1])).entries()) {
          const key = entry[0];
          const value = entry[1];

          if (key !== '') {
            ent.push({
              name: key,
              total: value,
            });

            if (idx < 9) {
              idx += 1;
            } else {
              break;
            }
          }
        }

        this.tableData2.data.push({ title: `1. ${ent[0].name} - ${ent[0].total} Nodes` });
        this.tableData2.data.push({ title: `2. ${ent[1].name} - ${ent[1].total} Nodes` });
        this.tableData2.data.push({ title: `3. ${ent[2].name} - ${ent[2].total} Nodes` });
        this.tableData2.data.push({ title: `4. ${ent[3].name} - ${ent[3].total} Nodes` });
        this.tableData2.data.push({ title: `5. ${ent[4].name} - ${ent[4].total} Nodes` });
      }).then(() => {
        axios
          .get('https://stats.runonflux.io/fluxhistorystats')
          .then((response) => {
            for (const [key, value] of Object.entries(response.data.data)) {
              this.tableData1.push({
                roundTime: key,
                cumulus: value.cumulus,
                nimbus: value.nimbus,
                stratus: value.stratus,
              });
            }
            const statsLength = Object.keys(response.data.data).length;
            const item1 = new Date(parseInt(this.tableData1[statsLength - 1].roundTime, 10));
            const item2 = new Date(parseInt(this.tableData1[statsLength - 2].roundTime, 10));
            const item3 = new Date(parseInt(this.tableData1[statsLength - 3].roundTime, 10));
            const item4 = new Date(parseInt(this.tableData1[statsLength - 4].roundTime, 10));
            const item5 = new Date(parseInt(this.tableData1[statsLength - 5].roundTime, 10));

            this.lineChart.data.series = [
              [
                this.tableData1[statsLength - 1].cumulus,
                this.tableData1[statsLength - 2].cumulus,
                this.tableData1[statsLength - 3].cumulus,
                this.tableData1[statsLength - 4].cumulus,
                this.tableData1[statsLength - 5].cumulus,
                this.tableData1[statsLength - 6].cumulus,
              ],
              [
                this.tableData1[statsLength - 1].nimbus,
                this.tableData1[statsLength - 2].nimbus,
                this.tableData1[statsLength - 3].nimbus,
                this.tableData1[statsLength - 4].nimbus,
                this.tableData1[statsLength - 5].nimbus,
                this.tableData1[statsLength - 6].nimbus,
              ],
              [
                this.tableData1[statsLength - 1].stratus,
                this.tableData1[statsLength - 2].stratus,
                this.tableData1[statsLength - 3].stratus,
                this.tableData1[statsLength - 4].stratus,
                this.tableData1[statsLength - 5].stratus,
                this.tableData1[statsLength - 6].stratus,
              ],
            ];

            this.lineChart.data.labels = [
              `${item1.toLocaleDateString()} ${item1.toLocaleTimeString()}`,
              `${item2.toLocaleDateString()} ${item2.toLocaleTimeString()}`,
              `${item3.toLocaleDateString()} ${item3.toLocaleTimeString()}`,
              `${item4.toLocaleDateString()} ${item4.toLocaleTimeString()}`,
              `${item5.toLocaleDateString()} ${item5.toLocaleTimeString()}`,
            ];
          }).then(() => {
            this.isLoading = false;
            this.isFetching = false;
          });
      });
  },
};
</script>
<style>
</style>
