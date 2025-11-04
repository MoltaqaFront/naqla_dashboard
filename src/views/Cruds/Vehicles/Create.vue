<template>
  <div class="crud_form_wrapper">
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.addvehicles") }}</h4>
    </div>
    <div class="col-12 text-end">
      <v-btn @click="$router.go(-1)" style="color: #007f5f">
        <i class="fas fa-backward"></i>
      </v-btn>
    </div>

    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.nameAr')"
            v-model.trim="data.name_ar"
            required
          />
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.nameEn')"
            v-model.trim="data.name_en"
            required
          />

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
  name: "CreateVehicle",

  data() {
    return {
      isWaitingRequest: false,
      data: {
        name_ar: null,
        name_en: null,
        active: true,
      },
    };
  },

  methods: {
    validateFormInputs() {
      this.isWaitingRequest = true;
      this.submitForm();
    },

    async submitForm() {
      const REQUEST_DATA = new FormData();
      if (this.data.name_ar) {
        REQUEST_DATA.append("name_ar", this.data.name_ar);
      }
      if (this.data.name_en) {
        REQUEST_DATA.append("name_en", this.data.name_en);
      }
      REQUEST_DATA.append("is_active", this.data.active ? 1 : 0);

      try {
        await this.$axios({
          method: "POST",
          url: `vehicles`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.addedSuccessfully"));
        this.$router.push({ path: "/vehicles/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
  },
};
</script>
