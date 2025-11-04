<template>
  <div class="show_all_content_wrapper">
    <main>
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
              <base-input
                col="5"
                type="text"
                :placeholder="$t('PLACEHOLDERS.name')"
                v-model.trim="filterOptions.name"
              />

              <base-select-input
                col="5"
                :optionsList="activeStatuses"
                :placeholder="$t('PLACEHOLDERS.status')"
                v-model="filterOptions.is_active"
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

      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("PLACEHOLDERS.vehicles") }}</h5>
          <button
            v-if="!filterFormIsActive"
            class="filter_toggler"
            @click.stop="filterFormIsActive = !filterFormIsActive"
          >
            <i class="fal fa-search"></i>
          </button>
        </div>

        <div
          class="title_route_wrapper"
          v-if="$can('vehicles create', 'vehicles')"
        >
          <router-link to="/vehicles/create">
            {{ $t("PLACEHOLDERS.addvehicles") }}
          </router-link>
        </div>
      </div>

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

        <template v-slot:[`item.is_active`]="{ item }">
          <div
            class="activation"
            dir="ltr"
            style="z-index: 1"
            v-if="$can('vehicles activate', 'vehicles')"
          >
            <v-switch
              class="mt-2"
              color="success"
              v-model="item.is_active"
              hide-details
              @change="changeActivationStatus(item)"
            ></v-switch>
          </div>
        </template>

        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <a-tooltip
              placement="bottom"
              v-if="$can('vehicles delete', 'vehicles')"
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
              v-if="$can('vehicles edit', 'vehicles')"
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
              v-if="$can('vehicles show', 'vehicles')"
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

        <template v-slot:top>
          <description-modal
            v-if="dialogDescription"
            :modalIsOpen="dialogDescription"
            :modalDesc="selectedDescriptionTextToShow"
            @toggleModal="dialogDescription = !dialogDescription"
          />

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
  name: "AllVehicles",

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
        is_active: null,
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
          text: this.$t("PLACEHOLDERS.name"),
          value: "name",
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
          text: this.$t("PLACEHOLDERS.created_at"),
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

      dialogDescription: false,
      selectedDescriptionTextToShow: "",
      dialogDelete: false,
      itemToDelete: null,

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
        await this.$router.push({ path: "/vehicles/all", query: { page: 1 } });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.name = null;
      this.filterOptions.is_active = null;

      if (this.$route.query.page !== "1") {
        await this.$router.push({ path: "/vehicles/all", query: { page: 1 } });
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
          url: "vehicles",
          params: {
            page: this.paginations.current_page,
            name: this.filterOptions.name,
            is_active: this.filterOptions.is_active?.value,
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

    showReplayModal(replay) {
      this.dialogDescription = true;
      this.selectedDescriptionTextToShow = replay;
    },

    showImageModal(image) {
      this.dialogImage = true;
      this.selectedItemImage = image;
    },

    async changeActivationStatus(item) {
      const REQUEST_DATA = new FormData();
      try {
        await this.$axios({
          method: "POST",
          url: `vehicles/toggle/${item.id}`,
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
      this.$router.push({ path: `/vehicles/edit/${item.id}` });
    },
    showItem(item) {
      this.$router.push({ path: `/vehicles/show/${item.id}` });
    },

    selectDeleteItem(item) {
      this.dialogDelete = true;
      this.itemToDelete = item;
    },

    async confirmDeleteItem() {
      try {
        await this.$axios({
          method: "DELETE",
          url: `vehicles/${this.itemToDelete.id}`,
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
