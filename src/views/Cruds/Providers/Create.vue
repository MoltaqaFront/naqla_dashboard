<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.add_provider") }}</h4>
    </div>
    <div class="col-12 text-end">
      <v-btn @click="$router.go(-1)" style="color: #26a1d6">
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
            v-model="data.mobile"
            @dialCode="dialCode"
            @isoCode="isoCode"
            :placeholder="$t('PLACEHOLDERS.phone')"
            :defaultCountry="data.iso_code"
            :key="key"
          />
          <!-- End:: mobile Input -->

          <input
            type="text"
            name="fake-user"
            style="display: none"
            autocomplete="username"
          />
          <input
            type="password"
            name="fake-pass"
            style="display: none"
            autocomplete="new-password"
          />

          <!-- Start:: Password Input -->
          <base-input
            col="6"
            type="password"
            :placeholder="$t('PLACEHOLDERS.password')"
            v-model.trim="data.password"
            required
          />
          <!-- End:: Password Input -->

          <!-- Start:: Confirm Password Input -->
          <base-input
            col="6"
            type="password"
            :placeholder="$t('PLACEHOLDERS.confirmPassword')"
            v-model.trim="data.password_confirmation"
            required
          />
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
export default {
  name: "CreateArea",
  components: {
    BasePhoneInput,
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
        name: null,
        email: null,
        exhibition: null,
        exhibitor_name: null,
        mobile: null,
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
        // active: true,
      },
      exhibitions: [],

      // End:: Data Collection To Send
    };
  },

  computed: {
    allTypes() {
      return [
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.subscription"),
          value: "subscription",
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.commission"),
          value: "commission",
        },
      ];
    },
  },

  methods: {
    isoCode(iosCode) {
      this.data.iso_code = iosCode;
    },
    dialCode(dialCode) {
      console.log(dialCode);
      this.data.dial_code = dialCode;
    },
    selectImage(selectedImage) {
      this.data.image = selectedImage;
    },
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

      REQUEST_DATA.append("email", this.data.email);
      REQUEST_DATA.append("mobile", this.data.mobile);
      REQUEST_DATA.append("code", this.data.iso_code);
      REQUEST_DATA.append("dial_code", "+" + this.data.dial_code);
      REQUEST_DATA.append("password", this.data.password);
      REQUEST_DATA.append(
        "password_confirmation",
        this.data.password_confirmation
      );
      // REQUEST_DATA.append("is_active", +this.data.active);
      // Start:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `providers`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.addedSuccessfully"));
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
    this.getExhibitions();
  },
};
</script>
