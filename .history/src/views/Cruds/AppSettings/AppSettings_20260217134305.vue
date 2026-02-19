<template>
  <div class="crud_form_wrapper app_settings_class">
    <div class="table_title_wrapper">
      <div class="title_text_wrapper">
        <h5 style="color: #007f5f" class="font-weight-bold">
          {{ $t("SIDENAV.settings.general") }}
        </h5>
      </div>
      <div class="col-12 text-end">
        <v-btn @click="$router.go(-1)" style="color: #007f5f">
          <i class="fas fa-backward"></i>
        </v-btn>
      </div>
    </div>
    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">
          <!-- VAT Rate -->
          <base-input
            type="number"
            col="6"
            :placeholder="$t('PLACEHOLDERS.vat_rate')"
            v-model.trim="data.tax"
            step="0.01"
            min="0"
            max="100"
          />

          <!-- App Commission Rate -->
          <base-input
            type="number"
            col="6"
            :placeholder="
              $t('PLACEHOLDERS.price_per_km') + ` (${$t('PLACEHOLDERS.riyal')})`
            "
            v-model.trim="data.price_per_km"
            step="0.01"
            min="0"
            max="100"
          />

          <!-- Cancellation Time Limit (Hours) -->
          <base-input
            type="number"
            col="6"
            :placeholder="$t('PLACEHOLDERS.driver_order_percentage') + '(%)'"
            v-model.trim="data.driver_order_percentage"
            min="0"
          />
          <!-- VAT Rate -->
          <base-input
            type="number"
            col="6"
            :placeholder="$t('PLACEHOLDERS.vat_rate')"
            v-model.trim="data.fixed_tax"
            step="0.01"
            min="0"
            max="100"
          />
          <!-- <base-input
            type="number"
            col="6"
            :placeholder="$t('PLACEHOLDERS.cancellation_time_limit_hours')"
            v-model.trim="data.cancellation_time_limit_hours"
            min="0"
          /> -->

          <div class="btn_wrapper">
            <base-button
              class="mt-2"
              styleType="primary_btn"
              :btnText="$t('BUTTONS.save')"
              :isLoading="isWaitingRequest"
              :disabled="isWaitingRequest"
            />
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: "AppSetting",
  data() {
    return {
      isWaitingRequest: false,
      data: {
        tax: null,
        price_per_km: null,
        driver_order_percentage: null,

        package_tax_percentage: null,
        documentation_amount: null,
        financial_liability_ratio: null,
        verified_financial_liability_ratio: null,
      },
    };
  },
  methods: {
    async getDataToEdit() {
      try {
        let res = await this.$axios.get("settings?key=general-settings");
        const settings = res.data.data.data[0].value;

        // Map the response data to component data
        this.data.tax = settings.tax;
        this.data.price_per_km = settings.price_per_km;
        this.data.driver_order_percentage = settings.driver_order_percentage;
      } catch (error) {
        console.error(error.response.data.message);
      }
    },
    async submitForm() {
      this.isWaitingRequest = true;
      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("key", "general-settings");

      // Append all form data
      REQUEST_DATA.append("value[tax]", this.data.tax);
      REQUEST_DATA.append("value[price_per_km]", this.data.price_per_km);
      REQUEST_DATA.append(
        "value[financial_liability_ratio]",
        this.data.financial_liability_ratio
      );
      REQUEST_DATA.append(
        "value[driver_order_percentage]",
        this.data.driver_order_percentage
      );

      try {
        await this.$axios.post("settings?key=general-settings", REQUEST_DATA);
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.savedSuccessfully"));
        this.getDataToEdit();
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
    validateFormInputs() {
      this.submitForm();
    },
  },
  created() {
    this.getDataToEdit();
  },
};
</script>
