<template>
  <div class="crud_form_wrapper">
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.editType") }}</h4>
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
  name: "EditVehicle",

  data() {
    return {
      isWaitingRequest: false,
      data: {
        name_ar: null,
        name_en: null,
        active: null,
        created_at: null,
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
      REQUEST_DATA.append("_method", "PUT");

      try {
        await this.$axios({
          method: "POST",
          url: `vehicles/${this.$route.params?.id}`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.editedSuccessfully"));
        this.$router.push({ path: "/vehicles/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },

    async showVehicle() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `vehicles/${this.$route.params?.id}`,
        });
        this.data.name_ar = res.data.data.Vehicle.name_ar;
        this.data.name_en = res.data.data.Vehicle.name_en;
        this.data.created_at = res.data.data.Vehicle.created_at;
        this.data.active = res.data.data.Vehicle.is_active;
      } catch (error) {
        this.loading = false;
        console.log(error?.response?.data?.message);
      }
    },
  },

  created() {
    this.showVehicle();
  },
};
</script>
