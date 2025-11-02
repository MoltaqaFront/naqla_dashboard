<template>
  <div class="crud_form_wrapper single_show_content_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.showProvider", { name: data.first_name }) }}</h4>
    </div>
    <div class="col-12 text-end">
      <v-btn @click="$router.go(-1)" style="color: #26a1d6">
        <i class="fas fa-backward"></i>
      </v-btn>
    </div>
    <!-- End:: Title -->

    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <!-- ==== Start:: Status Badges ==== -->
      <div class="badges_wrapper justify-content-between">
        <div class="wrapper">
          <v-chip color="amber darken-2" text-color="white">
            {{ $t("TITLES.numberOfVisits", { number: data.numberOfVisits }) }}
          </v-chip>
          <v-chip color="amber darken-2" text-color="white">
            {{ $t("TITLES.lastVisit", { date: data.lastVisit }) }}
          </v-chip>
        </div>

        <v-chip :color="data.active ? 'green' : 'red'" text-color="white">
          {{ data.active ? $t("STATUS.active") : $t("STATUS.notActive") }}
        </v-chip>
      </div>
      <!-- ==== End:: Status Badges ==== -->

      <!-- ==== Start:: Client Main Data ==== -->
      <form>
        <div class="row">
          <!-- Start:: Image Upload Input -->
          <base-image-upload-input
            col="12"
            identifier="logo"
            :preSelectedImage="data.image.path"
            :placeholder="$t('PLACEHOLDERS.logo')"
            @selectImage="selectImage"
            disabled
          />

          <!-- Start:: first_name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.owner_name')"
            v-model.trim="data.name"
            disabled
          />
          <!-- End:: first_name Input -->

          <!-- Start:: first_name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.exhibitor_name')"
            v-model.trim="data.exhibitor_name"
            disabled
          />
          <!-- End:: first_name Input -->
          <base-select-input
            col="4"
            :optionsList="exhibitions"
            :placeholder="$t('PLACEHOLDERS.exhibitor_type_trans')"
            v-model="data.exhibition"
            disabled
          />
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.exp_years')"
            v-model.trim="data.exp_years"
            disabled
          />
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.verfiy')"
            v-model.trim="data.verfiy"
            disabled
          />
          <!-- {{ data.commision }} -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.commision')"
            v-model.trim="data.commision"
            disabled
          />
          <!-- Start:: last_name Input -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model.trim="data.email"
            disabled
          />
          <!-- End:: last_name Input -->

          <!-- Start:: mobile Input -->
          <base-phone-input
            col="6"
            disabled
            v-model="data.phone"
            @dialCode="dialCode"
            @isoCode="isoCode"
            :placeholder="$t('PLACEHOLDERS.phone')"
            :defaultCountry="data.iso_code"
            :key="key"
          />
          <!-- End:: mobile Input -->
          <h2 v-if="data.branches?.length">
            {{ $t("PLACEHOLDERS.branches") }}
          </h2>
          <div
            v-if="data.branches?.length"
            class="row col-12"
            v-for="item in data?.branches"
            :key="item"
          >
            <base-input
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.name')"
              v-model.trim="item.name"
              disabled
            />
            <base-input
              v-if="item.city?.country"
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.country')"
              v-model.trim="item.city.country.name"
              disabled
            />
            <base-input
              v-if="item.city"
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.city')"
              v-model.trim="item.city.name"
              disabled
            />
          </div>

          <div
            class="col-12"
            v-if="data.id_image?.path || data.licens_image?.path"
          >
            <h6>
              {{ $t("TABS.attachments") }}
            </h6>

            <div class="row">
              <base-image-upload-input
                v-if="data.exhibition?.value == 'individual'"
                col="6"
                identifier="id_image"
                :preSelectedImage="data.id_image.path"
                :placeholder="$t('PLACEHOLDERS.id_image')"
                @selectImage="selectImage"
                disabled
              />

              <base-image-upload-input
                v-else
                col="6"
                identifier="licens_image"
                :preSelectedImage="data.licens_image.path"
                :placeholder="$t('PLACEHOLDERS.commercial_register')"
                @selectImage="selectImage"
                disabled
              />
            </div>
          </div>

          <div
            class="col-12"
            v-if="data.bank_name || data.bank_number || data.iban"
          >
            <h6>
              {{ $t("PLACEHOLDERS.bank_info") }}
            </h6>

            <div class="row">
              <base-input
                col="6"
                type="text"
                :placeholder="$t('PLACEHOLDERS.bank_name')"
                v-model.trim="data.bank_name"
                disabled
              />
              <base-input
                col="6"
                type="text"
                :placeholder="$t('PLACEHOLDERS.bank_number')"
                v-model.trim="data.bank_number"
                disabled
              />
              <base-input
                col="6"
                type="text"
                :placeholder="$t('PLACEHOLDERS.iban')"
                v-model.trim="data.iban"
                disabled
              />
            </div>
          </div>
        </div>
      </form>
      <!-- ==== End:: Client Main Data ==== -->
    </div>
    <!-- End:: Single Step Form Content -->
  </div>
