<template>
  <div class="show_all_content_wrapper">
    <!-- Start:: Main Section -->
    <main>
      <!--  =========== Start:: Filter Form =========== -->
      <div
        class="filter_content_wrapper"
        :class="{ active: filterFormIsActive }"
      >
        <button
          class="filter_toggler"
          @click="filterFormIsActive = !filterFormIsActive"
        >
          <i class="fal fa-times"></i>
        </button>
        <div class="filter_title_wrapper">
          <h5>{{ $t("TITLES.searchBy") }}</h5>
        </div>
        <div class="filter_form_wrapper">
          <form @submit.prevent="submitFilterForm">
            <div class="row justify-content-center align-items-center w-100">
              <!-- Start:: Delivery Agent Name -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.driverName')"
                v-model="filterOptions.driverName"
              />
              <!-- End:: Delivery Agent Name -->

              <!-- Start:: Mobile Number -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.mobileNumber')"
                v-model="filterOptions.mobile"
              />
              <!-- End:: Mobile Number -->

              <!-- Start:: Email Address -->
              <base-input
                col="4"
                type="email"
                :placeholder="$t('PLACEHOLDERS.email')"
                v-model="filterOptions.email"
              />
              <!-- End:: Email Address -->

              <!-- Start:: Time Period Filter -->
              <base-select-input
                col="4"
                :optionsList="timePeriods"
                :placeholder="$t('PLACEHOLDERS.timePeriod')"
                v-model="filterOptions.timePeriod"
              />
              <!-- End:: Time Period Filter -->

              <!-- Start:: Start Date Input (Custom Range) -->
              <base-picker-input
                col="4"
                type="date"
                :placeholder="$t('PLACEHOLDERS.dateFrom')"
                v-model.trim="filterOptions.dateFrom"
                :disabled="
                  filterOptions.timePeriod &&
                  filterOptions.timePeriod.value !== 'custom'
                "
              />
              <!-- End:: Start Date Input -->

              <!-- Start:: End Date Input (Custom Range) -->
              <base-picker-input
                col="4"
                type="date"
                :placeholder="$t('PLACEHOLDERS.dateTo')"
                v-model.trim="filterOptions.dateTo"
                :disabled="
                  filterOptions.timePeriod &&
                  filterOptions.timePeriod.value !== 'custom'
                "
              />
              <!-- End:: End Date Input -->
            </div>

            <div class="btns_wrapper">
              <button class="submit_btn" :disabled="isWaitingRequest">
                <i class="fal fa-search"></i>
              </button>
              <button
                class="reset_btn"
                type="button"
                :disabled="isWaitingRequest"
                @click="resetFilter"
              >
                <i class="fal fa-redo"></i>
              </button>
            </div>
          </form>
        </div>
      </div>
      <!--  =========== End:: Filter Form =========== -->

      <!--  =========== Start:: Table Title =========== -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>
            {{ $t("PLACEHOLDERS.deliveryAgentFinancialReports") }}
          </h5>
          <button
            v-if="!filterFormIsActive"
            class="filter_toggler"
            @click.stop="filterFormIsActive = !filterFormIsActive"
          >
            <i class="fal fa-search"></i>
          </button>
        </div>

        <div class="title_route_wrapper">
          <base-button
            class="mt-0 pdf_btn"
            styleType="solid_btn"
            :btnText="$t('BUTTONS.downloadPdf')"
            @fireClick="downloadPdf"
          >
            <template v-slot:btn_icon>
              <i class="fal fa-file-pdf"></i>
            </template>
          </base-button>
          <base-button
            class="mt-0 excel_btn"
            styleType="solid_btn"
            :btnText="$t('BUTTONS.downloadExcel')"
            @fireClick="downloadExcelAllData"
            :disabled="excelDownloadBtnIsLoading"
          >
            <template v-slot:btn_icon>
              <i class="fal fa-file-excel"></i>
            </template>
          </base-button>
        </div>
      </div>
      <!--  =========== End:: Table Title =========== -->

      <!--  =========== Start:: Data Table =========== -->
      <v-data-table
        class="thumb"
        :loading="loading"
        :loading-text="$t('TABLES.loadingData')"
        :search="searchValue"
        :headers="tableHeaders"
        :items="tableRows"
        item-class="ltr"
        :items-per-page="paginations.items_per_page"
        hide-default-footer
      >
        <template v-slot:[`item.serial_number`]="{ index }">
          {{
            (paginations.current_page - 1) * paginations.items_per_page +
            index +
            1
          }}
        </template>
        <!-- Start:: No Data State -->
        <template v-slot:no-data>
          {{ $t("TABLES.noData") }}
        </template>
        <!-- End:: No Data State -->

        <template v-slot:[`item.serial`]="{ item, index }">
          <div class="table_image_wrapper">
            <h6 class="text-danger" v-if="!item.id">
              {{ $t("TABLES.noData") }}
            </h6>
            <p v-else>
              {{
                (paginations.current_page - 1) * paginations.items_per_page +
                index +
                1
              }}
            </p>
          </div>
        </template>

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("BUTTONS.show") }}</span>
              </template>
              <button class="btn_show" @click="showItem(item)">
                <i class="fal fa-eye"></i>
              </button>
            </a-tooltip>
          </div>
        </template>
        <!-- End:: Actions -->
      </v-data-table>
      <!--  =========== End:: Data Table =========== -->

      <!-- ==== Start:: Overall_statistics Addresses ==== -->
      <div class="table_content mt-5">
        <v-simple-table class="stat-table">
          <template v-slot:default>
            <thead class="all-stat">
              <tr>
                <th class="text-center" style="color: white !important">
                  {{ $t("PLACEHOLDERS.totalCompletedOrders") }}
                </th>
                <th class="text-center" style="color: white !important">
                  {{ $t("PLACEHOLDERS.totalEarnings") }}
                </th>
                <th class="text-center" style="color: white !important">
                  {{ $t("PLACEHOLDERS.totalAdminFee") }}
                </th>
              </tr>
            </thead>
            <tbody>
              <tr class="text-center">
                <td>{{ totals.total_complete_orders_count }}</td>
                <td>{{ totals.total_earned }}</td>
                <td>{{ totals.total_commission }}</td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
      </div>
      <!-- ==== End:: Overall_statistics Addresses ==== -->
    </main>
    <!-- End:: Main Section -->

    <!-- Start:: Pagination -->
    <template>
      <div class="pagination_container text-center mt-3 mb-0">
        <v-pagination
          class="py-0"
          square
          v-model="paginations.current_page"
          :length="paginations.last_page"
          :total-visible="6"
          @input="updateRouterQueryParam($event)"
          :prev-icon="
            getAppLocale == 'ar' ? 'fal fa-angle-right' : 'fal fa-angle-left'
          "
          :next-icon="
            getAppLocale == 'ar' ? 'fal fa-angle-left' : 'fal fa-angle-right'
          "
        />
      </div>
    </template>
    <!-- End:: Pagination -->

    <!-- Start:: Generate PDF Template Content -->
    <vue-html2pdf
      :show-layout="false"
      :float-layout="true"
      :enable-download="true"
      :preview-modal="true"
      :filename="$t('PLACEHOLDERS.deliveryAgentFinancialReports')"
      :pdf-quality="2"
      pdf-format="a4"
      :manual-pagination="false"
      :paginate-elements-by-height="1400"
      pdf-content-width="100%"
      @hasGenerated="$message.success($t('MESSAGES.generatedSuccessfully'))"
      ref="html2Pdf"
    >
      <section slot="pdf-content">
        <div class="pdf_file_content">
          <h3 class="file_title">
            {{ $t("PLACEHOLDERS.deliveryAgentFinancialReports") }}
          </h3>
          <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th
                    v-for="(header, index) in tableHeaders"
                    :key="header.value"
                  >
                    <span>{{ header.text }}</span>
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, index) in tableRows" :key="row.id">
                  <td>
                    {{
                      (paginations.current_page - 1) *
                        paginations.items_per_page +
                      index +
                      1
                    }}
                  </td>
                  <td>{{ row.name }}</td>
                  <td>{{ row.mobile }}</td>
                  <td>{{ row.email }}</td>
                  <td>{{ row.completed_orders_count }}</td>
                  <td>{{ row.total_earned }}</td>
                  <td>{{ row.total_commission }}</td>
                  <td>{{ row.current_user_balance }}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
          <!-- ==== Start:: Overall_statistics Addresses ==== -->
          <div class="table_content mt-5">
            <v-simple-table class="stat-table">
              <template v-slot:default>
                <thead class="all-stat">
                  <tr>
                    <th class="text-center" style="color: white !important">
                      {{ $t("PLACEHOLDERS.totalCompletedOrders") }}
                    </th>
                    <th class="text-center" style="color: white !important">
                      {{ $t("PLACEHOLDERS.totalEarnings") }}
                    </th>
                    <th class="text-center" style="color: white !important">
                      {{ $t("PLACEHOLDERS.totalAdminFee") }}
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr class="text-center">
                    <td>{{ totals.total_complete_orders_count }}</td>
                    <td>{{ totals.total_earned }}</td>
                    <td>{{ totals.total_commission }}</td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
          </div>
          <!-- ==== End:: Overall_statistics Addresses ==== -->
        </div>
      </section>
    </vue-html2pdf>
    <!-- End:: Generate PDF Template Content -->
  </div>
