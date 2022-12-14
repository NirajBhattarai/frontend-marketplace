<template>
  <div class="section position-absolute">
    <PreventUnload
      :when="transactionStatus === STATUS.DEPOSITING && !!this.transactionHash"
      message="Please stay on this page until the deposit transaction is confirmed on Ethereum!"
    />
    <div
      v-bsl="show"
      class="modal receive-modal-wrapper"
      :class="{ show: show }"
    >
      <div
        class="modal-dialog w-sm-100 align-self-center"
        role="document"
      >
        <div class="box deposit-box">
          <div class="box-header justify-content-center">
            <div
              class="font-heading-medium font-semibold align-self-center w-100 text-center"
            >
              {{ $t("deposit.title") }}
            </div>
            <span
              class="left-arrow align-self-center float-right cursor-pointer"
              :class="{
                'disabled-cursor':
                  transactionStatus === STATUS.DEPOSITING && transactionHash,
              }"
              @click="onCancel()"
            >
              <svg-sprite-icon
                name="close"
                class="close align-self-center float-left"
              />
            </span>
          </div>
          <div class="box-body">
            <div class="container p-0">
              <div
                class="col-12 ps-x-40 ps-b-20 ps-t-16 container-wrapper font-body-small"
              >
                <div class="d-flex ps-b-20 ps-t-18">
                  <img
                    class="cate-icon align-self-center ms-r-8"
                    :src="category.img_url"
                  >
                  <div class="text-white align-self-center">
                    {{ selectedTokens.length || 0 }} {{ $t("nftSelected") }}
                  </div>
                </div>
                <div class="container card-list hide-scrollbar d-flex p-0">
                  <div
                    v-for="token in selectedTokens"
                    :key="token.token_id"
                    class="token-img d-flex ms-x-6 ps-4 justify-content-center"
                  >
                    <img
                      class="align-self-center"
                      :src="token.img_url"
                      :alt="token.name"
                    >
                  </div>
                </div>
              </div>
            </div>
            <div class="container">
              <div class="row ps-x-40 ps-b-60">
                <div class="col-12 p-0">
                  <div class="mark-wrapper check float-left">
                    <img
                      v-if="transactionStatus === STATUS.INITIATING"
                      src="~/static/img/yellow-check.svg"
                      alt="In Progress"
                    >
                    <img
                      v-if="transactionStatus >= STATUS.INITIATED"
                      src="~/static/img/green-check.svg"
                      alt="Green Check"
                    >
                  </div>
                  <div class="float-left body-medium ps-2 ms-l-12 d-flex">
                    <span
                      v-if="transactionStatus === STATUS.INITIATING"
                      class="ps-t-0"
                    >{{ $t("deposit.steps.preInit") }}</span>
                    <span
                      v-if="transactionStatus >= STATUS.INITIATED"
                      class="ps-t-2"
                    >{{ $t("deposit.steps.init") }}</span>
                  </div>
                </div>
                <div class="col-12 p-0">
                  <div
                    class="float-left process-msg font-caption text-gray ms-l-12 ms-b-2 ps-l-24"
                  />
                </div>
                <div class="col-12 p-0">
                  <div
                    class="mark-wrapper float-left"
                    :class="{ check: transactionStatus >= STATUS.INITIATED }"
                  >
                    <img
                      v-if="transactionStatus === STATUS.INITIATED"
                      class="ms-l-2"
                      src="~/static/img/information-check.svg"
                      alt="Green Check"
                    >
                    <img
                      v-if="transactionStatus === STATUS.DEPOSITING"
                      src="~/static/img/yellow-check.svg"
                      alt="Green Check"
                    >
                    <img
                      v-if="transactionStatus >= STATUS.DEPOSITED"
                      src="~/static/img/green-check.svg"
                      alt="Green Check"
                    >
                  </div>
                  <div class="float-left body-medium ps-2 ps-t-0 ms-l-12">
                    {{ $t("deposit.steps.deposit") }}
                  </div>
                </div>
                <div class="col-12 p-0">
                  <div
                    class="float-left process-msg font-caption text-gray ms-l-12 ms-b-2 ps-l-24"
                  >
                    <div class="ps-b-8">
                      <span v-if="transactionStatus === STATUS.INITIATED">
                        {{ $t("deposit.process.preDeposit") }}
                      </span>
                      <span v-if="transactionStatus === STATUS.DEPOSITING">{{
                        $t("deposit.process.depositing")
                      }}</span>
                      <a
                        v-if="
                          transactionStatus >= STATUS.DEPOSITING &&
                            transactionHash
                        "
                        :href="explorerURL"
                        target="_blank"
                        rel="noopener noreferrer"
                        :title="transactionHash"
                      >{{ $t("viewOnEtherscan") }}</a>
                    </div>
                    <div
                      v-if="
                        transactionStatus === STATUS.DEPOSITING &&
                          transactionHash
                      "
                      class="ps-b-16 text-red font-semibold"
                    >
                      {{ $t("preventUserDepositModalClose") }}
                    </div>
                  </div>
                </div>
                <div class="col-12 p-0">
                  <div
                    class="mark-wrapper float-left"
                    :class="{ check: transactionStatus >= STATUS.DEPOSITED }"
                  >
                    <img
                      v-if="transactionStatus >= STATUS.DEPOSITED"
                      src="~/static/img/green-check.svg"
                      alt="Green Check"
                    >
                  </div>
                  <div class="float-left body-medium ps-2 ms-l-12">
                    {{ this.$t("deposit.steps.finished") }}
                  </div>
                </div>
                <div class="col-12 p-0">
                  <div
                    class="float-left font-caption text-gray ms-l-12 ms-b-2 ps-l-24"
                  >
                    <span
                      v-if="transactionStatus >= STATUS.DEPOSITED"
                      class="ps-l-2"
                    >
                      {{ $t("deposit.process.deposited") }}
                    </span>
                  </div>
                </div>
              </div>
              <div
                v-if="error"
                class="row"
              >
                <div class="col-12 ps-x-32 ps-b-12 text-center text-red">
                  <div class="font-body-small text-red text-center mx-auto">
                    Please try again
                  </div>
                </div>
              </div>
              <div class="row p-0">
                <div class="col-12 p-0 d-flex justify-content-space-between">
                  <button-loader
                    v-if="transactionStatus >= STATUS.INITIATED && !isDeposited"
                    class="w-100"
                    :classes="['btn py-4 btn-pay no-top-border-radius']"
                    :block="true"
                    :text="'Confirm Deposit'"
                    color="primary"
                    :loadingText="'Confirming deposit'"
                    :click="deposit"
                    :loading="isLoading"
                  />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div
      class="modal-backdrop"
      :class="{ show: show }"
    />
  </div>
