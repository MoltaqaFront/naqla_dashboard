<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.editBrand") }}</h4>
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
          <!-- <base-image-upload-input
            col="12"
            identifier="countery_image"
            :preSelectedImage="data.image.path"
            :placeholder="$t('PLACEHOLDERS.countery_image')"
            @selectImage="selectImage"
            required
          /> -->
          <!-- Start:: Name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.nameAr')"
            v-model.trim="data.name_ar"
            required
          />
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.nameEn')"
            v-model.trim="data.name_en"
            required
          />
          <base-select-input
            col="4"
            :optionsList="vehicelTypes"
            :placeholder="$t('PLACEHOLDERS.vehicelType')"
            v-model="data.vehicelType"
            required
          />
          <!-- End:: Name Input -->

          <!-- Start:: Deactivate Switch Input -->
          <!-- <div class="input_wrapper switch_wrapper my-5 col-6">
            <v-switch
              color="green"
              :label="
                data.active
                  ? $t('PLACEHOLDERS.active')
                  : $t('PLACEHOLDERS.notActive')
              "
              v-model="data.active"
              hide-details
            ></v-switch>
          </div> -->
          <!-- End:: Deactivate Switch Input -->

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
import { mapActions, mapGetters } from "vuex";
export default {
  name: "EditCountry",

  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      // Start:: Data Collection To Send
      data: {
        image: {
          path: null,
          file: null,
        },
        name_ar: null,
        name_en: null,
        active: null,
        vehicelType: null,
      },
      vehicelTypes: [],
      // End:: Data Collection To Send
    };
  },

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),
  },

  methods: {
    selectImage(selectedImage) {
      this.data.image = selectedImage;
    },
    // Start:: validate Form Inputs
    validateFormInputs() {
      this.isWaitingRequest = true;
      this.submitForm();
    },

    async submitForm() {
      const REQUEST_DATA = new FormData();
      // Start:: Append Request Data

      if (this.data.vehicelType) {
        REQUEST_DATA.append("vehicle_type_id", this.data.vehicelType?.id);
      }
      if (this.data.name_ar) {
        REQUEST_DATA.append("name[ar]", this.data.name_ar);
      }
      if (this.data.name_en) {
        REQUEST_DATA.append("name[en]", this.data.name_en);
      }
      REQUEST_DATA.append("_method", "PUT");
      // REQUEST_DATA.append("is_active", this.data.active ? 1 : 0);
      try {
        await this.$axios({
          method: "POST",
          url: `vehicle-brands/${this.$route.params?.id}`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.editedSuccessfully"));
        this.$router.push({ path: "/vehiclebrands/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
    async getvehicelTypes() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `vehicle-types?page=0&limit=0&is_active=1`,
        });
        this.vehicelTypes = res.data.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },
    // start show data
    async showCountry() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `vehicle-brands/${this.$route.params?.id}`,
        });
        this.data.name_ar = res.data.data.VehicleBrand.name_ar;
        this.data.name_en = res.data.data.VehicleBrand.name_en;
        this.data.vehicelType = res.data.data.VehicleBrand.vehicle_type;
        this.data.created_at = res.data.data.VehicleBrand.created_at;
        this.data.active = res.data.data.VehicleBrand.is_active;
      } catch (error) {
        this.loading = false;
        console.log(error?.response?.data?.message);
      }
    },
    // end show data
  },

  created() {
    this.showCountry();
    this.getvehicelTypes();
  },
};
</script>
