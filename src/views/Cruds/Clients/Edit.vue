<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("TITLES.editUser") }}</h4>
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
            identifier="userImage"
            :preSelectedImage="data.image.path"
            :placeholder="$t('PLACEHOLDERS.userImage')"
            @selectImage="selectImage"
          />
          <!-- End:: Image Upload Input -->

          <!-- Start:: User Name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.user_name')"
            v-model.trim="data.user_name"
            required
            class=""
          />
          <!-- End:: User Name Input -->

          <!-- Start:: Phone Input -->
          <base-phone-input
            col="4"
            required
            v-model="data.phone"
            @dialCode="dialCode"
            @isoCode="isoCode"
            :placeholder="$t('PLACEHOLDERS.phone')"
            :defaultCountry="data.iso_code"
            :key="key"
          />
          <!-- End:: Phone Input -->

          <!-- Start:: Email Input -->
          <base-input
            col="4"
            type="email"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model.trim="data.email"
            required
          />
          <!-- End:: Email Input -->

          <!-- Start:: User Type Input -->
          <base-select-input
            col="4"
            :optionsList="userTypes"
            :placeholder="$t('PLACEHOLDERS.userType')"
            v-model="data.userType"
            required
          />
          <!-- End:: User Type Input -->

          <!-- Start:: Commercial Register Number (only for companies) -->
          <base-input
            v-if="data.userType && data.userType.value === 1"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.commercial_register_number')"
            v-model.trim="data.commercial_register_number"
          />
          <!-- End:: Commercial Register Number -->

          <!-- Start:: Discount Percentage (only for companies) -->
          <base-input
            v-if="data.userType && data.userType.value === 1"
            col="4"
            type="number"
            min="0"
            max="100"
            step="0.01"
            :placeholder="$t('PLACEHOLDERS.discount_percentage')"
            v-model.number="data.discount_percentage"
          />
          <!-- End:: Discount Percentage -->

          <!-- Start:: Password Section -->
          <div class="col-12">
            <div class="input_wrapper switch_wrapper my-3">
              <v-switch
                color="green"
                :label="$t('PLACEHOLDERS.editPassword')"
                v-model="data.enableEditPassword"
                hide-details
              ></v-switch>
            </div>
          </div>

          <Transition name="fadeInUp" mode="out-in">
            <div class="col-12" v-if="data.enableEditPassword">
              <div class="row">
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
              </div>
            </div>
          </Transition>
          <!-- End:: Password Section -->

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
        user_name: null,
        phone: null,
        dial_code: null,
        iso_code: null,
        email: null,
        userType: null,
        commercial_register_number: null,
        discount_percentage: null,
        password: null,
        password_confirmation: null,
        enableEditPassword: false,
        bank_name: null,
        bank_number: null,
        iban: null,
        numberOfVisits: null,
        lastVisit: null,
      },
      allAreas: [],
      roles: [],
      key: 0,
      // End:: Data Collection To Send
    };
  },

  computed: {
    // Start:: Vuex Getters
    ...mapGetters({
      allRoles: "ApiGetsModule/allRoles",
    }),
    // End:: Vuex Getters

    userTypes() {
      return [
        {
          id: 0,
          name: this.$t("PLACEHOLDERS.personal"),
          value: 0,
        },
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.business"),
          value: 1,
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
          url: `clients/${this.id}`,
        });

        const client = res.data.data.Client;

        this.data.image.path = client.avatar;
        this.data.user_name = client.name;
        this.data.phone = client.mobile;
        this.data.dial_code = client?.dial_code;
        this.data.iso_code = client?.code;
        this.data.email = client.email;

        // Set user type based on is_company field
        this.data.userType = this.userTypes.find(
          (type) => type.value === (client.is_company ? 1 : 0)
        );

        // Set company-specific fields if user is a company
        if (client.is_company) {
          this.data.commercial_register_number =
            client.commercial_register_number;
          this.data.discount_percentage = client.discount_percentage;
        }
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
      // Password validation
      if (this.data.enableEditPassword) {
        if (!this.data.password) {
          this.$message.error(this.$t("VALIDATION.password"));
          return;
        }
        if (this.data.password !== this.data.password_confirmation) {
          this.$message.error(this.$t("VALIDATION.passwordConfirmation"));
          return;
        }
      }

      // User type validation
      if (!this.data.userType) {
        this.$message.error(this.$t("VALIDATION.userType"));
        return;
      }

      // Company-specific validation
      if (this.data.userType.value === 1) {
        if (!this.data.commercial_register_number) {
          this.$message.error(this.$t("VALIDATION.commercial_register_number"));
          return;
        }
        if (
          this.data.discount_percentage === null ||
          this.data.discount_percentage === undefined
        ) {
          this.$message.error(this.$t("VALIDATION.discount_percentage"));
          return;
        }
        if (
          this.data.discount_percentage < 0 ||
          this.data.discount_percentage > 100
        ) {
          this.$message.error(this.$t("VALIDATION.discount_percentage_range"));
          return;
        }
      }

      this.isWaitingRequest = true;
      this.submitForm();
    },
    // End:: validate Form Inputs

    // Start:: Submit Form
    async submitForm() {
      const REQUEST_DATA = new FormData();
      if (this.data.image.file) {
        REQUEST_DATA.append("avatar", this.data.image.file);
      }

      if (this.data.user_name) {
        REQUEST_DATA.append("name", this.data.user_name);
      }

      if (this.data.phone) {
        REQUEST_DATA.append("mobile", this.data.phone);
        REQUEST_DATA.append(
          "dial_code",
          this.data?.dial_code?.startsWith("+")
            ? this.data?.dial_code
            : "+" + this.data?.dial_code
        );
        REQUEST_DATA.append("code", this.data.iso_code);
      }

      if (this.data.email) {
        REQUEST_DATA.append("email", this.data.email);
      }

      // User type and company-related fields
      if (this.data.userType) {
        REQUEST_DATA.append("is_company", this.data.userType.value);

        // Add company-specific fields only if user is a company
        if (this.data.userType.value === 1) {
          if (this.data.commercial_register_number) {
            REQUEST_DATA.append(
              "commercial_register_number",
              this.data.commercial_register_number
            );
          }
          if (
            this.data.discount_percentage !== null &&
            this.data.discount_percentage !== undefined
          ) {
            REQUEST_DATA.append(
              "discount_percentage",
              this.data.discount_percentage
            );
          }
        }
      }

      // Password fields (only if editing password is enabled)
      if (this.data.enableEditPassword) {
        if (this.data.password) {
          REQUEST_DATA.append("password", this.data.password);
        }
        if (this.data.password_confirmation) {
          REQUEST_DATA.append(
            "password_confirmation",
            this.data.password_confirmation
          );
        }
      }

      REQUEST_DATA.append("_method", "PUT");
      // Start:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `clients/${this.id}`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.editedSuccessfully"));
        this.$router.push({ path: "/clients/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },
    // End:: Submit Form
  },

  async created() {
    // Start:: Fire Methods
    this.getRoles();
    this.getDataToEdit();
    this.getAllareas();
    // End:: Fire Methods
  },
};
</script>
