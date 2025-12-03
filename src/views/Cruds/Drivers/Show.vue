<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.showDriver") }}</h4>
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
          <!-- Start:: Driver Name Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.driverName')"
            v-model.trim="data.name"
            disabled
          />

          <!-- Start:: Email Input -->
          <base-input
            col="6"
            type="email"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model.trim="data.email"
            disabled
          />
          <!-- End:: Email Input -->

          <!-- Start:: Mobile Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.mobileNumber')"
            v-model.trim="data.mobile"
            disabled
          />
          <!-- End:: Mobile Input -->

          <!-- Start:: Age Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.age')"
            :value="calculatedAge"
            disabled
          />
          <!-- End:: Age Input -->

          <!-- Start:: Join Date Input -->
          <base-picker-input
            col="6"
            type="date"
            :placeholder="$t('PLACEHOLDERS.joinDate')"
            v-model.trim="data.created_at"
            disabled
          />
          <!-- End:: Join Date Input -->

          <!-- Start:: Vehicle Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.assignedVehicle')"
            :value="data.vehicle_name"
            disabled
          />
          <!-- End:: Vehicle Input -->

          <!-- Start:: Bank Name Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.bankName')"
            v-model.trim="data.bank_name"
            disabled
          />
          <!-- End:: Bank Name Input -->

          <!-- Start:: Account Holder Name Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.accountHolderName')"
            v-model.trim="data.account_holder_name"
            disabled
          />
          <!-- End:: Account Holder Name Input -->

          <!-- Start:: Account Number Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.accountNumber')"
            v-model.trim="data.account_number"
            disabled
          />
          <!-- End:: Account Number Input -->

          <!-- Start:: IBAN Number Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.ibanNumber')"
            v-model.trim="data.iban_number"
            disabled
          />
          <!-- End:: IBAN Number Input -->

          <!-- Start:: Deactivate Switch Input -->
          <div class="input_wrapper switch_wrapper my-5 col-6">
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
          <!-- End:: Deactivate Switch Input -->
        </div>
      </form>
    </div>
    <!-- END:: Single Step Form Content -->
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
