<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.on_boarding") }}</h4>
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
          <!-- Loop over items -->
          <div
            class="row mb-4"
            v-for="(item, index) in data"
            :key="index"
            style="border: 1px solid #eee; padding: 15px; border-radius: 8px"
          >
            <!-- Image (optional) -->
            <base-image-upload-input
              col="12"
              :identifier="`image_${index}`"
              :placeholder="$t('PLACEHOLDERS.image')"
              :preSelectedImage="
                typeof item.image === 'string' ? item.image : item.image?.path
              "
              @selectImage="(img) => selectImage(index, img)"
            />

            <!-- Title Ar -->
            <base-input
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.nameAr')"
              v-model.trim="item.title.ar"
              required
            />

            <!-- Title En -->
            <base-input
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.nameEn')"
              v-model.trim="item.title.en"
              required
            />

            <!-- Order -->
            <base-select-input
              col="2"
              :placeholder="$t('PLACEHOLDERS.order')"
              :optionsList="getAvailableOrderOptions(index)"
              v-model="item.order"
              required
            />

            <!-- Status -->
            <div class="col-lg-2 activation" dir="ltr" style="z-index: 1">
              <v-switch
                col="6"
                class="mt-2"
                color="success"
                v-model="item.is_active"
                hide-details
              ></v-switch>
            </div>
          </div>

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
export default {
  name: "CreatePackage",
  data() {
    return {
      isWaitingRequest: false,
      // Each object = value[index]
      data: [
        {
          order: null,
          title: { ar: "", en: "" },
          is_active: null,
          image: { file: null },
        },
        {
          order: null,
          title: { ar: "", en: "" },
          is_active: null,
          image: { file: null },
        },
        {
          order: null,
          title: { ar: "", en: "" },
          is_active: null,
          image: { file: null },
        },
      ],
      orderOptions: [
        { id: 1, name: "1", value: 1 },
        { id: 2, name: "2", value: 2 },
        { id: 3, name: "3", value: 3 },
      ],
      statusOptions: [
        { id: 1, name: this.$t("STATUS.active"), value: 1 },
        { id: 0, name: this.$t("STATUS.notActive"), value: 0 },
      ],
    };
  },
  methods: {
    selectImage(index, selectedImage) {
      // Ensure we're updating the correct item
      this.$set(this.data, index, {
        ...this.data[index],
        image: selectedImage,
      });
    },
    validateFormInputs() {
      // Check for duplicate orders
      const orders = this.data
        .map((item) => item.order?.value || item.order?.id || item.order)
        .filter((order) => order !== null);
      const uniqueOrders = [...new Set(orders)];

      if (orders.length !== uniqueOrders.length) {
        this.$message.error(
          this.$t("MESSAGES.duplicateOrderError") ||
            "Duplicate order numbers are not allowed"
        );
        return;
      }

      // Check if all required fields are filled
      const hasEmptyFields = this.data.some(
        (item) =>
          !item.title?.ar ||
          !item.title?.en ||
          !item.order ||
          item.is_active === null
      );

      if (hasEmptyFields) {
        this.$message.error(
          this.$t("MESSAGES.fillRequiredFields") ||
            "Please fill all required fields"
        );
        return;
      }

      this.isWaitingRequest = true;
      this.submitForm();
    },
    async submitForm() {
      const REQUEST_DATA = new FormData();

      // Fixed key
      REQUEST_DATA.append("key", "onboarding");

      // Build array structure
      this.data.forEach((item, index) => {
        if (item.order !== undefined && item.order !== null) {
          REQUEST_DATA.append(
            `value[${index}][order]`,
            item.order?.value ?? item.order
          );
        }
        if (item.title?.ar) {
          REQUEST_DATA.append(`value[${index}][title][ar]`, item.title.ar);
        }
        if (item.title?.en) {
          REQUEST_DATA.append(`value[${index}][title][en]`, item.title.en);
        }
        if (item.is_active !== null) {
          REQUEST_DATA.append(
            `value[${index}][is_active]`,
            item.is_active?.value ?? item.is_active
          );
        }
        // ✅ Handle image properly
        if (typeof item.image === "string") {
          // Existing image (URL)
          REQUEST_DATA.append(`value[${index}][image]`, item.image);
        } else if (item.image?.file instanceof File) {
          // Newly uploaded file
          REQUEST_DATA.append(`value[${index}][image]`, item.image.file);
        }
      });

      try {
        await this.$axios({
          method: "POST",
          url: "settings",
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.addedSuccessfully"));
        // this.$router.push({ path: "/home" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response?.data?.message || "Error");
      }
    },

    getAvailableOrderOptions(currentIndex) {
      // Get all selected orders except for current item
      const selectedOrders = this.data
        .map((item, index) => {
          if (index !== currentIndex && item.order) {
            return item.order.value || item.order.id || item.order;
          }
          return null;
        })
        .filter((order) => order !== null);

      // Keep all options, but mark selected ones as disabled
      return this.orderOptions.map((option) => ({
        ...option,
        disabled: selectedOrders.includes(option.value),
      }));
    },

    async getDataToEdit() {
      try {
        let res = await this.$axios.get("settings?key=onboarding");
        const settings = res.data.data[0].value;

        // Map the response data to component data
        this.data = settings.map((item) => ({
          ...item,
          order: {
            id: parseInt(item.order),
            name: item.order,
            value: parseInt(item.order),
          },
        }));
      } catch (error) {
        console.error(error.response.data.message);
      }
    },
  },

  created() {
    this.getDataToEdit();
  },
};
</script>
