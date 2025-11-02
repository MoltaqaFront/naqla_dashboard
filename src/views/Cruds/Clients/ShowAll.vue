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
              <!-- Start:: Name Input -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.name')"
                v-model.trim="filterOptions.name"
              />
              <!-- End:: Name Input -->

              <!-- Start:: Phone Input -->
              <base-input
                col="4"
                type="tel"
                :placeholder="$t('PLACEHOLDERS.phone')"
                v-model.trim="filterOptions.phone"
              />
              <!-- End:: Phone Input -->

              <!-- Start:: Status Input -->
              <base-select-input
                col="4"
                :optionsList="activeStatuses"
                :placeholder="$t('PLACEHOLDERS.status')"
                v-model="filterOptions.isActive"
              />
              <!-- End:: Status Input -->
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
          <h5>{{ $t("PLACEHOLDERS.manage_all_users") }}</h5>
          <button
            v-if="!filterFormIsActive"
            class="filter_toggler"
            @click.stop="filterFormIsActive = !filterFormIsActive"
          >
            <i class="fal fa-search"></i>
          </button>
        </div>
        <!-- <div
          class="title_route_wrapper"
          v-if="$can('clients create', 'clients')"
        >
          <router-link to="/clients/create">
            {{ $t("PLACEHOLDERS.add_user") }}
          </router-link>
        </div> -->
        <div class="title_route_wrapper">
          <base-button
            class="mt-0 pdf_btn"
            styleType="solid_btn"
            :btnText="$t('BUTTONS.downloadExcel')"
            @fireClick="DownloadExcel"
          >
            <!-- :disabled="pdfDownloadBtnIsLoading" -->
            <template v-slot:btn_icon>
              <i class="fal fa-file-pdf"></i>
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
        <!-- Start:: No Data State -->
        <template v-slot:no-data>
          {{ $t("TABLES.noData") }}
        </template>
        <!-- Start:: No Data State -->

        <template v-slot:[`item.id`]="{ item, index }">
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

        <template v-slot:[`item.type`]="{ item }">
          <h6 class="text-danger" v-if="!item.type">
            {{ $t("TABLES.noData") }}
          </h6>
          <h6 v-else>
            <template v-if="item.type == 'personal'">
              {{ $t("PLACEHOLDERS.personal") }}
            </template>
            <template v-else-if="item.type == 'business'">
              {{ $t("PLACEHOLDERS.business") }}
            </template>
          </h6>
        </template>

        <!-- Start:: Name -->
        <template v-slot:[`item.name`]="{ item }">
          <h6 class="text-danger" v-if="!item.name">
            {{ $t("TABLES.noData") }}
          </h6>
          <h6 v-else>{{ item.name }}</h6>
        </template>
        <!-- End:: Name -->

        <template v-slot:[`item.is_verified`]="{ item }">
          <v-chip
            :color="item.is_verified ? 'green' : 'red'"
            text-color="white"
            small
          >
            <template v-if="item.is_verified">
              {{ $t("PLACEHOLDERS.completed") }}
            </template>
            <template v-else>
              {{ $t("PLACEHOLDERS.notcompleted") }}
            </template>
          </v-chip>
        </template>

        <template v-slot:[`item.mobile`]="{ item }">
          {{ item.country_code + item.mobile }}
        </template>

        <template v-slot:[`item.has_verification`]="{ item }">
          <div class="activation" dir="ltr" style="z-index: 1">
            <v-switch
              class="mt-2"
              color="success"
              v-model="item.has_verification"
              hide-details
              @change="toggleVerfiy(item)"
            ></v-switch>
          </div>
        </template>
        <!-- Start:: Activation -->
        <template v-slot:[`item.is_active`]="{ item }">
          <div class="activation_buttons" dir="ltr" style="z-index: 1">
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{
                  item.is_active
                    ? $t("BUTTONS.deactivate")
                    : $t("BUTTONS.activate")
                }}</span>
              </template>
              <button
                v-if="item.is_active"
                class="btn_deactivate"
                @click="selectDeactivateItem(item)"
              >
                <i class="far fa-times"></i>
              </button>
              <button v-else class="btn_activate" @click="activateItem(item)">
                <i class="far fa-check"></i>
              </button>
            </a-tooltip>
          </div>
        </template>
        <!-- End:: Activation -->

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <!-- <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("PLACEHOLDERS.editWallet") }}</span>
              </template>
              <button class="btn_wallet" @click="showWalletModal(item)">
                <i class="fal fa-wallet"></i>
              </button>
            </a-tooltip> -->
            <!-- <a-tooltip
              placement="bottom"
              v-if="$can('clients show', 'clients')"
            >
              <template slot="title">
                <span>{{ $t("BUTTONS.show") }}</span>
              </template>
              <button class="btn_show" @click="showItem(item)">
                <i class="fal fa-eye"></i>
              </button>
            </a-tooltip> -->

            <a-tooltip
              placement="bottom"
              v-if="$can('clients edit', 'clients') && item?.id !== 1"
            >
              <template slot="title">
                <span>{{ $t("BUTTONS.edit") }}</span>
              </template>
              <button class="btn_edit" @click="editItem(item)">
                <i class="fal fa-edit"></i>
              </button>
            </a-tooltip>
            <a-tooltip
              placement="bottom"
              v-if="$can('clients delete', 'clients')"
              :class="{ disable_parent: item.can_delete === true }"
            >
              <template slot="title">
                <span>{{ $t("BUTTONS.delete") }}</span>
              </template>
              <button
                class="btn_delete"
                @click="selectDeleteItem(item)"
                :class="{ disable_child: item.can_delete === true }"
              >
                <i class="fal fa-trash-alt"></i>
              </button>
            </a-tooltip>
          </div>
        </template>
        <!-- End:: Actions -->

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>
          <!-- Start:: Deactivate Modal -->
          <v-dialog v-model="dialogDeactivate">
            <v-card>
              <v-card-title
                class="text-h5 justify-center"
                v-if="itemToChangeActivationStatus"
              >
                {{
                  $t("TITLES.DeactivateConfirmingMessage", {
                    name:
                      itemToChangeActivationStatus.name ||
                      itemToChangeActivationStatus.first_name,
                  })
                }}
              </v-card-title>

              <form class="w-100">
                <base-input
                  col="12"
                  rows="3"
                  type="textarea"
                  :placeholder="$t('PLACEHOLDERS.deactivateReason')"
                  v-model.trim="deactivateReason"
                  required
                />
              </form>

              <v-card-actions>
                <v-btn
                  class="modal_confirm_btn"
                  @click="confirmDeactivateItem"
                  :disabled="
                    !!!deactivateReason || deactivateReason?.length < 3
                  "
                >
                  {{ $t("BUTTONS.ok") }}
                </v-btn>

                <v-btn
                  class="modal_cancel_btn"
                  @click="dialogDeactivate = false"
                  >{{ $t("BUTTONS.cancel") }}</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-dialog v-model="dialogDelete">
            <v-card>
              <v-card-title class="text-h5 justify-center" v-if="itemToDelete">
                {{
                  $t("TITLES.DeleteConfirmingMessage", {
                    name: itemToDelete.name,
                  })
                }}
              </v-card-title>
              <v-card-actions>
                <v-btn class="modal_confirm_btn" @click="confirmDeleteItem">{{
                  $t("BUTTONS.ok")
                }}</v-btn>

                <v-btn class="modal_cancel_btn" @click="dialogDelete = false">{{
                  $t("BUTTONS.cancel")
                }}</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-dialog v-model="dialogWallet">
            <v-card>
              <v-card-title class="text-h5 justify-center">
                {{
                  $t("PLACEHOLDERS.walletAmount") +
                  " " +
                  itemToChargeWallet?.wallet_balance
                }}
              </v-card-title>

              <!-- <h2>{{ walletBalance }}</h2> -->
              <form class="w-100" @submit.prevent="EditWalletBalance">
                <base-input
                  col="12"
                  type="number"
                  :placeholder="$t('PLACEHOLDERS.amout')"
                  v-model.trim="walletBalance"
                />
                <base-input
                  col="12"
                  type="textarea"
                  :placeholder="$t('PLACEHOLDERS.reason')"
                  v-model.trim="reason"
                />
              </form>

              <v-card-actions>
                <v-btn
                  class="modal_confirm_btn"
                  @click="EditWalletBalance"
                  :disabled="walletBalance?.length == 0"
                >
                  {{ $t("BUTTONS.ok") }}
                </v-btn>

                <v-btn class="modal_cancel_btn" @click="dialogWallet = false">{{
                  $t("BUTTONS.cancel")
                }}</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <!-- End:: Deactivate Modal -->
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
import { mapGetters } from "vuex";

