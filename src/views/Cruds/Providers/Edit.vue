<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.editProvider") }}</h4>
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
          <base-image-upload-input
            col="12"
            identifier="logo"
            :preSelectedImage="data.image.path"
            :placeholder="$t('PLACEHOLDERS.logo')"
            @selectImage="selectImage"
          />

          <!-- Start:: first_name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.owner_name')"
            v-model.trim="data.name"
            required
          />
          <!-- End:: first_name Input -->

          <!-- Start:: first_name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.exhibitor_name')"
            v-model.trim="data.exhibitor_name"
            required
          />
          <!-- End:: first_name Input -->
          <base-select-input
            col="4"
            :optionsList="exhibitions"
            :placeholder="$t('PLACEHOLDERS.exhibitor_type_trans')"
            v-model="data.exhibition"
            required
          />

          <base-input
            col="4"
            type="number"
            :placeholder="$t('PLACEHOLDERS.exp_years')"
            v-model.trim="data.exp_years"
            required
          />
          <!-- <base-select-input
            col="4"
            :optionsList="verfiyOptions"
            :placeholder="$t('PLACEHOLDERS.verfiy')"
            v-model.trim="data.verfiy"
            required
          /> -->
          <!-- {{ data.commision }} -->
          <base-input
            col="4"
            type="number"
            :placeholder="$t('PLACEHOLDERS.commision')"
            v-model.trim="data.commision"
          />

          <!-- Start:: last_name Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model.trim="data.email"
            required
          />
          <!-- End:: last_name Input -->

          <!-- Start:: mobile Input -->
          <base-phone-input
            col="6"
            required
            v-model="data.phone"
            @dialCode="dialCode"
            @isoCode="isoCode"
            :placeholder="$t('PLACEHOLDERS.phone')"
            :defaultCountry="data.iso_code"
            :key="key"
          />
          <!-- End:: mobile Input -->

          <!-- End:: Confirm Password Input -->

          <!-- Start:: Deactivate Switch Input -->
          <!-- <div class="input_wrapper switch_wrapper my-5">
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
import BasePhoneInput from "@/components/formInputs/BasePhoneInput.vue";
import { mapGetters, mapActions } from "vuex";

export default {
  name: "EditAdmin",
  components: {
    BasePhoneInput,
  },
  props: {
    id: {
      type: String,
      required: true,
    },
  },

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
        id_image: {
          path: null,
          file: null,
        },
        licens_image: {
          path: null,
          file: null,
        },
        name: null,
        email: null,
        exhibition: null,
        exhibitor_name: null,
        exp_years: null,
        verfiy: null,
        commision: null,
        phone: null,
        password: null,
        password_confirmation: null,
        dial_code: null,
        academic_year: null,
        academic_stage: null,
        specialization: null,
        gender: null,
        foundation: null,
        foundation_name: null,
        date: null,
        iso_code: null,
      },
      exhibitions: [],
      roles: [],
      // End:: Data Collection To Send
    };
  },

  computed: {
    // Start:: Vuex Getters
    ...mapGetters({
      allRoles: "ApiGetsModule/allRoles",
    }),
    // End:: Vuex Getters

    verfiyOptions() {
      return [
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.is_verified"),
          value: 1,
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.not_verified"),
          value: 0,
        },
      ];
    },
  },

  methods: {
    dialCode(dialCode) {
      this.data.dial_code = dialCode;
    },
    isoCode(iosCode) {
      this.data.iso_code = iosCode;
    },
    async getAllareas() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: "areas?page=0&limit=0&is_active=1",
        });
        // console.log("All Data ==>", res.data.data);
        this.allAreas = res.data.data;
      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    async getRoles() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: "roles?page=0&limit=0&status=1",
        });
        this.roles = res.data.data.data;
      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // Start:: Get Data To Show
    async getDataToEdit() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `providers/${this.id}`,
        });

        const client = res.data.data.Provider;

        this.data.image.path = client.logo;
        this.data.name = client.name;
        this.data.exhibitor_name = client.exhibitor_name;
        this.data.exp_years = client.experience_years;
        // this.data.verfiy =
        //   client.has_verification == true
        //     ? {
        //         id: 1,
        //         name: this.$t("PLACEHOLDERS.is_verified"),
        //         value: 1,
        //       }
        //     : {
        //         id: 2,
        //         name: this.$t("PLACEHOLDERS.not_verified"),
        //         value: 0,
        //       };
        this.data.commision = client.commission;
        this.data.phone = client.mobile;
        this.data.dial_code = client.dial_code;
        this.data.iso_code = client.code;
        this.data.exhibition = {
          name: client.exhibitor_type_trans,
          value: client.exhibitor_type,
        };
        this.data.email = client.email;
        this.data.lastVisit = client.last_logged;
        this.data.numberOfVisits = client.login_count;
        this.data.exp_years = client.experience_years;
        // this.data.verfiy = client.is_verified
        //   ? this.$i18n.t("PLACEHOLDERS.is_verified")
        //   : this.$i18n.t("PLACEHOLDERS.not_verified");
        // this.data.commision = client.login_count;
        this.data.active = client.is_active;
      } catch (error) {
        console.log(error.response?.data?.message || error.message);
      }
    },
    // End:: Get Data To Show

    // Start:: Select Upload Image
    selectImage(selectedImage) {
      this.data.image = selectedImage;
    },
    selectImage1(selectedImage) {
      this.data.id_image = selectedImage;
    },
    selectImage2(selectedImage) {
      this.data.licens_image = selectedImage;
    },
    // End:: Select Upload Image

    // Start:: validate Form Inputs
    validateFormInputs() {
      this.isWaitingRequest = true;

      this.submitForm();
    },
    // End:: validate Form Inputs

    // Start:: Submit Form
    async submitForm() {
      const REQUEST_DATA = new FormData();
      if (this.data.image.file) {
        REQUEST_DATA.append("logo", this.data.image.file);
      }
      REQUEST_DATA.append("name", this.data.name);
      REQUEST_DATA.append("exhibitor_name", this.data.exhibitor_name);
      REQUEST_DATA.append("exhibitor_type", this.data.exhibition?.value);

      if (this.data.exp_years) {
        REQUEST_DATA.append("experience_years", this.data.exp_years);
      }
      // REQUEST_DATA.append("has_verification", this.data.verfiy?.value);
      if (this.data.commision) {
        REQUEST_DATA.append("commission", this.data.commision);
      }

      REQUEST_DATA.append("email", this.data.email);
      REQUEST_DATA.append("mobile", this.data.phone);
      REQUEST_DATA.append("code", this.data.iso_code);
      REQUEST_DATA.append(
        "dial_code",
        this.data?.dial_code?.startsWith("+")
          ? this.data?.dial_code
          : "+" + this.data?.dial_code
      );
      REQUEST_DATA.append("_method", "PUT");
      // Start:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `providers/${this.id}`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.editedSuccessfully"));
        this.$router.push({ path: "/providers/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
    async getExhibitions() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `providers-exhibition-type?page=0&limit=0&is_active=1`,
        });
        this.exhibitions = res.data.data?.map((ele, index) => ({
          name: ele.value,
          id: index,
          value: ele.key,
        }));
      } catch (error) {
        console.log(error.response.data.message);
      }
    },
    // End:: Submit Form
  },

  async created() {
    // Start:: Fire Methods
    this.getExhibitions();
    this.getRoles();
    this.getAllareas();
    this.getDataToEdit();
    // End:: Fire Methods
  },
};
</script>
