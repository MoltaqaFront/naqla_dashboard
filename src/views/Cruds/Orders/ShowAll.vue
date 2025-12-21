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
              <!-- Start:: Order Number Input -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.orderNum')"
                v-model.trim="filterOptions.orderNum"
              />
              <!-- End:: Order Number Input -->

              <!-- Start:: Customer Name Input -->
              <!-- <base-select-input
                col="4"
                :optionsList="users"
                :placeholder="$t('PLACEHOLDERS.clientName')"
                v-model="filterOptions.clientName"
              /> -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.clientName')"
                v-model.trim="filterOptions.clientName"
              />
              <!-- End:: Customer Name Input -->

              <!-- Start:: Driver Name Input -->
              <!-- <base-select-input
                col="4"
                :optionsList="drivers"
                :placeholder="$t('PLACEHOLDERS.driverName')"
                v-model="filterOptions.driverName"
              /> -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.driverName')"
                v-model.trim="filterOptions.driverName"
              />
              <!-- End:: Driver Name Input -->

              <!-- Start:: Status Input -->
              <base-select-input
                col="4"
                :optionsList="orderStatuses"
                :placeholder="$t('PLACEHOLDERS.orderStatus')"
                v-model="filterOptions.status"
              />
              <!-- End:: Status Input -->

              <!-- Start:: Date From Input -->
              <base-picker-input
                col="4"
                type="date"
                :placeholder="$t('PLACEHOLDERS.dateFrom')"
                v-model.trim="filterOptions.dateFrom"
              />
              <!-- End:: Date From Input -->

              <!-- Start:: Date To Input -->
              <base-picker-input
                col="4"
                type="date"
                :placeholder="$t('PLACEHOLDERS.dateTo')"
                v-model.trim="filterOptions.dateTo"
              />
              <!-- End:: Date To Input -->
            </div>

            <div class="btns_wrapper">
              <a-tooltip placement="bottom">
                <template slot="title">
                  <span>{{ $t("BUTTONS.search") }}</span>
                </template>
                <span
                  class="submit_btn"
                  @click="submitFilterForm"
                  :disabled="isWaitingRequest"
                >
                  <i class="fal fa-search"></i>
                </span>
              </a-tooltip>

              <a-tooltip placement="bottom">
                <template slot="title">
                  <span>{{ $t("BUTTONS.rseet_search") }}</span>
                </template>
                <span
                  class="reset_btn"
                  :disabled="isWaitingRequest"
                  @click="resetFilter"
                >
                  <i class="fal fa-redo"></i>
                </span>
              </a-tooltip>
            </div>
          </form>
        </div>
      </div>
      <!--  =========== End:: Filter Form =========== -->

      <!--  =========== Start:: Table Title =========== -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("PLACEHOLDERS.orders") }}</h5>
          <button
            v-if="!filterFormIsActive"
            class="filter_toggler"
            @click.stop="filterFormIsActive = !filterFormIsActive"
          >
            <i class="fal fa-search"></i>
          </button>
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
        <!-- Start:: No Data State -->
        <template v-slot:no-data>
          {{ $t("TABLES.noData") }}
        </template>
        <!-- Start:: No Data State -->

        <template v-slot:[`item.id`]="{ item, index }">
          <p class="blue-grey--text text--darken-1 fs-3" v-if="!item.id">-</p>
          <p v-else>
            {{
              (paginations.current_page - 1) * paginations.items_per_page +
              index +
              1
            }}
          </p>
        </template>

        <!-- <template v-slot:[`item.id`]="{ item }">
          <h6 class="text-danger" v-if="!item.id">
            {{ $t("TABLES.noData") }}
          </h6>
          <h6 v-else>{{ item.id }}</h6>
        </template> -->

        <!-- <template v-slot:[`item.serialNumber`]="{ item }">
          <p class="blue-grey--text text--darken-1 fs-3" v-if="!item.serialNumber">-</p>
          <p v-else>{{ item.serialNumber }}</p>
        </template> -->

        <!-- Start:: Activation Status -->
        <template v-slot:[`item.status`]="{ item }">
          <v-chip :color="getStatusColor(item.status)">
            {{ getStatusTranslation(item.status) }}
          </v-chip>
        </template>
        <!-- End:: Activation Status -->

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <!-- Show Action -->
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("BUTTONS.show") }}</span>
              </template>
              <button class="btn_show" @click="showItem(item)">
                <i class="fal fa-eye"></i>
              </button>
            </a-tooltip>

            <!-- Change Status Action -->
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("BUTTONS.changeStatus") }}</span>
              </template>
              <button class="btn_edit" @click="selectChangeStatusItem(item)">
                <i class="fal fa-sync-alt"></i>
              </button>
            </a-tooltip>

            <!-- Update Driver Action -->
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("BUTTONS.updateDriver") }}</span>
              </template>
              <button class="" @click="selectUpdateDriverItem(item)">
                <i class="fal fa-user-edit"></i>
              </button>
            </a-tooltip>

            <!-- Delete Action -->
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("BUTTONS.delete") }}</span>
              </template>
              <button class="btn_delete" @click="selectDeleteItem(item)">
                <i class="fal fa-trash-alt"></i>
              </button>
            </a-tooltip>
          </div>
        </template>
        <!-- End:: Actions -->

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>
          <!-- Start:: Change Status Modal -->
          <v-dialog v-model="dialogChangeStatus" max-width="500px">
            <v-card>
              <v-card-title class="text-h5 justify-center">
                {{ $t("PLACEHOLDERS.changeOrderStatus") }}
              </v-card-title>

              <form class="w-100 px-4">
                <base-select-input
                  col="12"
                  :optionsList="orderStatuses"
                  :placeholder="$t('PLACEHOLDERS.orderStatus')"
                  v-model="selectedStatus"
                  required
                />

                <!-- Show cancel reason if status is canceled -->
                <base-input
                  v-if="selectedStatus && selectedStatus.value === 'canceled'"
                  col="12"
                  rows="3"
                  type="textarea"
                  :placeholder="$t('PLACEHOLDERS.cancelReason')"
                  v-model.trim="cancelReason"
                  required
                />
              </form>

              <v-card-actions>
                <v-btn
                  class="modal_confirm_btn"
                  @click="confirmChangeStatus"
                  :disabled="isWaitingRequest"
                >
                  {{ $t("BUTTONS.ok") }}
                </v-btn>

                <v-btn
                  class="modal_cancel_btn"
                  @click="dialogChangeStatus = false"
                >
                  {{ $t("BUTTONS.cancel") }}
                </v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <!-- End:: Change Status Modal -->

          <!-- Start:: Update Driver Modal -->
          <v-dialog v-model="dialogUpdateDriver" max-width="500px">
            <v-card>
              <v-card-title class="text-h5 justify-center">
                {{ $t("PLACEHOLDERS.updateDriver") }}
              </v-card-title>

              <form class="w-100 px-4">
                <base-select-input
                  col="12"
                  :optionsList="drivers"
                  :placeholder="$t('PLACEHOLDERS.driverName')"
                  v-model="selectedDriver"
                  required
                />
              </form>

              <v-card-actions>
                <v-btn
                  class="modal_confirm_btn"
                  @click="confirmUpdateDriver"
                  :disabled="isWaitingRequest"
                >
                  {{ $t("BUTTONS.ok") }}
                </v-btn>

                <v-btn
                  class="modal_cancel_btn"
                  @click="dialogUpdateDriver = false"
                >
                  {{ $t("BUTTONS.cancel") }}
                </v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <!-- End:: Update Driver Modal -->

          <!-- Start:: Delete Modal -->
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5 justify-center" v-if="itemToDelete">
                {{
                  $t("TITLES.DeleteConfirmingMessage", {
                    name:
                      itemToDelete.order_number || itemToDelete.number_order,
                  })
                }}
              </v-card-title>

              <v-card-actions>
                <v-btn
                  class="modal_confirm_btn"
                  @click="confirmDeleteItem"
                  :disabled="isWaitingRequest"
                >
                  {{ $t("BUTTONS.ok") }}
                </v-btn>

                <v-btn class="modal_cancel_btn" @click="dialogDelete = false">
                  {{ $t("BUTTONS.cancel") }}
                </v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <!-- End:: Delete Modal -->
        </template>
        <!-- ======================== End:: Dialogs ======================== -->
      </v-data-table>
      <!--  =========== End:: Data Table =========== -->
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
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

