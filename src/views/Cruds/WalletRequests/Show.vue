<template>
  <div class="show_single_content_wrapper">
    <!-- Start:: Main Section -->
    <main>
      <!-- Start:: Title -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("PLACEHOLDERS.requestDetails") }}</h5>
        </div>
      </div>
      <!-- End:: Title -->

      <!-- Start:: Content Card -->
      <div class="content_card_wrapper">
        <v-form>
          <div class="row">
            <!-- ========== Start:: Settlement Request Information Section ========== -->
            <div class="col-12">
              <h6 class="section_title">
                {{ $t("PLACEHOLDERS.requestInformation") }}
              </h6>
            </div>

            <!-- Start:: Order Serial Number -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.orderNum')"
              v-model.trim="data.serial_number"
              disabled
            />
            <!-- End:: Order Serial Number -->

            <!-- Start:: Request Date -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.requestDate')"
              v-model.trim="data.created_at"
              disabled
            />
            <!-- End:: Request Date -->

            <!-- Start:: Requested Amount -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.requestedAmount')"
              v-model.trim="data.amount"
              disabled
            >
              <template v-slot:suffix>
                <span>{{ $t("PLACEHOLDERS.riyal") }}</span>
              </template>
            </base-input>
            <!-- End:: Requested Amount -->

            <!-- Start:: Request Status -->
            <div class="col-md-6">
              <div class="input_wrapper">
                <label for="status">{{
                  $t("PLACEHOLDERS.requestStatus")
                }}</label>
                <v-chip
                  :color="getStatusColor(data.status_enum || data.status)"
                  class="ma-2"
                >
                  {{ getStatusTranslation(data.status_enum || data.status) }}
                </v-chip>
              </div>
            </div>
            <!-- End:: Request Status -->

            <!-- Start:: Rejection Reason (if rejected) -->
            <base-input
              v-if="(data.status_enum || data.status) === 'rejected'"
              col="12"
              type="textarea"
              rows="3"
              :placeholder="$t('PLACEHOLDERS.rejectionReason')"
              v-model.trim="data.rejection_reason"
              disabled
            />
            <!-- End:: Rejection Reason -->

            <!-- ========== Start:: Delivery Agent Information Section ========== -->
            <div class="col-12 mt-5">
              <h6 class="section_title">
                {{ $t("PLACEHOLDERS.driverInformation") }}
              </h6>
            </div>

            <!-- Start:: Agent Avatar -->
            <div class="col-12 d-flex justify-content-center mb-4">
              <div class="avatar_wrapper">
                <img
                  v-if="data.user?.avatar"
                  :src="data.user.avatar"
                  :alt="data.user.name"
                  class="avatar"
                />
                <div v-else class="avatar_placeholder">
                  <i class="fal fa-user"></i>
                </div>
              </div>
            </div>
            <!-- End:: Agent Avatar -->

            <!-- Start:: Agent Name -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.driverName')"
              v-model.trim="data.user.name"
              disabled
            />
            <!-- End:: Agent Name -->

            <!-- Start:: Agent Mobile -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.mobileNumber')"
              v-model.trim="data.user.mobile"
              disabled
            />
            <!-- End:: Agent Mobile -->

            <!-- Start:: Agent Email -->
            <base-input
              col="6"
              type="email"
              :placeholder="$t('PLACEHOLDERS.email')"
              v-model.trim="data.user.email"
              disabled
            />
            <!-- End:: Agent Email -->

            <!-- Start:: Current Wallet Balance -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.currentWalletBalance')"
              v-model.trim="data.user.current_user_balance"
              disabled
            >
              <template v-slot:suffix>
                <span>{{ $t("PLACEHOLDERS.riyal") }}</span>
              </template>
            </base-input>
            <!-- End:: Current Wallet Balance -->

            <!-- ========== Start:: Bank Account Information Section ========== -->
            <div class="col-12 mt-5">
              <h6 class="section_title">
                {{ $t("PLACEHOLDERS.bankingInformation") }}
              </h6>
            </div>

            <!-- Start:: Bank Name -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.bankName')"
              v-model.trim="data.bank_name"
              disabled
            />
            <!-- End:: Bank Name -->

            <!-- Start:: Account Holder Name -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.accountHolderName')"
              v-model.trim="data.account_holder_name"
              disabled
            />
            <!-- End:: Account Holder Name -->

            <!-- Start:: Account Number -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.accountNumber')"
              v-model.trim="data.account_number"
              disabled
            />
            <!-- End:: Account Number -->

            <!-- Start:: IBAN Number -->
            <base-input
              col="6"
              type="text"
              :placeholder="$t('PLACEHOLDERS.ibanNumber')"
              v-model.trim="data.iban_number"
              disabled
            />
            <!-- End:: IBAN Number -->
          </div>
        </v-form>
      </div>
      <!-- End:: Content Card -->
    </main>
    <!-- End:: Main Section -->
  </div>