</template>

<script>
import VueHtml2pdf from "vue-html2pdf";
import { mapGetters } from "vuex";
import * as XLSX from "xlsx";

export default {
  name: "AllFinancialReports",

  components: {
    VueHtml2pdf,
  },

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),
  },

  data() {
    return {
      loading: false,
      isWaitingRequest: false,
      excelDownloadBtnIsLoading: false,

      filterFormIsActive: false,
      filterOptions: {
        driverName: null,
        mobile: null,
        email: null,
        timePeriod: null,
        dateFrom: null,
        dateTo: null,
      },
      timePeriods: [
        { id: 1, name: this.$t("BUTTONS.daily"), value: "daily" },
        { id: 2, name: this.$t("BUTTONS.weekly"), value: "weekly" },
        { id: 3, name: this.$t("BUTTONS.monthly"), value: "monthly" },
        {
          id: 4,
          name: this.$t("PLACEHOLDERS.customDateRange"),
          value: "custom",
        },
      ],
      tableHeaders: [
        {
          text: this.$t("TABLES.Admins.serialNumber"),
          value: "serial_number",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.driverName"),
          value: "name",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.mobileNumber"),
          value: "mobile",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.email"),
          value: "email",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.totalCompletedOrders"),
          value: "completed_orders_count",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.totalEarnings"),
          value: "total_earned",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.totalAdminFee"),
          value: "total_commission",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.currentWalletBalance"),
          value: "current_user_balance",
          align: "center",
          sortable: false,
        },
      ],
      tableRows: [],
      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
      totals: {
        total_complete_orders_count: 0,
        total_earned: 0,
        total_commission: 0,
      },
    };
  },

  methods: {
    async submitFilterForm() {
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.driverName = null;
      this.filterOptions.mobile = null;
      this.filterOptions.email = null;
      this.filterOptions.timePeriod = null;
      this.filterOptions.dateFrom = null;
      this.filterOptions.dateTo = null;
      this.setTableRows();
    },
    async setTableRows() {
      this.loading = true;
      try {
        let res = await this.$axios({
          method: "GET",
          url: "financial-reports",
          params: {
            page: this.paginations.current_page,
            search: this.filterOptions.driverName,
            mobile: this.filterOptions.mobile,
            email: this.filterOptions.email,
            type: this.filterOptions.timePeriod?.value,
            from: this.filterOptions.dateFrom,
            to: this.filterOptions.dateTo,
          },
        });
        this.loading = false;
        this.tableRows = res.data.data?.data || [];
        this.totals.total_complete_orders_count =
          res.data.data?.total?.total_complete_orders_count || 0;
        this.totals.total_earned = res.data.data?.total?.total_earned || 0;
        this.totals.total_commission =
          res.data.data?.total?.total_commission || 0;
        this.paginations.last_page = res.data.data?.links?.last_page || 1;
        this.paginations.items_per_page = res.data.data?.links?.per_page || 15;
      } catch (error) {
        this.loading = false;
        console.log(error.response?.data?.message || error.message);
      }
    },
    async downloadPdf() {
      await this.$refs.html2Pdf.generatePdf();
    },
    async downloadExcelAllData() {
      this.loading = true;
      try {
        let res = await this.$axios({
          method: "GET",
          url: "financial-reports",
          params: {
            search: this.filterOptions.driverName,
            mobile: this.filterOptions.mobile,
            email: this.filterOptions.email,
            type: this.filterOptions.timePeriod?.value,
            from: this.filterOptions.dateFrom,
            to: this.filterOptions.dateTo,
          },
        });
        const allData = res.data.data?.data || [];

        // Map data with translated headers
        const translatedData = allData.map((row, index) => {
          if (this.getAppLocale == "ar") {
            return {
              [this.$t("PLACEHOLDERS.currentWalletBalance")]:
                row.current_user_balance,
              [this.$t("PLACEHOLDERS.totalAdminFee")]: row.total_commission,
              [this.$t("PLACEHOLDERS.totalEarnings")]: row.total_earned,
              [this.$t("PLACEHOLDERS.totalCompletedOrders")]:
                row.completed_orders_count,
              [this.$t("PLACEHOLDERS.email")]: row.email,
              [this.$t("PLACEHOLDERS.mobileNumber")]: row.mobile,
              [this.$t("PLACEHOLDERS.driverName")]: row.name,
              [this.$t("TABLES.Admins.serialNumber")]: index + 1,
            };
          } else {
            return {
              [this.$t("TABLES.Admins.serialNumber")]: index + 1,
              [this.$t("PLACEHOLDERS.driverName")]: row.name,
              [this.$t("PLACEHOLDERS.mobileNumber")]: row.mobile,
              [this.$t("PLACEHOLDERS.email")]: row.email,
              [this.$t("PLACEHOLDERS.totalCompletedOrders")]:
                row.completed_orders_count,
              [this.$t("PLACEHOLDERS.totalEarnings")]: row.total_earned,
              [this.$t("PLACEHOLDERS.totalAdminFee")]: row.total_commission,
              [this.$t("PLACEHOLDERS.currentWalletBalance")]:
                row.current_user_balance,
            };
          }
        });

        translatedData.push({});
        if (this.getAppLocale === "ar") {
          // Append overall statistics at the bottom
          translatedData.push({
            [this.$t("PLACEHOLDERS.driverName")]: this.$t(
              "PLACEHOLDERS.totalCompletedOrders"
            ),
            [this.$t("PLACEHOLDERS.mobileNumber")]:
              this.totals.total_complete_orders_count,
          });

          translatedData.push({
            [this.$t("PLACEHOLDERS.driverName")]: this.$t(
              "PLACEHOLDERS.totalEarnings"
            ),
            [this.$t("PLACEHOLDERS.mobileNumber")]: this.totals.total_earned,
          });

          translatedData.push({
            [this.$t("PLACEHOLDERS.driverName")]: this.$t(
              "PLACEHOLDERS.totalAdminFee"
            ),
            [this.$t("PLACEHOLDERS.mobileNumber")]:
              this.totals.total_commission,
          });
        } else {
          // Append overall statistics at the bottom
          translatedData.push({
            [this.$t("TABLES.Admins.serialNumber")]: this.$t(
              "PLACEHOLDERS.totalCompletedOrders"
            ),
            [this.$t("PLACEHOLDERS.driverName")]:
              this.totals.total_complete_orders_count,
          });

          translatedData.push({
            [this.$t("TABLES.Admins.serialNumber")]: this.$t(
              "PLACEHOLDERS.totalEarnings"
            ),
            [this.$t("PLACEHOLDERS.driverName")]: this.totals.total_earned,
          });

          translatedData.push({
            [this.$t("TABLES.Admins.serialNumber")]: this.$t(
              "PLACEHOLDERS.totalAdminFee"
            ),
            [this.$t("PLACEHOLDERS.driverName")]: this.totals.total_commission,
          });
        }

        const worksheet = XLSX.utils.json_to_sheet(translatedData);
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "Financial Reports");

        // Generate Excel file
        const excelBuffer = XLSX.write(workbook, {
          bookType: "xlsx",
          type: "array",
        });
        const data = new Blob([excelBuffer], {
          type: "application/octet-stream",
        });
        const url = window.URL.createObjectURL(data);
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute(
          "download",
          `${this.$t("PLACEHOLDERS.deliveryAgentFinancialReports")}.xlsx`
        );
        document.body.appendChild(link);
        link.click();
        link.remove();
      } catch (error) {
        console.log(error.response?.data?.message || error.message);
      } finally {
        this.loading = false;
      }
    },

    // Navigate to driver details
    showItem(item) {
      this.$router.push({ path: `/drivers/show/${item.id}` });
    },

    updateRouterQueryParam(pagenationValue) {
      this.$router.push({
        query: {
          ...this.$route.query,
          page: pagenationValue,
        },
      });

      // Scroll To Screen's Top After Get Data
      document.body.scrollTop = 0; // For Safari
      document.documentElement.scrollTop = 0; // For Chrome, Firefox, IE and Opera
    },
  },

  created() {
    this.setTableRows();
  },
};
</script>

<style scoped>
.ex-btn-s {
  color: black !important;
  font-size: 20px;
  font-weight: bold;
  text-decoration: underline !important;
}

.all-stat {
  background: var(--main_theme_clr);
}

.all-stat th {
  color: white !important;
  font-size: 16px !important;
  font-weight: 800;
}

.excel {
  background: darkgreen;
  padding: 7px 10px;
  border-radius: 12px;
  color: #fff;
  cursor: pointer;
}
</style>
