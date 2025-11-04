<template>
  <div class="crud_form_wrapper">
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.showDriver") }}</h4>
    </div>
    <div class="col-12 text-end">
      <v-btn @click="$router.go(-1)" style="color: #007f5f">
        <i class="fas fa-backward"></i>
      </v-btn>
    </div>

    <div class="single_step_form_content_wrapper">
      <div class="row">
        <!-- Driver Information -->
        <div class="col-12 mb-4">
          <h5>{{ $t("PLACEHOLDERS.driverInformation") }}</h5>
        </div>

        <!-- Driver Name -->
        <base-input
          col="4"
          type="text"
          :placeholder="$t('PLACEHOLDERS.driverName')"
          v-model.trim="data.name"
          disabled
        />

        <!-- Email -->
        <base-input
          col="4"
          type="email"
          :placeholder="$t('PLACEHOLDERS.email')"
          v-model.trim="data.email"
          disabled
        />

        <!-- Mobile -->
        <base-input
          col="4"
          type="text"
          :placeholder="$t('PLACEHOLDERS.mobileNumber')"
          v-model.trim="data.mobile"
          disabled
        />

        <!-- Age (calculated from birth date) -->
        <base-input
          col="4"
          type="text"
          :placeholder="$t('PLACEHOLDERS.age')"
          :value="calculatedAge"
          disabled
        />

        <!-- Join Date -->
        <base-picker-input
          col="4"
          type="date"
          :placeholder="$t('PLACEHOLDERS.joinDate')"
          v-model.trim="data.created_at"
          disabled
        />

        <!-- Status -->
        <div class="input_wrapper switch_wrapper my-5 col-4">
          <v-switch
            color="green"
            :label="
              data.is_active
                ? $t('PLACEHOLDERS.active')
                : $t('PLACEHOLDERS.notActive')
            "
            v-model="data.is_active"
            hide-details
            disabled
          ></v-switch>
        </div>

        <!-- Vehicle Information -->
        <base-input
          col="4"
          type="text"
          :placeholder="$t('PLACEHOLDERS.assignedVehicle')"
          :value="data.vehicle_name"
          disabled
        />

        <!-- Banking Information Section -->
        <div class="col-12 mb-4 mt-4">
          <h5>{{ $t("PLACEHOLDERS.bankingInformation") }}</h5>
        </div>

        <!-- Bank Name -->
        <base-input
          col="3"
          type="text"
          :placeholder="$t('PLACEHOLDERS.bankName')"
          v-model.trim="data.bank_name"
          disabled
        />

        <!-- Account Holder Name -->
        <base-input
          col="3"
          type="text"
          :placeholder="$t('PLACEHOLDERS.accountHolderName')"
          v-model.trim="data.account_holder_name"
          disabled
        />

        <!-- Account Number -->
        <base-input
          col="3"
          type="text"
          :placeholder="$t('PLACEHOLDERS.accountNumber')"
          v-model.trim="data.account_number"
          disabled
        />

        <!-- IBAN Number -->
        <base-input
          col="3"
          type="text"
          :placeholder="$t('PLACEHOLDERS.ibanNumber')"
          v-model.trim="data.iban_number"
          disabled
        />

        <!-- Images Section -->
        <div class="col-12 mb-4 mt-4">
          <h5>{{ $t("PLACEHOLDERS.driverImages") }}</h5>
        </div>

        <!-- Profile Image -->
        <div class="col-4" v-if="data.profile_image.path">
          <div class="image_preview_wrapper">
            <label>{{ $t("PLACEHOLDERS.profileImage") }}</label>
            <div class="image_container">
              <img
                :src="data.profile_image.path"
                alt="Profile Image"
                class="preview_image"
              />
            </div>
          </div>
        </div>

        <!-- ID Image -->
        <div class="col-4" v-if="data.id_image.path">
          <div class="image_preview_wrapper">
            <label>{{ $t("PLACEHOLDERS.idImage") }}</label>
            <div class="image_container">
              <img
                :src="data.id_image.path"
                alt="ID Image"
                class="preview_image"
              />
            </div>
          </div>
        </div>

        <!-- License Image -->
        <div class="col-4" v-if="data.license_image.path">
          <div class="image_preview_wrapper">
            <label>{{ $t("PLACEHOLDERS.licenseImage") }}</label>
            <div class="image_container">
              <img
                :src="data.license_image.path"
                alt="License Image"
                class="preview_image"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ShowDriver",

  data() {
    return {
      data: {
        name: null,
        email: null,
        mobile: null,
        birth_date: null,
        created_at: null,
        is_active: null,
        vehicle_name: null,
        bank_name: null,
        account_holder_name: null,
        account_number: null,
        iban_number: null,
        id_image: {
          path: null,
        },
        license_image: {
          path: null,
        },
        profile_image: {
          path: null,
        },
      },
    };
  },

  computed: {
    calculatedAge() {
      if (this.data.birth_date) {
        const today = new Date();
        const birthDate = new Date(this.data.birth_date);
        let age = today.getFullYear() - birthDate.getFullYear();
        const monthDiff = today.getMonth() - birthDate.getMonth();

        if (
          monthDiff < 0 ||
          (monthDiff === 0 && today.getDate() < birthDate.getDate())
        ) {
          age--;
        }

        return `${age} ${this.$t("PLACEHOLDERS.years")}`;
      }
      return "";
    },
  },

  methods: {
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
        this.data.vehicle_name =
          driver.vehicle?.name || driver.vehicle_name || "Not assigned";
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
  },

  created() {
    this.showDriver();
  },
};
</script>

<style scoped>
.image_preview_wrapper {
  margin-bottom: 20px;
}

.image_preview_wrapper label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
}

.image_container {
  width: 100%;
  max-width: 200px;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.preview_image {
  width: 100%;
  height: 150px;
  object-fit: cover;
  display: block;
}
</style>
