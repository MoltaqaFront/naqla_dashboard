<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.view_delegate_join_request") }}</h4>
    </div>
    <div class="col-12 text-end">
      <v-btn @click="$router.go(-1)" style="color: #007f5f">
        <i class="fas fa-backward"></i>
      </v-btn>
    </div>
    <!-- End:: Title -->

    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <form>
        <div class="row">
          <!-- Start:: Personal Photo -->
          <div class="preview-container text-center my-3">
            <h6 style="color: #007f5f">
              {{ $t("PLACEHOLDERS.personal_photo") }}
            </h6>
            <img
              v-if="data.avatar"
              col="12"
              :src="data.avatar"
              :alt="$t('PLACEHOLDERS.personal_photo')"
              style="max-width: 300px; max-height: 300px; border-radius: 8px"
            />
          </div>
          <!-- End:: Personal Photo -->

          <!-- Start:: Basic Information -->
          <div class="col-12 mb-4">
            <h5
              style="
                color: #007f5f;
                border-bottom: 2px solid #007f5f;
                padding-bottom: 10px;
              "
            >
              {{ $t("PLACEHOLDERS.delegate_delivery_data") }}
            </h5>
          </div>

          <!-- Start:: Delegate Name -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.delegate_name')"
            v-model.trim="data.name"
            disabled
          />
          <!-- End:: Delegate Name -->

          <!-- Start:: Mobile Number -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.phone')"
            v-model.trim="data.mobile"
            style="direction: ltr"
            disabled
          />
          <!-- End:: Mobile Number -->

          <!-- Start:: Email -->
          <base-input
            v-if="data.email"
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model.trim="data.email"
            disabled
          />
          <!-- End:: Email -->

          <!-- Start:: Vehicle Type -->
          <base-input
            v-if="data.vehicle"
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.vehicle_type')"
            v-model.trim="data.vehicle"
            disabled
          />
          <!-- End:: Vehicle Type -->

          <!-- Start:: Join Request Date -->
          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.request_date')"
            v-model.trim="data.created_at"
            disabled
          />
          <!-- End:: Join Request Date -->

          <!-- Start:: Documents Section -->
          <div class="col-12 mt-4 mb-3">
            <h5
              style="
                color: #007f5f;
                border-bottom: 2px solid #007f5f;
                padding-bottom: 10px;
              "
            >
              {{ $t("PLACEHOLDERS.required_documents") }}
            </h5>
          </div>

          <!-- Start:: Identity Image -->
          <div class="col-6 mb-4">
            <h6 style="color: #007f5f; margin-bottom: 15px">
              {{ $t("PLACEHOLDERS.identity_image") }}
            </h6>
            <div class="preview-container text-center">
              <img
                v-if="data.identity_image"
                :src="data.identity_image"
                :alt="$t('PLACEHOLDERS.identity_image')"
                style="
                  max-width: 100%;
                  max-height: 300px;
                  border-radius: 8px;
                  border: 1px solid #ddd;
                "
                @click="openImageModal(data.identity_image)"
              />
              <p v-else class="text-muted">
                {{ $t("PLACEHOLDERS.no_image_available") }}
              </p>
            </div>
          </div>
          <!-- End:: Identity Image -->

          <!-- Start:: Driving License -->
          <div class="col-6 mb-4">
            <h6 style="color: #007f5f; margin-bottom: 15px">
              {{ $t("PLACEHOLDERS.driving_license") }}
            </h6>
            <div class="preview-container text-center">
              <img
                v-if="data.license_image"
                :src="data.license_image"
                :alt="$t('PLACEHOLDERS.driving_license')"
                style="
                  max-width: 100%;
                  max-height: 300px;
                  border-radius: 8px;
                  border: 1px solid #ddd;
                "
                @click="openImageModal(data.license_image)"
              />
              <p v-else class="text-muted">
                {{ $t("PLACEHOLDERS.no_image_available") }}
              </p>
            </div>
          </div>
          <!-- End:: Driving License -->

          <!-- Start:: Car Image -->
          <div class="col-6 mb-4">
            <h6 style="color: #007f5f; margin-bottom: 15px">
              {{ $t("PLACEHOLDERS.carImage") }}
            </h6>
            <div class="preview-container text-center">
              <img
                v-if="data.car_image"
                :src="data.car_image"
                :alt="$t('PLACEHOLDERS.carImage')"
                style="
                  max-width: 100%;
                  max-height: 300px;
                  border-radius: 8px;
                  border: 1px solid #ddd;
                "
                @click="openImageModal(data.car_image)"
              />
              <p v-else class="text-muted">
                {{ $t("PLACEHOLDERS.no_image_available") }}
              </p>
            </div>
          </div>
          <!-- End:: Car Image -->

          <!-- Start:: License Plate Image -->
          <div class="col-6 mb-4">
            <h6 style="color: #007f5f; margin-bottom: 15px">
              {{ $t("PLACEHOLDERS.licensePlateImage") }}
            </h6>
            <div class="preview-container text-center">
              <img
                v-if="data.license_plate"
                :src="data.license_plate"
                :alt="$t('PLACEHOLDERS.licensePlateImage')"
                style="
                  max-width: 100%;
                  max-height: 300px;
                  border-radius: 8px;
                  border: 1px solid #ddd;
                "
                @click="openImageModal(data.license_plate)"
              />
              <p v-else class="text-muted">
                {{ $t("PLACEHOLDERS.no_image_available") }}
              </p>
            </div>
          </div>
          <!-- End:: License Plate Image -->
        </div>
      </form>
    </div>
    <!-- END:: Single Step Form Content -->
  </div>
