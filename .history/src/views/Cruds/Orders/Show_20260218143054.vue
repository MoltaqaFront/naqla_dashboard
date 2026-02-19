<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.showOrder") }}</h4>
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
          <!-- Start:: Order Basic Info -->
          <div class="col-12 mb-4">
            <h5>{{ $t("PLACEHOLDERS.orderInformation") }}</h5>
          </div>

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.orderNum')"
            v-model.trim="data.order_number"
            disabled
          />

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.orderDate')"
            v-model.trim="data.created_at"
            disabled
          />

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.orderStatus')"
            v-model="data.status"
            disabled
          />

          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.orderName')"
            v-model.trim="data.name"
            disabled
          />

          <base-input
            col="6"
            type="text"
            :placeholder="$t('PLACEHOLDERS.internalReference')"
            v-model.trim="data.internal_reference"
            disabled
          />

          <base-input
            col="12"
            rows="3"
            type="textarea"
            :placeholder="$t('PLACEHOLDERS.orderDescription')"
            v-model.trim="data.description"
            disabled
          />

          <base-input
            col="12"
            type="text"
            :placeholder="$t('PLACEHOLDERS.pickupAddress')"
            v-model.trim="data.pickup_address"
            disabled
          />

          <base-input
            col="12"
            type="text"
            :placeholder="$t('PLACEHOLDERS.dropoffAddress')"
            v-model.trim="data.dropoff_address"
            disabled
          />

          <div class="input_wrapper switch_wrapper my-5 col-6">
            <v-switch
              color="green"
              :label="$t('PLACEHOLDERS.needHelp')"
              v-model="data.need_help"
              hide-details
              disabled
            ></v-switch>
          </div>

          <div class="input_wrapper switch_wrapper my-5 col-6">
            <v-switch
              color="orange"
              :label="$t('PLACEHOLDERS.isFragile')"
              v-model="data.is_fragile"
              hide-details
              disabled
            ></v-switch>
          </div>

          <!-- Order Images -->
          <div class="col-12" v-if="data.images && data.images.length">
            <h6 class="mb-3">{{ $t("PLACEHOLDERS.orderImages") }}</h6>
            <div class="d-flex gap-3 flex-wrap">
              <div
                v-for="(image, index) in data.images"
                :key="index"
                class="image-preview"
              >
                <img :src="image" :alt="'Order Image ' + (index + 1)" />
              </div>
            </div>
          </div>

          <!-- Start:: Customer Information -->
          <div class="col-12 mb-4 mt-4">
            <h5>{{ $t("PLACEHOLDERS.customerInformation") }}</h5>
          </div>

          <div class="col-12 mb-3" v-if="data.user && data.user.avatar">
            <div class="text-center">
              <img
                :src="data.user.avatar"
                alt="Customer Avatar"
                class="avatar-image"
              />
            </div>
          </div>

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.clientName')"
            v-model="data.user.name"
            disabled
          />

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.clientPhone')"
            v-model="data.user.mobile"
            disabled
          />

          <base-input
            col="4"
            type="email"
            :placeholder="$t('PLACEHOLDERS.email')"
            v-model="data.user.email"
            disabled
          />

          <!-- Start:: Driver Information -->
          <div class="col-12 mb-4 mt-4" v-if="data.driver">
            <h5>{{ $t("PLACEHOLDERS.driverInformation") }}</h5>
          </div>

          <template v-if="data.driver">
            <div class="col-12 mb-3" v-if="data.driver.avatar">
              <div class="text-center">
                <img
                  :src="data.driver.avatar"
                  alt="Driver Avatar"
                  class="avatar-image"
                />
              </div>
            </div>

            <base-input
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.driverName')"
              v-model="data.driver.name"
              disabled
            />

            <base-input
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.driverPhone')"
              v-model="data.driver.mobile"
              disabled
            />

            <base-input
              col="4"
              type="text"
              :placeholder="$t('PLACEHOLDERS.driverRating')"
              v-model="data.driver.avg_rate"
              disabled
            />

            <base-input
              v-if="data.driver.vehicle"
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.vehicleName')"
              v-model="data.driver.vehicle.name"
              disabled
            />

            <div class="input_wrapper switch_wrapper my-5 col-6">
              <v-switch
                color="green"
                :label="
                  data.driver.is_active
                    ? $t('PLACEHOLDERS.active')
                    : $t('PLACEHOLDERS.notActive')
                "
                v-model="data.driver.is_active"
                hide-details
                disabled
              ></v-switch>
            </div>
          </template>

          <!-- Start:: Financial Information -->
          <div class="col-12 mb-4 mt-4">
            <h5>{{ $t("PLACEHOLDERS.financialInformation") }}</h5>
          </div>

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.mainPrice')"
            v-model="data.main_price"
            disabled
          />

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.discount')"
            v-model="data.discount"
            disabled
          />

          <base-input
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.finalPrice')"
            v-model="data.price"
            disabled
          />

          <base-input
            v-if="data.payment_data"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.paymentStatus')"
            v-model="data.payment_data.payment_status"
            disabled
          />

          <base-input
            v-if="data.payment_data"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.paymentMethod')"
            v-model="data.payment_data.payment_method"
            disabled
          />

          <base-input
            v-if="data.payment_data"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.gatewayReference')"
            v-model="data.payment_data.gateway_reference"
            disabled
          />

          <base-input
            v-if="data.payment_data"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.commissionRatio')"
            v-model="data.payment_data.commission_ratio"
            disabled
          />

          <base-input
            v-if="data.payment_data"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.commissionAmount')"
            v-model="data.payment_data.commission_amount"
            disabled
          />

          <base-input
            v-if="data.payment_data"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.driverProfit')"
            v-model="data.payment_data.provider_profit"
            disabled
          />aaa
          <base-input
            v-if="data.payment_data.fixed_tax"
            col="4"
            type="text"
            :placeholder="$t('PLACEHOLDERS.fixed_tax')"
            v-model="data.payment_data.fixed_tax"
            disabled
          />bbb
          <!-- Cancel Reason if canceled -->
          <div class="col-12 mb-4 mt-4" v-if="data.cancel_reason">
            <h5>{{ $t("PLACEHOLDERS.cancelInformation") }}</h5>
          </div>

          <base-input
            v-if="data.cancel_reason"
            col="12"
            rows="3"
            type="textarea"
            :placeholder="$t('PLACEHOLDERS.cancelReason')"
            v-model="data.cancel_reason"
            disabled
          />
        </div>
      </form>
    </div>
    <!-- END:: Single Step Form Content -->
  </div>
