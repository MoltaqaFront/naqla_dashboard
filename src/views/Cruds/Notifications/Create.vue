<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("BUTTONS.add_notification") }}</h4>
    </div>
    <div class="col-12 text-end">
      <v-btn @click="$router.go(-1)" style="color: #007f5f">
        <i class="fas fa-backward"></i>
      </v-btn>
    </div>
    <!-- End:: Title -->

    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">
          <!-- Start:: service_providers Type Input -->
          <base-select-input
            v-if="data.providers && data.receiverType.value === 'provider'"
            col="6"
            :optionsList="data.providers"
            :placeholder="$t('PLACEHOLDERS.service_providers_plural')"
            v-model="data.provider"
            required
            multiple
          />
          <!-- End:: Drivers Type Input -->

          <div class="col-12">
            <div class="row">
              <!-- Start:: Title Ar Input -->
              <base-input
                col="6"
                type="text"
                :placeholder="$t('PLACEHOLDERS.titleAr')"
                v-model.trim="data.titleAr"
                required
              />
              <!-- End:: Title Ar Input -->

              <!-- Start:: Title En Input -->
              <base-input
                col="6"
                type="text"
                :placeholder="$t('PLACEHOLDERS.titleEn')"
                v-model.trim="data.titleEn"
                required
              />
              <!-- End:: Title En Input -->

              <!-- Start:: Content Ar Input -->
              <base-input
                col="6"
                rows="7"
                type="textarea"
                :placeholder="$t('PLACEHOLDERS.contentAr')"
                v-model.trim="data.contentAr"
                required
              />
              <!-- End:: Content Ar Input -->

              <!-- Start:: Content En Input -->
              <base-input
                col="6"
                rows="7"
                type="textarea"
                :placeholder="$t('PLACEHOLDERS.contentEn')"
                v-model.trim="data.contentEn"
                required
              />
              <!-- End:: Content En Input -->
            </div>
          </div>
          <!-- Start:: Receiver Type Input -->
          <base-select-input
            col="6"
            :optionsList="receiverTypes"
            :placeholder="$t('PLACEHOLDERS.receiverType')"
            v-model="data.receiverType"
            required
          />
          <!-- End:: Receiver Type Input -->

          <!-- Start:: Clients Type Input -->
          <base-select-input
            v-if="data.clients && data.receiverType.value === 'client'"
            col="6"
            :optionsList="data.clients"
            :placeholder="$t('PLACEHOLDERS.clients_menu')"
            v-model="data.client"
            required
            multiple
          />
          <!-- End:: Clients Type Input -->

          <!-- Start:: Drivers Selection Input -->
          <base-select-input
            v-if="data.drivers && data.receiverType.value === 'driver'"
            col="6"
            :optionsList="data.drivers"
            :placeholder="$t('PLACEHOLDERS.drivers')"
            v-model="data.driver"
            required
            multiple
          />
          <!-- End:: Drivers Selection Input -->

          <!-- Start:: Submit Button Wrapper -->
          <div class="btn_wrapper">
            <base-button
              class="mt-2"
              styleType="primary_btn"
              :btnText="$t('BUTTONS.save')"
              :isLoading="isWaitingRequest"
              :disabled="isWaitingRequest"
            />
          </div>
          <!-- End:: Submit Button Wrapper -->
        </div>
      </form>
    </div>
    <!-- END:: Single Step Form Content -->
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

