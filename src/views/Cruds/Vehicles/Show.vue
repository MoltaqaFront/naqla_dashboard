<template>
  <div class="crud_form_wrapper">
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.showType") }}</h4>
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
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.nameAr')"
            v-model.trim="data.name_ar"
            disabled
          />
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.nameEn')"
            v-model.trim="data.name_en"
            disabled
          />

          <base-picker-input
            col="4"
            type="date"
            :placeholder="$t('TABLES.Workplaces.date')"
            v-model.trim="data.created_at"
            disabled
          />

          <div class="input_wrapper switch_wrapper my-5 col-6">
            <v-switch
              color="green"
              :label="
                data.active
                  ? $t('PLACEHOLDERS.active')
                  : $t('PLACEHOLDERS.notActive')
              "
              v-model="data.active"
              hide-details
              disabled
            ></v-switch>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: "ShowVehicle",

  data() {
    return {
      isWaitingRequest: false,
      data: {
        name_ar: null,
        name_en: null,
        created_at: null,
        active: null,
      },
    };
  },

  methods: {
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
