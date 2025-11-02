<template>
  <div class="crud_form_wrapper single_show_content_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("TITLES.showClient", { name: data.first_name }) }}</h4>
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
      <!-- <div class="badges_wrapper justify-content-between">
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
      </div> -->
      <!-- ==== End:: Status Badges ==== -->

      <!-- ==== Start:: Client Main Data ==== -->
      <form>
        <div class="row">
          <!-- Start:: Image Upload Input -->
          <base-image-upload-input
            col="12"
            identifier="userImage"
            :preSelectedImage="data.image.path"
            :placeholder="$t('PLACEHOLDERS.userImage')"
            @selectImage="selectImage"
            disabled
          />
          <!-- End:: Image Upload Input -->

          <!-- Start:: first_name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.user_name')"
            v-model.trim="data.user_name"
            disabled
            class="disabled_input"
          />
          <!-- End:: first_name Input -->

          <!-- Start:: last_name Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.phone')"
            v-model.trim="data.phone"
            disabled
            class="disabled_input"
          />
          <!-- End:: last_name Input -->

          <!-- Start:: Phone Input -->
          <base-input
            col="4"
            type="tel"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model.trim="data.email"
            readonly
            class="disabled_input"
          />
          <!-- End:: Phone Input -->

          <!-- Start:: Second Phone Input -->

           <base-image-upload-input
            col="6"
            identifier="id_image"
            :preSelectedImage="data.id_image.path"
            :placeholder="$t('PLACEHOLDERS.id_image')"
            @selectImage="selectImage"
            disabled
          />
           <base-image-upload-input
            col="6"
            identifier="licens_image"
            :preSelectedImage="data.licens_image.path"
            :placeholder="$t('PLACEHOLDERS.licens_image')"
            @selectImage="selectImage"
            disabled
          />

          <!-- Start:: gender Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.bank_name')"
            v-model.trim="data.bank_name"
            readonly
            class="disabled_input"
          />
          <!-- End:: gender Input -->

          <!-- Start:: Joining Date Input -->
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.bank_number')"
            v-model.trim="data.bank_number"
            readonly
            class="disabled_input"
          />
          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.iban')"
            v-model.trim="data.iban"
            readonly
            class="disabled_input"
          />
          <!-- End:: Joining Date Input -->
        </div>
      </form>
      <!-- ==== End:: Client Main Data ==== -->
    </div>
    <!-- End:: Single Step Form Content -->
  </div>
</template>

<script>
export default {
  name: "SingleClient",

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
      },
      // End:: Data
    };
  },

  methods: {
    // Start:: Get Data To Show
    async getDataToShow() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `clients/${this.$route.params.id}`,
        });
        const client = res.data.data.Client;

        this.data.image.path = client.avatar;
        this.data.user_name = client.name;
        this.data.phone = client.mobile;
        this.data.dial_code = client.dial_code;
        this.data.iso_code = client.code;
        this.data.email = client.email;
        this.data.id_image.path = client.identity_image;
        this.data.licens_image.path = client.license_image;
        this.data.bank_name = client.bank_name;
        this.data.bank_number = client.account_number;
        this.data.iban = client.iban;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },
    // End:: Get Data To Show
  },

  created() {
    // Start:: Fire Methods
    this.getDataToShow();
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