export default {
  name: "CreateNotification",

  computed: {
    // Start:: Vuex Getters
    ...mapGetters({
      allClientsData: "ApiGetsModule/allClientsData",
      allDriversData: "ApiGetsModule/allDriversData",
    }),
    // End:: Vuex Getters

    receiverTypes() {
      return [
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.allClients"),
          value: "allClients",
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.allDrivers"),
          value: "allDrivers",
        },
        {
          id: 3,
          name: this.$t("PLACEHOLDERS.specificClients"),
          value: "client",
        },
        {
          id: 4,
          name: this.$t("PLACEHOLDERS.specificDrivers"),
          value: "driver",
        },
        {
          id: 5,
          name: this.$t("PLACEHOLDERS.all"),
          value: "all",
        },
      ];
    },
  },

  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      // Start:: Data Collection To Send
      data: {
        receiverType: {
          id: null,
          name: null,
          value: null,
        },
        clients: [],
        providers: [],
        drivers: [],
        client: [],
        provider: [],
        driver: [],
        titleAr: null,
        titleEn: null,
        contentAr: null,
        contentEn: null,
      },

      // End:: Data Collection To Send
    };
  },
  watch: {
    // 'data.clients'() {
    //   this.data.clients.forEach((value) => {
    //     if (value.id == '-1') {
    //       this.data.clients = this.allClientsData
    //       return;
    //     }
    //   })
    // },
    // 'data.drivers'() {
    //   this.data.drivers.forEach((value) => {
    //     if (value.id == '-1') {
    //       this.data.drivers = this.allDriversData
    //       return;
    //     }
    //   })
    // }
  },

  methods: {
    // End:: Vuex Actions

    async getClients() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `clients?page=0&limit=0&is_active=1`,
        });

        this.data.clients = res.data.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },

    async getProviders() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `accepted-providers?page=0&limit=0`,
        });
        this.data.providers = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },

    async getDrivers() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `drivers?page=0&limit=0`,
        });
        this.data.drivers = res.data.data?.data || [];
      } catch (error) {
        console.log(error.response?.data?.message || error.message);
      }
    },

    // Start:: validate Form Inputs
    validateFormInputs() {
      this.isWaitingRequest = true;

      // if (!this.data.receiverType) {
      //   this.isWaitingRequest = false;
      //   this.$message.error(this.$t("VALIDATION.receiverType"));
      //   return;
      // } else if (
      //   this.data.receiverType?.value === "clients" &&
      //   this.data.clients.length === 0
      // ) {
      //   this.isWaitingRequest = false;
      //   this.$message.error(this.$t("VALIDATION.clientAtLeast"));
      //   return;
      // } else if (
      //   this.data.receiverType?.value === "providers" &&
      //   this.data.providers.length === 0
      // ) {
      //   this.isWaitingRequest = false;
      //   this.$message.error(this.$t("VALIDATION.driverAtLeast"));
      //   return;
      // }

      // else
      if (!this.data.titleAr) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.titleAr"));
        return;
      } else if (!this.data.titleEn) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.titleEn"));
        return;
      } else if (!this.data.contentAr) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.contentAr"));
        return;
      } else if (!this.data.contentEn) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.contentEn"));
        return;
      } else {
        this.submitForm();
        return;
      }
    },
    // End:: validate Form Inputs

    // Start:: Submit Form
    async submitForm() {
      const REQUEST_DATA = new FormData();
      // Start:: Append Request Data

      // Handle different receiver types
      if (this.data.receiverType.value === "allClients") {
        REQUEST_DATA.append("to_type", "allClients");
      } else if (this.data.receiverType.value === "allDrivers") {
        REQUEST_DATA.append("to_type", "allDrivers");
      } else if (this.data.receiverType.value === "all") {
        REQUEST_DATA.append("to_type", "all");
      } else if (this.data.receiverType.value === "client") {
        // Specific clients - send users array
        this.data.client.forEach((element) => {
          REQUEST_DATA.append(`users[]`, element.id);
        });
      } else if (this.data.receiverType.value === "driver") {
        // Specific drivers - send users array
        this.data.driver.forEach((element) => {
          REQUEST_DATA.append(`users[]`, element.id);
        });
      }

      REQUEST_DATA.append("title[ar]", this.data.titleAr);
      REQUEST_DATA.append("title[en]", this.data.titleEn);
      REQUEST_DATA.append("body[ar]", this.data.contentAr);
      REQUEST_DATA.append("body[en]", this.data.contentEn);
      // End:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `notification/store`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.sentSuccessfully"));
        this.$router.push({ path: "/all-notifications/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
    // End:: Submit Form
  },

  created() {
    // Start:: Fire Methods
    this.getClients();
    this.getDrivers();
    // this.getProviders();
    // End:: Fire Methods
  },
};
</script>