</template>

<script>
export default {
  name: "ShowProvider",
  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      file: null,
      fileType: "",

      // Start:: Data Collection To Send
      data: {
        id: null,
        name: null,
        email: null,
        mobile: null,
        avatar: null,
        license_image: null,
        identity_image: null,
        car_image: null,
        license_plate: null,
        created_at: null,
        vehicle: null,
      },
      // End:: Data Collection To Send
      areas: [],
      arabicRegex: /^[\u0600-\u06FF\s]+$/,
      EnRegex: /[\u0600-\u06FF]/,

      providerPoints: [],
      coordinates: [],
    };
  },

  methods: {
    disabledDate(current) {
      return current && current < moment().startOf("day");
    },

    onCopy(event) {
      event.preventDefault();
    },
    onPaste(event) {
      event.preventDefault();
    },

    validateInput() {
      // Remove non-Arabic characters from the input
      this.data.nameAr = this.data.nameAr.replace(/[^\u0600-\u06FF\s]/g, "");
    },
    removeArabicCharacters() {
      this.data.nameEn = this.data.nameEn.replace(this.EnRegex, "");
    },

    handleFileSelected({ file, fileType }) {
      this.file = file; // Store the selected file in your data
      this.fileType = fileType; // Store the selected file in your data
    },
    handleFileRemoved() {
      this.file = null; // Reset the file when it's removed
      this.fileType = "";
    },

    handleSaveProvider(coordinates) {
      // Handle the saved provider coordinates from the child component

      this.coordinates = coordinates;
    },
    // start show data
    async showData() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `delegate-requests/${this.$route.params?.id}`,
        });
        const delegateData = res.data.data?.DelegateRequest;

        this.data.id = delegateData.id;
        this.data.name = delegateData.name;
        this.data.email = delegateData.email;
        this.data.mobile = delegateData.mobile;
        this.data.avatar = delegateData.avatar;
        this.data.license_image = delegateData.license_image;
        this.data.identity_image = delegateData.identity_image;
        this.data.car_image = delegateData.car_image;
        this.data.license_plate = delegateData.license_plate;
        this.data.created_at = delegateData.created_at;
        this.data.vehicle = delegateData.vehicle;
      } catch (error) {
        this.loading = false;
        console.log(error?.response?.data?.message);
      }
    },

    // Open image in modal for better viewing
    openImageModal(imageUrl) {
      if (imageUrl) {
        window.open(imageUrl, "_blank");
      }
    },
    // end show data
  },

  created() {
    this.showData();
  },
  computed: {
    // Add any computed properties if needed
  },
};
</script>
<style scope>
.download_btn {
  color: #007f5f !important;
  transition: background-color 0.4s, color 0.4s;
}

.download_btn:hover {
  background-color: #007f5f;
  color: #fff !important;
}
</style>
