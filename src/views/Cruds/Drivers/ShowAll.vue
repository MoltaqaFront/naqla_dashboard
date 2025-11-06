<template>
  <div class="show_all_content_wrapper">
    <main>
      <!-- Filter Form -->
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
              <!-- Name Search -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.driverName')"
                v-model.trim="filterOptions.name"
              />

              <!-- Mobile Search -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.mobileNumber')"
                v-model.trim="filterOptions.mobile"
              />

              <!-- Email Search -->
              <base-input
                col="4"
                type="text"
                :placeholder="$t('PLACEHOLDERS.email')"
                v-model.trim="filterOptions.email"
              />

              <!-- Status Filter -->
              <base-select-input
                col="4"
                :optionsList="activeStatuses"
                :placeholder="$t('PLACEHOLDERS.status')"
                v-model="filterOptions.is_active"
              />

              <!-- Join Date From -->
              <base-picker-input
                col="4"
                type="date"
                :placeholder="$t('PLACEHOLDERS.fromDate')"
                v-model.trim="filterOptions.from_date"
              />

              <!-- Join Date To -->
              <base-picker-input
                col="4"
                type="date"
                :placeholder="$t('PLACEHOLDERS.toDate')"
                v-model.trim="filterOptions.to_date"
              />

              <!-- Age From -->
              <base-input
                col="3"
                type="number"
                :placeholder="$t('PLACEHOLDERS.ageFrom')"
                v-model.trim="filterOptions.age_from"
              />

              <!-- Age To -->
              <base-input
                col="3"
                type="number"
                :placeholder="$t('PLACEHOLDERS.ageTo')"
                v-model.trim="filterOptions.age_to"
              />
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

      <!-- Table Title -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("PLACEHOLDERS.drivers") }}</h5>
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
          v-if="$can('drivers create', 'drivers')"
        >
          <router-link to="/drivers/create">
            {{ $t("PLACEHOLDERS.addDrivers") }}
          </router-link>
        </div> -->
      </div>

      <!-- Data Table -->
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
        <template v-slot:[`item.id`]="{ index }">
          {{
            (paginations.current_page - 1) * paginations.items_per_page +
            index +
            1
          }}
        </template>

        <template v-slot:no-data>
          {{ $t("TABLES.noData") }}
        </template>

        <!-- Status Toggle -->
        <template v-slot:[`item.is_active`]="{ item }">
          <div
            class="activation"
            dir="ltr"
            style="z-index: 1"
            v-if="$can('drivers activate', 'drivers')"
          >
            <v-switch
              class="mt-2"
              color="success"
              v-model="item.is_active"
              hide-details
              @change="handleActivationChange(item)"
            ></v-switch>
          </div>
        </template>

        <!-- Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <a-tooltip
              placement="bottom"
              v-if="$can('drivers delete', 'drivers')"
            >
              <template slot="title">
                <span>{{ $t("BUTTONS.delete") }}</span>
              </template>
              <button class="btn_delete" @click="selectDeleteItem(item)">
                <i class="fal fa-trash-alt"></i>
              </button>
            </a-tooltip>

            <a-tooltip
              placement="bottom"
              v-if="$can('drivers edit', 'drivers')"
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
              v-if="$can('drivers show', 'drivers')"
            >
              <template slot="title">
                <span>{{ $t("BUTTONS.show") }}</span>
              </template>
              <button class="btn_show" @click="showItem(item)">
                <i class="fal fa-eye"></i>
              </button>
            </a-tooltip>

            <template v-else>
              <i
                class="fal fa-lock-alt fs-5 blue-grey--text text--darken-1"
              ></i>
            </template>
          </div>
        </template>

        <!-- Dialogs -->
        <template v-slot:top>
          <!-- Deactivation Reason Modal -->
          <v-dialog v-model="dialogDeactivate" max-width="600px">
            <v-card>
              <v-card-title>
                <span class="headline">{{
                  $t("TITLES.deactivateDriver")
                }}</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <base-input
                        col="12"
                        type="textarea"
                        :placeholder="$t('PLACEHOLDERS.deactivationReason')"
                        v-model.trim="deactivationReason"
                        required
                      />
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn class="modal_cancel_btn" @click="cancelDeactivation">
                  {{ $t("BUTTONS.cancel") }}
                </v-btn>
                <v-btn
                  class="modal_confirm_btn"
                  @click="confirmDeactivation"
                  :disabled="!deactivationReason"
                >
                  {{ $t("BUTTONS.deactivate") }}
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <!-- Delete Modal -->
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
        </template>
      </v-data-table>
    </main>

    <!-- Pagination -->
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
  </div>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  name: "AllDrivers",

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
      loading: false,
      isWaitingRequest: false,
      filterFormIsActive: false,
      filterOptions: {
        name: null,
        mobile: null,
        email: null,
        is_active: null,
        from_date: null,
        to_date: null,
        age_from: null,
        age_to: null,
      },

      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.Admins.serialNumber"),
          value: "id",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.driverName"),
          value: "name",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.mobileNumber"),
          value: "mobile",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.status"),
          value: "is_active",
          sortable: false,
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.joinDate"),
          value: "created_at",
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

      // Dialog controls
      dialogDelete: false,
      itemToDelete: null,
      dialogDeactivate: false,
      itemToDeactivate: null,
      deactivationReason: "",

      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
    };
  },

  watch: {
    ["$route.query.page"]() {
      this.paginations.current_page = +this.$route.query.page;
      this.setTableRows();
    },
  },

  methods: {
    async submitFilterForm() {
      if (this.$route.query.page !== "1") {
        await this.$router.push({ path: "/drivers/all", query: { page: 1 } });
      }
      this.setTableRows();
    },

    async resetFilter() {
      this.filterOptions = {
        name: null,
        mobile: null,
        email: null,
        is_active: null,
        from_date: null,
        to_date: null,
        age_from: null,
        age_to: null,
      };

      if (this.$route.query.page !== "1") {
        await this.$router.push({ path: "/drivers/all", query: { page: 1 } });
      }
      this.setTableRows();
    },

    updateRouterQueryParam(pagenationValue) {
      this.$router.push({
        query: {
          ...this.$route.query,
          page: pagenationValue,
        },
      });

      document.body.scrollTop = 0;
      document.documentElement.scrollTop = 0;
    },

    async setTableRows() {
      this.loading = true;
      try {
        let res = await this.$axios({
          method: "GET",
          url: "drivers",
          params: {
            page: this.paginations.current_page,
            name: this.filterOptions.name,
            mobile: this.filterOptions.mobile,
            email: this.filterOptions.email,
            is_active: this.filterOptions.is_active?.value,
            from_date: this.filterOptions.from_date,
            to_date: this.filterOptions.to_date,
            age_from: this.filterOptions.age_from,
            age_to: this.filterOptions.age_to,
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

    handleActivationChange(item) {
      if (!item.is_active) {
        // Driver is being deactivated, show reason modal
        this.itemToDeactivate = item;
        this.dialogDeactivate = true;
        // Reset the switch until confirmation
        item.is_active = true;
      } else {
        // Driver is being activated, proceed directly
        this.changeActivationStatus(item);
      }
    },

    cancelDeactivation() {
      this.dialogDeactivate = false;
      this.itemToDeactivate = null;
      this.deactivationReason = "";
    },

    async confirmDeactivation() {
      if (this.itemToDeactivate && this.deactivationReason) {
        // Set the item as inactive and proceed with API call
        this.itemToDeactivate.is_active = false;
        await this.changeActivationStatus(
          this.itemToDeactivate,
          this.deactivationReason
        );
        this.cancelDeactivation();
      }
    },

    async changeActivationStatus(item, reason = null) {
      const REQUEST_DATA = new FormData();
      if (reason) {
        REQUEST_DATA.append("reason", reason);
      }

      try {
        await this.$axios({
          method: "POST",
          url: `drivers/toggle/${item.id}`,
          data: REQUEST_DATA,
        });
        this.setTableRows();
        this.$message.success(this.$t("MESSAGES.changeActivation"));
      } catch (error) {
        this.setTableRows();
        this.$message.error(error.response.data.message);
      }
    },

    editItem(item) {
      this.$router.push({ path: `/drivers/edit/${item.id}` });
    },

    showItem(item) {
      this.$router.push({ path: `/drivers/show/${item.id}` });
    },

    selectDeleteItem(item) {
      this.dialogDelete = true;
      this.itemToDelete = item;
    },

    async confirmDeleteItem() {
      try {
        await this.$axios({
          method: "DELETE",
          url: `drivers/${this.itemToDelete.id}`,
        });
        this.dialogDelete = false;
        this.setTableRows();
        this.$message.success(this.$t("MESSAGES.deletedSuccessfully"));
      } catch (error) {
        this.dialogDelete = false;
        this.$message.error(error.response.data.message);
      }
    },
  },

  created() {
    window.addEventListener("click", () => {
      this.filterFormIsActive = false;
    });
    if (this.$route.query.page) {
      this.paginations.current_page = +this.$route.query.page;
    }
    this.setTableRows();
  },
};
</script>