</template>

<script>
import Vue from 'vue'
import Component from 'nuxt-class-component'
import { mapGetters } from 'vuex'
import app from '~/plugins/app'
import getAxios from '~/plugins/axios'
import { getWalletProvider } from '~/plugins/helpers/providers'

import PreventUnload from 'vue-prevent-unload'
const MaticPOSClient = require('@maticnetwork/maticjs').MaticPOSClient

const STATUS = {
  INITIATING: 0,
  INITIATED: 1,
  DEPOSITING: 2,
  DEPOSITED: 3,
}

@Component({
  props: {
    show: {
      type: Boolean,
      required: false,
      default: true,
    },
    isApproving: {
      type: Boolean,
      required: false,
      default: true,
    },
    selectedTokens: {
      type: Array,
      required: true,
    },
    category: {
      type: Object,
      required: true,
    },
    refreshBalance: {
      type: Function,
      required: true,
    },
    cancel: {
      type: Function,
      required: true,
    },
  },
  components: {
    PreventUnload,
  },
  methods: {},
  computed: {
    ...mapGetters('account', ['account']),
    ...mapGetters('network', ['networks', 'networkMeta']),
    ...mapGetters('page', ['selectedCategory']),
  },
})
export default class DepositConfirmationModal extends Vue {
  STATUS = STATUS;

  selectToken = false;
  isLoading = false;
  error = null;
  transactionHash = null;
  isDeposited = false;

  async mounted() {}

  // Getter
  get transactionStatus() {
    if (this.isApproving) {
      return STATUS.INITIATING
    } else if (!this.isApproving && !this.isLoading && !this.isDeposited) {
      return STATUS.INITIATED
    } else if (!this.isApproving && this.isLoading && !this.isDeposited) {
      return STATUS.DEPOSITING
    } else if (this.isDeposited) {
      return STATUS.DEPOSITED
    }
  }