</template>

<script>
export default {
  name: "ShowOrder",

  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      // Start:: Data Collection To Send
      data: {
        id: null,
        order_number: null,
        created_at: null,
        status: null,
        cancel_reason: null,
        name: null,
        description: null,
        need_help: false,
        is_fragile: false,
        images: [],
        pickup_address: null,
        dropoff_address: null,
        main_price: null,
        discount: null,
        price: null,
        internal_reference: null,
        driver: {
          id: null,
          name: null,
          mobile: null,
          is_active: false,
          avatar: null,
          vehicle: {
            name: null,
            id: null,
          },
          avg_rate: null,
        },
        user: {
          id: null,
          name: null,
          mobile: null,
          email: null,
          avatar: null,
        },
        payment_data: {
          payment_status: null,
          main_price: null,
          discount: null,
          payment_method: null,
          gateway_reference: null,
          commission_ratio: null,
          fixed_tax:null.
          commission_amount: null,
          provider_profit: null,
        },
      },
      // End:: Data Collection To Send
    };
  },

  computed: {},

  methods: {
    // start show data
    async showOrder() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `orders/${this.$route.params.id}`,
        });

        const order = res.data.data?.Order;

        // Map order data
        this.data.id = order.id;
        this.data.order_number = order.order_number;
        this.data.created_at = order.created_at;
        this.data.status = order.status;
        this.data.cancel_reason = order.cancel_reason;
        this.data.name = order.name;
        this.data.description = order.description;
        this.data.need_help = order.need_help;
        this.data.is_fragile = order.is_fragile;
        this.data.images = order.images || [];
        this.data.pickup_address = order.pickup_address;
        this.data.dropoff_address = order.dropoff_address;
        this.data.main_price = order.main_price;
        this.data.discount = order.discount;
        this.data.price = order.price;
        this.data.internal_reference = order.internal_reference;

        // Map driver data
        if (order.driver) {
          this.data.driver = {
            id: order.driver.id,
            name: order.driver.name,
            mobile: order.driver.mobile,
            is_active: order.driver.is_active,
            avatar: order.driver.avatar,
            vehicle: order.driver.vehicle || { name: null, id: null },
            avg_rate: order.driver.avg_rate,
          };
        }

        // Map user data
        if (order.user) {
          this.data.user = {
            id: order.user.id,
            name: order.user.name,
            mobile: order.user.mobile,
            email: order.user.email,
            avatar: order.user.avatar,
          };
        }

        // Map payment data
        if (order.payment_data) {
          this.data.payment_data = {
            payment_status: order.payment_data.payment_status,
            main_price: order.payment_data.main_price,
            discount: order.payment_data.discount,
            payment_method: order.payment_data.payment_method,
            gateway_reference: order.payment_data.gateway_reference,
            commission_ratio: order.payment_data.commission_ratio,
            commission_amount: order.payment_data.commission_amount,
            provider_profit: order.payment_data.provider_profit,
          };
        }
      } catch (error) {
        this.loading = false;
        console.log(error?.response?.data?.message);
      }
    },
    // end show data
  },

  created() {
    // Start:: Fire Methods
    this.showOrder();
    // End:: Fire Methods
  },
};
</script>

<style scoped>
.avatar-image {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid #007f5f;
}

.image-preview {
  width: 150px;
  height: 150px;
  border-radius: 8px;
  overflow: hidden;
  border: 2px solid #ddd;
}

.image-preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