export default {
  name: "AllClients",

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),

    activeStatuses() {
      return [
        {
          id: 1,
          name: this.$t("STATUS.active"),
          value: 1,
        },
        {
          id: 2,
          name: this.$t("STATUS.notActive"),
          value: 0,
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
        name: null,
        phone: null,
        isActive: null,
      },
      // End:: Filter Data

      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.Clients.serialNumber"),
          value: "id",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Clients.name"),
          value: "name",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Clients.phone"),
          value: "mobile",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Clients.email"),
          value: "email",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Users.kind"),
          value: "type",
          align: "center",
          sortable: false,
        },
        // {
        //   text: this.$t("PLACEHOLDERS.acount_status"),
        //   value: "is_verified",
        //   align: "center",
        //   sortable: false,
        // },
        {
          text: this.$t("TABLES.Clients.joiningDate"),
          value: "created_at",
          align: "center",
          sortable: false,
        },
        // {
        //   text: this.$t("PLACEHOLDERS.verfiy"),
        //   value: "has_verification",
        //   align: "center",
        //   sortable: false,
        // },
        {
          text: this.$t("TABLES.Clients.active"),
          value: "is_active",
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
      // End:: Table Data

      // Start:: Pagination Data
      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
      // End:: Pagination Data

      dialogDelete: false,
      itemToDelete: null,

      dialogDeactivate: false,
      itemToChangeActivationStatus: null,
      deactivateReason: null,

      dialogBalance: false,
      itemToBalance: null,
      excel_link: null,

      dialogWallet: false,
      walletBalance: null,
      newWalletBalance: null,
      itemToChargeWallet: null,
      reason: null,
      newWalletBalance: null,
      itemToChargeWallet: null,
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
    async downloadPdf() {
      window.location.href = this.excel_link;
    },
    async EditWalletBalance() {
      const REQUEST_DATA = new FormData();
      // Start:: Append Request Data
      if (this.walletBalance) {
        REQUEST_DATA.append("amount", this.walletBalance);
      }
      if (this.reason) {
        REQUEST_DATA.append("reason", this.reason);
      }
      try {
        await this.$axios({
          method: "POST",
          url: `update-wallet/${this.itemToChargeWallet.id}`,
          data: REQUEST_DATA,
        });
        this.dialogWallet = false;
        this.$message.success(this.$t("PLACEHOLDERS.walletUpdated"));
        this.setTableRows();
        this.itemToChargeWallet = null;
        this.reason = null;
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    showWalletModal(item) {
      this.dialogWallet = true;
      this.walletBalance = item.balance;
      this.itemToChargeWallet = item;
    },
    async confirmDeleteItem() {
      try {
        await this.$axios({
          method: "DELETE",
          url: `clients/${this.itemToDelete.id}`,
        });
        this.dialogDelete = false;
        this.tableRows = this.tableRows.filter((item) => {
          return item.id != this.itemToDelete.id;
        });
        this.setTableRows();
        this.$message.success(this.$t("MESSAGES.deletedSuccessfully"));
      } catch (error) {
        this.dialogDelete = false;
        this.$message.error(error.response.data.message);
      }
    },
    // Start:: Handel Filter
    async submitFilterForm() {
      if (this.$route.query.page !== "1") {
        await this.$router.push({ path: "/clients/all", query: { page: 1 } });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.name = null;
      this.filterOptions.phone = null;
      this.filterOptions.isActive = null;
      if (this.$route.query.page !== "1") {
        await this.$router.push({ path: "/clients/all", query: { page: 1 } });
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
        let nameParam = this.filterOptions.name;
        if (!nameParam) {
          nameParam = null;
        }

        let res = await this.$axios({
          method: "GET",
          url: "clients",
          params: {
            page: this.paginations.current_page,
            name: nameParam,
            mobile: this.filterOptions.phone,
            is_active: this.filterOptions.isActive?.value,
          },
        });
        this.loading = false;
        this.tableRows = res.data.data.data;
        this.paginations.last_page = res.data.data.meta.last_page;
        this.paginations.items_per_page = res.data.data.meta.per_page;
        this.excel_link = res.data.data.export_url;
      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // End:: Set Table Rows
    selectDeleteItem(item) {
      this.dialogDelete = true;
      this.itemToDelete = item;
    },
    // Start:: Change Activation Status
    async toggleStatus(item) {
      try {
        await this.$axios({
          method: "POST",
          url: `clients/toggle/${item.id}`,
        });
        this.setTableRows();
        this.$message.success(this.$t("MESSAGES.changeActivation"));
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    async activateItem(item) {
      try {
        await this.$axios({
          method: "POST",
          url: `clients/toggle/${item.id}`,
        });
        this.setTableRows();
        this.$message.success(this.$t("MESSAGES.changeActivation"));
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    async toggleVerfiy(item) {
      // console.log(item.id);

      // const REQUEST_DATA = new FormData();
      // REQUEST_DATA.append("id", item.id);
      try {
        await this.$axios({
          method: "POST",
          url: `clients/${item.id}/verifications`,
        });
        this.setTableRows();
        this.$message.success(this.$t("PLACEHOLDERS.changeVrification"));
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    // End:: Change Activation Status

    // ==================== Start:: Crud ====================
    // ===== Start:: Show
    showItem(item) {
      this.$router.push({ path: `/clients/show/${item.id}` });
    },
    // ===== End:: Show
    editItem(item) {
      this.$router.push({ path: `/clients/edit/${item.id}` });
    },
    // ===== Start:: Handling Activation & Deactivation
    selectDeactivateItem(item) {
      this.dialogDeactivate = true;
      this.itemToChangeActivationStatus = item;
    },
    async confirmDeactivateItem() {
      this.dialogDeactivate = false;
      let targetItem = this.itemToChangeActivationStatus;
      const REQUEST_DATA = {};
      // Start:: Append Request Data
      if (this.deactivateReason) {
        REQUEST_DATA.reason = this.deactivateReason;
      }
      // Start:: Append Request Data
      try {
        await this.$axios({
          method: "POST",
          url: `clients/toggle/${targetItem.id}`,
          data: REQUEST_DATA,
        });
        this.$message.success(this.$t("MESSAGES.changeActivation"));
        this.setTableRows();
        this.itemToChangeActivationStatus = null;
        this.deactivateReason = null;
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    // ===== End:: Handling Activation & Deactivation

    selectAcceptItem(item) {
      this.dialogBalance = true;
      this.itemToBalance = item;
    },
    async confirmAcceptItem(item) {
      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("balance", this.balance_package);
      // REQUEST_DATA.append("_method", "PUT");

      try {
        await this.$axios({
          method: "POST",
          url: `change-client-balance/${this.itemToBalance.id}`,
          data: REQUEST_DATA,
        });
        this.dialogBalance = false;
        (this.balance_package = null), this.setTableRows();
        this.$message.success(this.$t("MESSAGES.verifiedSuccessfully"));
      } catch (error) {
        this.dialogBalance = false;
        this.$message.error(error.response.data.message);
      }
    },

    async DownloadExcel() {
      await this.$axios({
        method: "GET",
        url: "clients/export-excel",
        params: {
          page: this.paginations.current_page,
          name: this.filterOptions.name,
          mobile: this.filterOptions.phone,
          status: this.filterOptions.isActive?.value,
        },
        responseType: "blob",
      })
        .then((response) => {
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const link = document.createElement("a");
          link.href = url;
          link.setAttribute("download", "clients.xlsx");
          document.body.appendChild(link);
          link.click();
        })
        .catch((error) => {
          this.$message.error(error.response.data.message);
        });
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

.activation_buttons {
  display: flex;
  justify-content: center;
  align-items: center;
}

.btn_activate,
.btn_deactivate {
  width: 35px;
  height: 35px;
  border: none;
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn_activate {
  background-color: #28a745;
  color: white;
}

.btn_activate:hover {
  background-color: #218838;
  transform: scale(1.1);
}

.btn_deactivate {
  background-color: #dc3545;
  color: white;
}

.btn_deactivate:hover {
  background-color: #c82333;
  transform: scale(1.1);
}
</style>
