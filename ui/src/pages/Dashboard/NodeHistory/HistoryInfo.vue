<template>
  <div class="row">
    <div class="col-md-12">
      <h2 class="title">
        Info
      </h2>
    </div>
    <p class="category" />
    <div>
      <loading
        :active.sync="isLoading"
        :can-cancel="false"
      />
    </div>
    <div class="col-12">
      <card>
        <div>
          <div class="col-12 d-flex justify-content-center justify-content-sm-between flex-wrap">
            <el-select
              v-model="pagination.perPage"
              class="select-default mb-3"
              style="width: 200px"
              placeholder="Per page"
            >
              <el-option
                v-for="item in pagination.perPageOptions"
                :key="item"
                class="select-default"
                :label="item"
                :value="item"
              />
            </el-select>
            <el-input
              v-model="searchQuery"
              type="search"
              class="mb-3"
              style="width: 200px"
              placeholder="Search Round Time"
              aria-controls="datatables"
            />
          </div>
          <div class="col-sm-12">
            <el-table
              stripe
              style="width: 100%;"
              :data="queriedData"
              border
            >
              <el-table-column
                v-for="column in tableColumns"
                :key="column.label"
                :min-width="column.minWidth"
                :prop="column.prop"
                :label="column.label"
                sortable
              />
            </el-table>
          </div>
        </div>
        <div
          slot="footer"
          class="col-12 d-flex justify-content-center justify-content-sm-between flex-wrap"
        >
          <div class="">
            <p class="card-category">
              Showing {{ from + 1 }} to {{ to }} of {{ total }} entries
            </p>
          </div>
          <l-pagination
            v-model="pagination.currentPage"
            class="pagination-no-border"
            :per-page="pagination.perPage"
            :total="pagination.total"
          />
        </div>
      </card>
    </div>
  </div>
</template>
<script>
import {
  Table, TableColumn, Select, Option,
} from 'element-ui';
import { Pagination as LPagination } from 'src/components/index';
import Fuse from 'fuse.js';
import axios from 'axios';
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';

export default {
  components: {
    LPagination,
    [Select.name]: Select,
    [Option.name]: Option,
    [Table.name]: Table,
    [TableColumn.name]: TableColumn,
    Loading,
  },
  data() {
    return {
      pagination: {
        perPage: 5,
        currentPage: 1,
        perPageOptions: [5, 10, 25, 50, 100, 200, 500, 1000, 2000, 5000, 10000],
        total: 0,
      },
      searchQuery: '',
      propsToSearch: ['roundTime'],
      tableColumns: [
        {
          prop: 'roundTime',
          label: 'Round Time',
          minWidth: 200,
        },
        {
          prop: 'roundTimeConverted',
          label: 'Round Time Converted',
          minWidth: 200,
        },
        {
          prop: 'cumulus',
          label: 'Cumulus',
          minWidth: 250,
        },
        {
          prop: 'nimbus',
          label: 'Nimbus',
          minWidth: 100,
        },
        {
          prop: 'stratus',
          label: 'Stratus',
          minWidth: 120,
        },
      ],
      tableData: [],
      fuseSearch: null,
      isLoading: false,
    };
  },
  computed: {
    pagedData() {
      return this.tableData.slice(this.from, this.to);
    },
    /** *
     * Searches through table data and returns a paginated array.
     * Note that this should not be used for table with a lot of data as it might be slow!
     * Do the search and the pagination on the server and display the data retrieved from server instead.
     * @returns {computed.pagedData}
     */
    queriedData() {
      let result;

      if (this.searchQuery !== '') {
        const temp = [];
        result = this.fuseSearch.search(`=${this.searchQuery}`);
        for (let i = 0; i < Object.keys(result).length; i += 1) {
          temp.push(result[i].item);
        }
        result = temp;
        this.paginationTotal(result.length);
      } else {
        this.paginationTotal(this.tableData.length);
        result = this.tableData;
      }

      return result.slice(this.from, this.to);
    },
    to() {
      let highBound = this.from + this.pagination.perPage;
      if (this.total < highBound) {
        highBound = this.total;
      }
      return highBound;
    },
    from() {
      return this.pagination.perPage * (this.pagination.currentPage - 1);
    },
    total() {
      this.paginationTotal(this.tableData.length);
      return this.tableData.length;
    },
  },
  mounted() {
    this.isLoading = true;
    axios
      .get('https://stats.runonflux.io/fluxhistorystats')
      .then((response) => {
        for (const [key, value] of Object.entries(response.data.data)) {
          this.tableData.push({
            roundTime: key,
            roundTimeConverted: `${new Date(parseInt(key, 10)).toLocaleDateString()} ${new Date(parseInt(key, 10)).toLocaleTimeString()}`,
            cumulus: value.cumulus,
            nimbus: value.nimbus,
            stratus: value.stratus,
          });
        }
        this.fuseSearch = new Fuse(this.tableData, { useExtendedSearch: true, keys: ['roundTime'] });
        this.isLoading = false;
      });
  },
  methods: {
    paginationTotal(value) {
      this.pagination.total = value;
    },
  },
};
</script>
<style>
</style>
