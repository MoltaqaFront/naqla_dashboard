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
            v-model.trim="data.package_tax_percentage"
            step="0.01"
            min="0"
            max="100"
          />

          <!-- App Commission Rate -->
          <base-input
            type="number"
            col="6"
            :placeholder="$t('PLACEHOLDERS.financial_liability_ratio')"
            v-model.trim="data.app_commission_rate"
            step="0.01"
            min="0"
            max="100"
          />

          <!-- Cancellation Time Limit (Hours) -->
          <base-input
            type="number"
            col="6"
            :placeholder="$t('PLACEHOLDERS.documentation_amount')"
            v-model.trim="data.cancellation_time_limit_hours"
            min="0"
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
        let res = await this.$axios.get("settings?key=general");
        const settings = res.data.data[0].value;

        // Map the response data to component data
        this.data.package_tax_percentage = settings.package_tax_percentage;
        this.data.documentation_amount = settings.documentation_amount;
        this.data.financial_liability_ratio =
          settings.financial_liability_ratio;
        this.data.verified_financial_liability_ratio =
          settings.verified_financial_liability_ratio;
      } catch (error) {
        console.error(error.response.data.message);
      }
    },
    async submitForm() {
      this.isWaitingRequest = true;
      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("key", "general");

      // Append all form data
      REQUEST_DATA.append(
        "value[package_tax_percentage]",
        this.data.package_tax_percentage
      );
      REQUEST_DATA.append(
        "value[documentation_amount]",
        this.data.documentation_amount
      );
      REQUEST_DATA.append(
        "value[financial_liability_ratio]",
        this.data.financial_liability_ratio
      );
      REQUEST_DATA.append(
        "value[verified_financial_liability_ratio]",
        this.data.verified_financial_liability_ratio
      );

      try {
        await this.$axios.post("settings", REQUEST_DATA);
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
