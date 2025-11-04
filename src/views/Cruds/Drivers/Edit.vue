<template>
  <div class="crud_form_wrapper">
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.editDriver") }}</h4>
    </div>
    <div class="col-12 text-end">
      <v-btn @click="$router.go(-1)" style="color: #007f5f">
        <i class="fas fa-backward"></i>
      </v-btn>
    </div>

    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">
          <!-- Driver Name -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.driverName')"
            v-model.trim="data.name"
            required
          />

          <!-- Email -->
          <base-input
            col="6"
            type="email"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model.trim="data.email"
            required
          />

          <!-- Mobile -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.mobileNumber')"
            v-model.trim="data.mobile"
            required
          />

          <!-- Birth Date -->
          <base-picker-input
            col="6"
            type="date"
            :placeholder="$t('PLACEHOLDERS.birthDate')"
            v-model.trim="data.birth_date"
            required
          />

          <!-- Vehicle Selection -->
          <base-select-input
            col="6"
            :optionsList="vehiclesList"
            :placeholder="$t('PLACEHOLDERS.selectVehicle')"
            v-model="data.vehicle_id"
          />

          <!-- Banking Information Section -->
          <div class="col-12 mb-3 mt-4">
            <h5>{{ $t("PLACEHOLDERS.bankingInformation") }}</h5>
          </div>

          <!-- Bank Name -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.bankName')"
            v-model.trim="data.bank_name"
          />

          <!-- Account Holder Name -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.accountHolderName')"
            v-model.trim="data.account_holder_name"
          />

          <!-- Account Number -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.accountNumber')"
            v-model.trim="data.account_number"
          />

          <!-- IBAN Number -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.ibanNumber')"
            v-model.trim="data.iban_number"
          />

          <!-- Images Section -->
          <div class="col-12 mb-3 mt-4">
            <h5>{{ $t("PLACEHOLDERS.driverImages") }}</h5>
          </div>

          <!-- ID Image Upload -->
          <base-image-upload-input
            col="4"
            identifier="id_image"
            :preSelectedImage="data.id_image.path"
            :placeholder="$t('PLACEHOLDERS.idImage')"
            @selectImage="selectIdImage"
          />

          <!-- License Image Upload -->
          <base-image-upload-input
            col="4"
            identifier="license_image"
            :preSelectedImage="data.license_image.path"
            :placeholder="$t('PLACEHOLDERS.licenseImage')"
            @selectImage="selectLicenseImage"
          />

          <!-- Profile Image Upload -->
          <base-image-upload-input
            col="4"
            identifier="profile_image"
            :preSelectedImage="data.profile_image.path"
            :placeholder="$t('PLACEHOLDERS.profileImage')"
            @selectImage="selectProfileImage"
          />

          <!-- Submit Button -->
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
  name: "EditDriver",

  data() {
    return {
      isWaitingRequest: false,
      vehiclesList: [],
      data: {
        name: null,
        email: null,
        mobile: null,
        birth_date: null,
        created_at: null,
        is_active: null,
        vehicle_id: null,
        bank_name: null,
        account_holder_name: null,
        account_number: null,
        iban_number: null,
        id_image: {
          path: null,
          file: null,
        },
        license_image: {
          path: null,
          file: null,
        },
        profile_image: {
          path: null,
          file: null,
        },
      },
    };
  },

  methods: {
    selectIdImage(selectedImage) {
      this.data.id_image = selectedImage;
    },

    selectLicenseImage(selectedImage) {
      this.data.license_image = selectedImage;
    },

    selectProfileImage(selectedImage) {
      this.data.profile_image = selectedImage;
    },

    validateFormInputs() {
      this.isWaitingRequest = true;
      this.submitForm();
    },

    async submitForm() {
      const REQUEST_DATA = new FormData();

      if (this.data.name) {
        REQUEST_DATA.append("name", this.data.name);
      }
      if (this.data.email) {
        REQUEST_DATA.append("email", this.data.email);
      }
      if (this.data.mobile) {
        REQUEST_DATA.append("mobile", this.data.mobile);
      }
      if (this.data.birth_date) {
        REQUEST_DATA.append("birth_date", this.data.birth_date);
      }
      if (this.data.vehicle_id) {
        REQUEST_DATA.append("vehicle_id", this.data.vehicle_id?.id);
      }
      if (this.data.bank_name) {
        REQUEST_DATA.append("bank_name", this.data.bank_name);
      }
      if (this.data.account_holder_name) {
        REQUEST_DATA.append(
          "account_holder_name",
          this.data.account_holder_name
        );
      }
      if (this.data.account_number) {
        REQUEST_DATA.append("account_number", this.data.account_number);
      }
      if (this.data.iban_number) {
        REQUEST_DATA.append("iban_number", this.data.iban_number);
      }
      if (this.data.id_image.file) {
        REQUEST_DATA.append("id_image", this.data.id_image.file);
      }
      if (this.data.license_image.file) {
        REQUEST_DATA.append("license_image", this.data.license_image.file);
      }
      if (this.data.profile_image.file) {
        REQUEST_DATA.append("profile_image", this.data.profile_image.file);
      }

      REQUEST_DATA.append("_method", "PUT");

      try {
        await this.$axios({
          method: "POST",
          url: `drivers/${this.$route.params?.id}`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.editedSuccessfully"));
        this.$router.push({ path: "/drivers/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },

    async showDriver() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `drivers/${this.$route.params?.id}`,
        });
        const driver = res.data.data.Driver || res.data.data;
        this.data.name = driver.name;
        this.data.email = driver.email;
        this.data.mobile = driver.mobile;
        this.data.birth_date = driver.birth_date;
        this.data.created_at = driver.created_at;
        this.data.is_active = driver.is_active;
        this.data.vehicle_id = driver.vehicle_id;
        this.data.bank_name = driver.bank_name;
        this.data.account_holder_name = driver.account_holder_name;
        this.data.account_number = driver.account_number;
        this.data.iban_number = driver.iban_number;
        this.data.id_image.path = driver.id_image;
        this.data.license_image.path = driver.license_image;
        this.data.profile_image.path = driver.profile_image;
      } catch (error) {
        this.loading = false;
        console.log(error?.response?.data?.message);
      }
    },

    async loadVehicles() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: "vehicles",
        });
        this.vehiclesList = res.data.data.data.map((vehicle) => ({
          id: vehicle.id,
          name: vehicle.name,
          value: vehicle.id,
        }));
      } catch (error) {
        console.log("Error loading vehicles:", error);
      }
    },
  },

  async created() {
    await this.loadVehicles();
    this.showDriver();
  },
};
</script>