  get parentNetwork() {
    return this.networks.main
  }

  get childNetwork() {
    return this.networks.matic
  }

  get networkId() {
    return this.parentNetwork.chainId
  }

  get explorerURL() {
    if (app.uiconfig.mainExplorer) {
      return `${app.uiconfig.mainExplorer}tx/${this.transactionHash}`
    }
    return null
  }

  get selectedTokenIds() {
    const tokenIds = []
    if (this.selectedTokens && this.selectedTokens.length > 0) {
      this.selectedTokens.forEach((token) => tokenIds.push(token.token_id))
    }
    return tokenIds
  }

  // Actions
  getMaticPOS() {
    const maticProvider = getWalletProvider({
      networks: this.networks,
      primaryProvider: 'child',
    })
    const parentProvider = getWalletProvider({
      networks: this.networks,
      primaryProvider: 'main',
    })

    return new MaticPOSClient({
      network: app.uiconfig.matic.deployment.network,
      version: app.uiconfig.matic.deployment.version,
      parentProvider,
      maticProvider,
      posRootChainManager: this.networkMeta.Main.POSContracts
        .RootChainManagerProxy,
      posERC20Predicate: this.networkMeta.Main.POSContracts.ERC20PredicateProxy,
      posERC721Predicate: this.networkMeta.Main.POSContracts
        .ERC721PredicateProxy,
    })
  }

  async deposit() {
    if (this.isLoading || this.isApproving) {
      return
    }

    try {
      this.isLoading = true

      const maticPoS = this.getMaticPOS()
      const ERC721 = this.selectedTokens[0].contract
      const token_ids = this.selectedTokenIds

      const txHash = await maticPoS.depositBatchERC721ForUser(
        ERC721,
        this.account.address,
        token_ids,
        {
          from: this.account.address,
          onTransactionHash: (txHash) => {
            this.transactionHash = txHash
          },
        },
      )
      if (txHash) {
        await this.handleDeposit(txHash, token_ids, this.category.id)
        this.isLoading = false
        this.isDeposited = true
      }
    } catch (error) {
      console.log(error)
      this.isLoading = false
      this.error = error.message
    }
  }

  async handleDeposit(txHash, token_ids, category_id) {
    console.log('Deposit transaction', txHash)
    try {
      const data = {
        txhash: this.transactionHash,
        token_array: token_ids,
        category_id: category_id,
        type: 'DEPOSIT',
      }
      const res = await getAxios().post('assetmigrate', data)
    } catch (error) {
      console.log(error)
    }
    this.refreshBalance()
  }

  onCancel() {
    if (this.transactionStatus === STATUS.DEPOSITING && this.transactionHash) {
      {
        return
      }
      this.cancel()
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~assets/css/theme/_theme";

.deposit-box {
  width: 446px;
}

@media (max-width: 446px) {
  .deposit-box {
    width: 100%;
  }
}
.cate-icon {
  height: 24px;
  width: 24px;
  border-radius: 50%;
}
.label {
  color: dark-color("500");
}
.amount {
  color: dark-color("700");
}

.bottom-border {
  border-bottom: 1px solid light-color("500");
}

.mark-wrapper:not(.check) {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 1px solid light-color("400");
  margin: 2px;
}
.mark-wrapper.check {
  width: 24px;
  height: 24px;
}

.process-msg {
  border-left: 1px solid light-color("400");
  min-height: 18px;
}
.text-gray {
  color: dark-color("300");
}
.text-red {
  color: red-color("600");
}
.disabled-cursor {
  cursor: default !important;
}

.btn-pay {
  border-radius: 12px;
}

.container-wrapper {
  background: dark-color("300");
  text-align: left;
  margin-bottom: 24px;

  .card-list {
    overflow-x: auto;
    -ms-overflow-style: none;
    scrollbar-width: none;
  }

  &::-webkit-scrollbar {
    display: none;
  }
}

.token-img {
  min-width: 76px;
  min-height: 76px;
  width: 76px;
  height: 76px;
  background: white;
  border-radius: 10px;

  img {
    max-height: 76px;
    max-width: 76px;
    height: 100%;
    width: auto;
  }
}
</style>