</template>

<script>
export default {
  name: "ShowWalletRequest",

  data() {
    return {
      // Start:: Data
      data: {
        id: null,
        serial_number: null,
        created_at: null,
        amount: null,
        status: null,
        status_enum: null,
        rejection_reason: null,
        bank_name: null,
        account_holder_name: null,
        account_number: null,
        iban_number: null,
        user: {
          name: null,
          mobile: null,
          email: null,
          avatar: null,
          current_user_balance: null,
        },
      },
      // End:: Data
    };
  },

  methods: {
    // Start:: Get Settlement Request Data
    async getRequestData() {
      try {
        const res = await this.$axios({
          method: "GET",
          url: `withdraw-requests/${this.$route.params.id}`,
        });

        // Map Response Data
        this.data.id = res.data.data.id;
        this.data.serial_number = res.data.data.serial_number;
        this.data.created_at = res.data.data.created_at;
        this.data.amount = res.data.data.amount;
        this.data.status = res.data.data.status;
        this.data.status_enum = res.data.data.status_enum;
        this.data.rejection_reason = res.data.data.rejection_reason || "-";
        this.data.bank_name = res.data.data.bank_name || "-";
        this.data.account_holder_name =
          res.data.data.account_holder_name || "-";
        this.data.account_number = res.data.data.account_number || "-";
        this.data.iban_number = res.data.data.iban_number || "-";

        // Map User Data
        if (res.data.data.user) {
          this.data.user.name = res.data.data.user.name || "-";
          this.data.user.mobile = res.data.data.user.mobile || "-";
          this.data.user.email = res.data.data.user.email || "-";
          this.data.user.avatar = res.data.data.user.avatar;
          this.data.user.current_user_balance =
            res.data.data.user.current_user_balance || 0;
        }
      } catch (error) {
        console.log(error.response?.data?.message || error.message);
      }
    },
    // End:: Get Settlement Request Data

    // Get status translation
    getStatusTranslation(status) {
      const statusMap = {
        new: this.$t("STATUS.pending"),
        accepted: this.$t("STATUS.approved"),
        rejected: this.$t("STATUS.rejected"),
      };
      return statusMap[status] || status;
    },

    // Get status color
    getStatusColor(status) {
      const colorMap = {
        new: "orange",
        accepted: "green",
        rejected: "red",
      };
      return colorMap[status] || "grey";
    },
  },

  created() {
    // Start:: Fire Methods
    this.getRequestData();
    // End:: Fire Methods
  },
};
</script>

<style lang="scss" scoped>
.section_title {
  font-weight: bold;
  color: #333;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #e0e0e0;
}

.avatar_wrapper {
  display: flex;
  justify-content: center;
  align-items: center;

  .avatar {
    width: 120px;
    height: 120px;
    border-radius: 50%;
    object-fit: cover;
    border: 3px solid #e0e0e0;
  }

  .avatar_placeholder {
    width: 120px;
    height: 120px;
    border-radius: 50%;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
    align-items: center;
    border: 3px solid #e0e0e0;

    i {
      font-size: 50px;
      color: #999;
    }
  }
}

.input_wrapper {
  padding: 10px;

  label {
    display: block;
    margin-bottom: 8px;
    font-weight: 500;
    color: #666;
  }
}
</style>
