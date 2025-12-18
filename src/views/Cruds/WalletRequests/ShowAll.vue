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
              <!-- Start:: Delivery Agent Name Input -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.driverName')"
                v-model.trim="filterOptions.driverName"
              />
              <!-- End:: Delivery Agent Name Input -->

              <!-- Start:: Email Input -->
              <base-input
                col="4"
                type="email"
                :placeholder="$t('PLACEHOLDERS.email')"
                v-model.trim="filterOptions.email"
              />
              <!-- End:: Email Input -->

              <!-- Start:: Mobile Number Input -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.mobileNumber')"
                v-model.trim="filterOptions.mobile"
              />
              <!-- End:: Mobile Number Input -->

              <!-- Start:: Order Number Input -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.orderNum')"
                v-model.trim="filterOptions.orderNumber"
              />
              <!-- End:: Order Number Input -->

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

              <!-- Start:: Status Input -->
              <base-select-input
                col="4"
                :optionsList="activeStatuses"
                :placeholder="$t('PLACEHOLDERS.requestStatus')"
                v-model="filterOptions.status"
              />
              <!-- End:: Status Input -->
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
          <h5>{{ $t("SIDENAV.Users.wallet-settlement-requests") }}</h5>
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
        <!-- Start:: Serial Number -->
        <template v-slot:[`item.serial_number`]="{ item, index }">
          {{
            (paginations.current_page - 1) * paginations.items_per_page +
            index +
            1
          }}
        </template>
        <!-- End:: Serial Number -->

        <!-- Start:: Status Column with Translation and Color -->
        <template v-slot:[`item.status`]="{ item }">
          <v-chip
            small
            :color="getStatusColor(item.status_enum || item.status)"
          >
            {{ getStatusTranslation(item.status_enum || item.status) }}
          </v-chip>
        </template>
        <!-- End:: Status Column -->

        <!-- Start:: No Data State -->
        <template v-slot:no-data>
          {{ $t("TABLES.noData") }}
        </template>
        <!-- End:: No Data State -->

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <!-- View Button -->
            <!-- <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("BUTTONS.show") }}</span>
              </template>
              <button class="btn_show" @click="showItem(item)">
                <i class="fad fa-eye"></i>
              </button>
            </a-tooltip> -->

            <!-- Approve Button (only for pending status) -->
            <a-tooltip
              placement="bottom"
              v-if="(item.status_enum || item.status) === 'new'"
            >
              <template slot="title">
                <span>{{ $t("BUTTONS.approve") }}</span>
              </template>
              <button
                class=""
                @click="openRequestStatusModal(item, 'accepted')"
              >
                <i class="fad fa-check-circle"></i>
              </button>
            </a-tooltip>

            <!-- Reject Button (only for pending status) -->
            <a-tooltip
              placement="bottom"
              v-if="(item.status_enum || item.status) === 'new'"
            >
              <template slot="title">
                <span>{{ $t("BUTTONS.reject") }}</span>
              </template>
              <button
                class=""
                @click="openRequestStatusModal(item, 'rejected')"
              >
                <i class="fad fa-times-circle"></i>
              </button>
            </a-tooltip>
          </div>
        </template>
        <!-- End:: Actions -->

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>
          <v-dialog v-model="dialogStatusAccept">
            <v-card>
              <v-card-title class="text-h5 justify-center">
                {{ $t("PLACEHOLDERS.approveSettlementRequest") }}
              </v-card-title>
              <div class="w-100 px-5 pb-5">
                <div class="mt-3">
                  <p class="mb-3">
                    <strong>{{ $t("PLACEHOLDERS.driverName") }}:</strong>
                    {{ modalRequest?.user?.name || "-" }}
                  </p>
                  <p class="mb-3">
                    <strong
                      >{{ $t("PLACEHOLDERS.currentWalletBalance") }}:</strong
                    >
                    {{ modalRequest?.user?.current_user_balance || 0 }}
                    {{ $t("PLACEHOLDERS.riyal") }}
                  </p>
                  <p class="mb-3">
                    <strong>{{ $t("PLACEHOLDERS.requestedAmount") }}:</strong>
                    {{ modalRequest?.amount || 0 }}
                    {{ $t("PLACEHOLDERS.riyal") }}
                  </p>

                  <div class="text-center mt-5">
                    <v-btn
                      class="modal_confirm_btn mx-2"
                      color="success"
                      @click="changeRequestStatus(modalRequest, 'accepted')"
                    >
                      {{ $t("BUTTONS.approve") }}
                    </v-btn>
                    <v-btn
                      class="modal_cancel_btn mx-2"
                      color="error"
                      @click="cancelRequest()"
                    >
                      {{ $t("BUTTONS.cancel") }}
                    </v-btn>
                  </div>
                </div>
              </div>
            </v-card>
          </v-dialog>

          <v-dialog v-model="dialogStatusReject">
            <v-card>
              <v-card-title class="text-h5 justify-center">
                {{ $t("PLACEHOLDERS.rejectSettlementRequest") }}
              </v-card-title>
              <div class="w-100 px-5 pb-5">
                <div class="mt-3">
                  <p class="mb-3">
                    <strong>{{ $t("PLACEHOLDERS.driverName") }}:</strong>
                    {{ modalRequest?.user?.name || "-" }}
                  </p>
                  <p class="mb-3">
                    <strong>{{ $t("PLACEHOLDERS.requestedAmount") }}:</strong>
                    {{ modalRequest?.amount || 0 }}
                    {{ $t("PLACEHOLDERS.riyal") }}
                  </p>

                  <div class="mt-4">
                    <label class="font-weight-bold d-block mb-2">
                      {{ $t("PLACEHOLDERS.rejectionReason") }} *
                    </label>
                    <textarea
                      v-model.trim="modalRequest.rejection_reason"
                      class="w-100 p-3"
                      rows="4"
                      :placeholder="$t('PLACEHOLDERS.enterRejectionReason')"
                      style="border: 1px solid #ccc; border-radius: 4px"
                    ></textarea>
                  </div>

                  <div class="text-center mt-5">
                    <v-btn
                      class="modal_confirm_btn mx-2"
                      color="error"
                      :disabled="!modalRequest.rejection_reason"
                      @click="changeRequestStatus(modalRequest, 'rejected')"
                    >
                      {{ $t("BUTTONS.reject") }}
                    </v-btn>
                    <v-btn
                      class="modal_cancel_btn mx-2"
                      @click="cancelRequest()"
                    >
                      {{ $t("BUTTONS.cancel") }}
                    </v-btn>
                  </div>
                </div>
              </div>
            </v-card>
          </v-dialog>
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
  name: "AllRequestWallets",

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),

    activeStatuses() {
      return [
        {
          id: 1,
          name: this.$t("STATUS.pending"),
          value: "new",
        },
        {
          id: 2,
          name: this.$t("STATUS.approved"),
          value: "accepted",
        },
        {
          id: 3,
          name: this.$t("STATUS.rejected"),
          value: "rejected",
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
        driverName: null,
        email: null,
        mobile: null,
        orderNumber: null,
        dateFrom: null,
        dateTo: null,
        status: null,
      },
      // End:: Filter Data
      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.Admins.serialNumber"),
          value: "serial_number",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.orderNum"),
          value: "serial_number",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.driverName"),
          value: "user.name",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.email"),
          value: "user.email",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.mobileNumber"),
          value: "user.mobile",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.requestedAmount"),
          value: "amount",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.requestDate"),
          value: "created_at",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.requestStatus"),
          value: "status",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("TABLES.Admins.actions"),
          value: "actions",
          sortable: false,
          align: "center",
        },
      ],
      tableRows: [],
      // End:: Table Data

      // Start:: Dialogs Control Data
      dialogStatusAccept: false,
      dialogStatusReject: false,
      selectedDescriptionTextToShow: "",
      // End:: Dialogs Control Data

      // Start:: Pagination Data
      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
      // End:: Pagination Data

      modalRequest: {},
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
    // Start:: Handel Filter
    async submitFilterForm() {
      if (this.$route.query.page !== "1") {
        await this.$router.push({
          path: "/request-wallets/all",
          query: { page: 1 },
        });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.driverName = null;
      this.filterOptions.email = null;
      this.filterOptions.mobile = null;
      this.filterOptions.orderNumber = null;
      this.filterOptions.dateFrom = null;
      this.filterOptions.dateTo = null;
      this.filterOptions.status = null;

      if (this.$route.query.page !== "1") {
        await this.$router.push({
          path: "/request-wallets/all",
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

      // Scroll To Screen's Top After Get requestwallets
      document.body.scrollTop = 0; // For Safari
      document.documentElement.scrollTop = 0; // For Chrome, Firefox, IE and Opera
    },
    async setTableRows() {
      this.loading = true;
      try {
        let res = await this.$axios({
          method: "GET",
          url: "withdraw-requests",
          params: {
            page: this.paginations.current_page,
            driver_name: this.filterOptions.driverName,
            email: this.filterOptions.email,
            mobile: this.filterOptions.mobile,
            order_number: this.filterOptions.orderNumber,
            date_from: this.filterOptions.dateFrom,
            date_to: this.filterOptions.dateTo,
            status: this.filterOptions.status?.value,
          },
        });
        this.loading = false;
        this.tableRows = res.data.data?.data || [];
        this.paginations.last_page = res.data.data?.links?.last_page || 1;
        this.paginations.items_per_page = res.data.data?.links?.per_page || 15;
      } catch (error) {
        this.loading = false;
        console.log(error.response?.data?.message || error.message);
      }
    },
    // End:: Set Table Rows

    openRequestStatusModal(item, status) {
      if (status == "accepted") {
        this.dialogStatusAccept = true;
      } else if (status == "rejected") {
        this.dialogStatusReject = true;
      }
      this.modalRequest = { ...item };
    },

    async changeRequestStatus(modalRequest, status) {
      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("status", status);

      if (status === "rejected" && modalRequest?.rejection_reason) {
        REQUEST_DATA.append("rejection_reason", modalRequest.rejection_reason);
      }

      if (status === "accepted" && modalRequest?.amount) {
        REQUEST_DATA.append("amount", modalRequest.amount);
      }

      try {
        await this.$axios({
          method: "POST",
          url: `withdraw-requests/change-status/${modalRequest.id}`,
          data: REQUEST_DATA,
        });

        this.dialogStatusAccept = false;
        this.dialogStatusReject = false;
        this.setTableRows();
        this.$message.success(this.$t("MESSAGES.editedSuccessfully"));
      } catch (error) {
        this.$message.error(
          error.response?.data?.message || this.$t("MESSAGES.failed")
        );
      }
    },

    // Get status translation
    getStatusTranslation(status) {
      const statusMap = {
        new: this.$t("STATUS.pending"),
        accepted: this.$t("STATUS.approved"),
        rejected: this.$t("STATUS.rejected"),
      };
      return statusMap[status] || status;
    },

    // Get status color
    getStatusColor(status) {
      const colorMap = {
        new: "orange",
        accepted: "green",
        rejected: "red",
      };
      return colorMap[status] || "grey";
    },

    // ==================== Start:: Crud ====================
    // ===== Start:: End
    editItem(item) {
      this.$router.push({ path: `/request-wallets/edit/${item.id}` });
    },
    showItem(item) {
      this.$router.push({ path: `/request-wallets/show/${item.id}` });
    },
    // ===== End:: End
    cancelRequest() {
      this.dialogStatusAccept = false;
      this.dialogStatusReject = false;
    },
    // ==================== End:: Crud ====================
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
    // End:: Fire Methods
  },
};
</script>