export default {
  name: "AllRegions",

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),
    orderStatuses() {
      return [
        {
          id: 1,
          name: this.$t("STATUS.new"),
          value: "new",
        },
        {
          id: 2,
          name: this.$t("STATUS.accepted"),
          value: "accepted",
        },
        {
          id: 3,
          name: this.$t("STATUS.picked_up"),
          value: "received",
        },
        {
          id: 4,
          name: this.$t("STATUS.on_the_way"),
          value: "on_the_way",
        },
        {
          id: 5,
          name: this.$t("STATUS.delivered"),
          value: "delivered",
        },
        {
          id: 6,
          name: this.$t("STATUS.canceled"),
          value: "canceled",
        },
      ];
    },
  },

  data() {
    return {
      // Start:: Loading Data
      loading: false,
      isWaitingRequest: false,
      // End:: Loading Data

      // Start:: Filter Data
      filterFormIsActive: false,
      filterOptions: {
        orderNum: null,
        clientName: null,
        driverName: null,
        status: null,
        dateFrom: null,
        dateTo: null,
      },
      // End:: Filter Data

      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.Clients.serialNumber"),
          value: "id",
          align: "center",
          width: "80",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.orderNum"),
          value: "order_number",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.clientName"),
          value: "user.name",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.clientPhone"),
          value: "user.mobile",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.driverName"),
          value: "driver.name",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.orderValue"),
          value: "price",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.orderStatus"),
          value: "status",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.orderDate"),
          value: "created_at",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Clients.actions"),
          value: "actions",
          sortable: false,
          align: "center",
        },
      ],
      tableRows: [],
      today: new Date(),
      // End:: Table Data

      // Start:: Pagination Data
      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
      // End:: Pagination Data

      // Start:: Dialogs Control Data
      dialogChangeStatus: false,
      itemToChangeStatus: null,
      selectedStatus: null,
      cancelReason: null,
      dialogUpdateDriver: false,
      itemToUpdateDriver: null,
      selectedDriver: null,
      dialogDelete: false,
      itemToDelete: null,
      // End:: Dialogs Control Data

      // Start:: Users & Drivers Data
      users: [],
      drivers: [],
      // End:: Users & Drivers Data
    };
  },

  watch: {
    // Start:: Page Query Param Watcher To Get Page Data Based On It's Change
    ["$route.query.page"]() {
      this.paginations.current_page = +this.$route.query.page;
      this.setTableRows();
    },
    // End:: Page Query Param Watcher To Get Page Data Based On It's Change
  },

  methods: {
    // Start:: Get Status Translation
    getStatusTranslation(status) {
      const statusMap = {
        new: this.$t("STATUS.new"),
        accepted: this.$t("STATUS.accepted"),
        picked_up: this.$t("STATUS.picked_up"),
        on_the_way: this.$t("STATUS.on_the_way"),
        delivered: this.$t("STATUS.delivered"),
        canceled: this.$t("STATUS.canceled"),
      };
      return statusMap[status] || status;
    },
    // End:: Get Status Translation

    // Start:: Get Status Color
    getStatusColor(status) {
      const colorMap = {
        new: "blue",
        accepted: "cyan",
        picked_up: "orange",
        on_the_way: "purple",
        delivered: "green",
        canceled: "red",
      };
      return colorMap[status] || "grey";
    },
    // End:: Get Status Color

    // Start:: Get Users (Clients)
    async getUsers() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: "clients",
          params: {
            page: 0,
            limit: 0,
          },
        });
        this.users = res.data.data.data.map((user) => ({
          id: user.id,
          name: user.name,
        }));
      } catch (error) {
        console.log(error.response.data.message);
      }
    },
    // End:: Get Users (Clients)

    // Start:: Get Drivers
    async getDrivers() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: "drivers",
          params: {
            page: 0,
            limit: 0,
          },
        });
        this.drivers = res.data.data.data.map((driver) => ({
          id: driver.id,
          name: driver.name,
        }));
      } catch (error) {
        console.log(error.response.data.message);
      }
    },
    // End:: Get Drivers

    // Start:: Handel Filter
    async submitFilterForm() {
      if (this.$route.query.page !== "1") {
        await this.$router.push({
          path: "/orders/all",
          query: { page: 1 },
        });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.orderNum = null;
      this.filterOptions.clientName = null;
      this.filterOptions.driverName = null;
      this.filterOptions.status = null;
      this.filterOptions.dateFrom = null;
      this.filterOptions.dateTo = null;
      if (this.$route.query.page !== "1") {
        await this.$router.push({
          path: "/orders/all",
          query: { page: 1 },
        });
      }
      this.setTableRows();
    },
    // End:: Handel Filter

    // Start:: Set Table Rows
    updateRouterQueryParam(pagenationValue) {
      this.$router.push({
        query: {
          ...this.$route.query,
          page: pagenationValue,
        },
      });

      // Scroll To Screen's Top After Get Products
      document.body.scrollTop = 0; // For Safari
      document.documentElement.scrollTop = 0; // For Chrome, Firefox, IE and Opera
    },
    async setTableRows() {
      this.loading = true;
      try {
        let res = await this.$axios({
          method: "GET",
          url: "orders",
          params: {
            page: this.paginations.current_page,
            serialNumber: this.filterOptions.orderNum,
            clientName: this.filterOptions.clientName,
            driverName: this.filterOptions.driverName,
            status: this.filterOptions.status?.value,
            from: this.filterOptions.dateFrom,
            to: this.filterOptions.dateTo,
          },
        });
        this.loading = false;
        this.tableRows = res.data.data.data;
        this.paginations.last_page = res.data.data.meta.last_page;
        this.paginations.items_per_page = res.data.data.meta.per_page;
      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // End:: Set Table Rows

    // Start:: Show Item
    showItem(item) {
      this.$router.push({ path: `/orders/show/${item.id}` });
    },
    // End:: Show Item

    // Start:: Change Status
    selectChangeStatusItem(item) {
      this.dialogChangeStatus = true;
      this.itemToChangeStatus = item;
      this.selectedStatus = null;
      this.cancelReason = null;
    },
    async confirmChangeStatus() {
      if (!this.selectedStatus) {
        this.$message.error(this.$t("VALIDATION.selectStatus"));
        return;
      }

      if (this.selectedStatus.value === "canceled" && !this.cancelReason) {
        this.$message.error(this.$t("VALIDATION.cancelReason"));
        return;
      }

      this.isWaitingRequest = true;
      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("status", this.selectedStatus.value);
      if (this.selectedStatus.value === "canceled") {
        REQUEST_DATA.append("cancel_reason", this.cancelReason);
      }

      try {
        await this.$axios({
          method: "POST",
          url: `orders/${this.itemToChangeStatus.id}/change-status`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.dialogChangeStatus = false;
        this.itemToChangeStatus = null;
        this.selectedStatus = null;
        this.cancelReason = null;
        this.$message.success(this.$t("MESSAGES.statusChanged"));
        this.setTableRows();
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
    // End:: Change Status

    // Start:: Update Driver
    selectUpdateDriverItem(item) {
      this.dialogUpdateDriver = true;
      this.itemToUpdateDriver = item;
      this.selectedDriver = null;
    },
    async confirmUpdateDriver() {
      if (!this.selectedDriver) {
        this.$message.error(this.$t("VALIDATION.selectDriver"));
        return;
      }

      this.isWaitingRequest = true;
      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("driver_id", this.selectedDriver.id);

      try {
        await this.$axios({
          method: "POST",
          url: `orders/${this.itemToUpdateDriver.id}/update-driver`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.dialogUpdateDriver = false;
        this.itemToUpdateDriver = null;
        this.selectedDriver = null;
        this.$message.success(this.$t("MESSAGES.driverUpdated"));
        this.setTableRows();
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
    // End:: Update Driver

    // Start:: Delete Item
    selectDeleteItem(item) {
      this.dialogDelete = true;
      this.itemToDelete = item;
    },
    async confirmDeleteItem() {
      this.isWaitingRequest = true;
      try {
        await this.$axios({
          method: "DELETE",
          url: `orders/${this.itemToDelete.id}`,
        });
        this.isWaitingRequest = false;
        this.dialogDelete = false;
        this.$message.success(this.$t("MESSAGES.deletedSuccessfully"));
        this.setTableRows();
        this.itemToDelete = null;
      } catch (error) {
        this.isWaitingRequest = false;
        this.dialogDelete = false;
        this.$message.error(error.response.data.message);
      }
    },
    // End:: Delete Item
  },

  created() {
    // Start:: Fire Methods
    window.addEventListener("click", () => {
      this.filterFormIsActive = false;
    });
    if (this.$route.query.page) {
      this.paginations.current_page = +this.$route.query.page;
    }
    this.setTableRows();
    this.getUsers();
    this.getDrivers();
    // End:: Fire Methods
  },
};
</script>
<style>
span.submit_btn {
  width: 45px;
  height: 45px;
  font-size: 16px;
  border-radius: 10px;
  color: var(--white_clr);
  transition: all 0.3s linear;
  background-color: #f6a513;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