</template>

<script>
import BasePhoneInput from "@/components/formInputs/BasePhoneInput.vue";
export default {
  name: "SingleClient",
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
      // Start:: Loading Data
      isWaitingRequest: false,
      // End:: Loading Data

      // Start:: Table Data
      addressesTableHeaders: [
        { text: this.$t("TABLES.Addresses.serialNumber") },
        { text: this.$t("TABLES.Addresses.address") },
        { text: this.$t("TABLES.Addresses.longitude") },
        { text: this.$t("TABLES.Addresses.latitude") },
        { text: this.$t("TABLES.Addresses.type") },
        { text: this.$t("TABLES.Addresses.isDefault") },
      ],
      // End:: Table Data

      // Start:: Data
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
        email: null,
        bank_name: null,
        bank_number: null,
        iban: null,
        numberOfVisits: null,
        lastVisit: null,
        name: null,
        exhibition: null,
        exhibitor_name: null,
        mobile: null,
        password: null,
        password_confirmation: null,
        dial_code: null,
        academic_year: null,
        exp_years: null,
        verfiy: null,
        commision: null,
        academic_stage: null,
        specialization: null,
        gender: null,
        foundation: null,
        foundation_name: null,
        date: null,
        iso_code: null,
        branches: null,
      },
      // End:: Data
      exhibitions: [],
    };
  },

  methods: {
    // Start:: Get Data To Show
    async getDataToShow() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `providers/${this.$route.params.id}`,
        });
        const client = res.data.data.Provider;

        this.data.image.path = client.logo;
        this.data.id_image.path = client?.identity_image;
        this.data.licens_image.path = client?.commercial_register_or_license;
        this.data.name = client.name;
        this.data.exhibitor_name = client.exhibitor_name;
        this.data.exhibition = {
          name: client.exhibitor_type_trans,
          value: client.exhibitor_type,
        };
        this.data.bank_name = client?.bank_name;
        this.data.bank_number = client?.account_number;
        this.data.iban = client?.iban;

        this.data.phone = client.mobile;
        this.data.dial_code = client.dial_code;
        this.data.iso_code = client.code;
        this.data.email = client.email;
        this.data.lastVisit = client.last_logged;
        this.data.numberOfVisits = client.login_count;
        this.data.exp_years = client.experience_years;
        this.data.commision = client.commission ?? "-";
        this.data.verfiy = client.has_verification
          ? this.$i18n.t("PLACEHOLDERS.is_verified")
          : this.$i18n.t("PLACEHOLDERS.not_verified");
        this.data.active = client.is_active;
        this.data.branches = client.branches;
      } catch (error) {
        console.log(error.response.data.message);
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
    // End:: Get Data To Show
  },

  created() {
    // Start:: Fire Methods
    this.getDataToShow();
    this.getExhibitions();
    // End:: Fire Methods
  },
};
</script>
<style>
.preview-container img {
  max-width: 100%;
  max-height: 200px;
  border: 1px solid #ccc;
  border-radius: 15px;
}
</style>
