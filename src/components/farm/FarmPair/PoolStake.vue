<template>
  <transition name="horizontal">
    <div v-if="isOpen" class="flex flex-col justify-between h-full w-full absolute">
      <div class="flex flex-col space-y-2">
        <div class="flex items-center space-x-2 mx-2">
          <i class="las la-donate text-xl text-oswapGreen"></i>
          <p class="text-sm dark:text-gray-300">Deposit your LP Tokens</p>
        </div>
        <div class="flex flex-col dark:bg-oswapDark-gray bg-gray-100 rounded-2xl">
          <div class="flex flex-1 shadow-lg rounded-2xl">
            <InputWithValidation :input="amount" :errors="errors" @catchInput="inputAmount" :rounded="'rounded-xl'" :errorTop="'pt-10'">
              <p class="text-xs z-20 right-1 absolute bg-gray-200 dark:bg-gray-700 rounded-lg p-2">{{pool.pair}}</p>
            </InputWithValidation>
          </div>
          <div class="grid grid-cols-3 gap-3 pt-3 pb-3 px-3">
            <div class="flex col-span-2 h-12 space-x-2">
              <div class="flex items-start h-full">
                <i class="las la-coins text-xl text-oswapGreen"></i>
              </div>
              <div class="flex flex-col h-full justify-between">
                <p class="text-xs text-oswapBlue-light">LP Tokens Available</p>
                <p class="text-lg dark:text-gray-400">{{parseFloat(this.getEthUnits(this.maxAmount)).toFixed(5)}}</p>
              </div>
            </div>
            <div class="flex items-center justify-end group-scope">
              <div @click="setMax()" class="flex items-center group-scope-hover:bg-oswapGreen text-oswapGreen-dark dark:text-oswapGreen border border-oswapGreen-dark dark:border-oswapGreen cursor-pointer p-3 rounded-xl space-x-2">
                <i class="las la-wallet text-xl dark:group-scope-hover:text-oswapDark-gray group-scope-hover:text-gray-100"></i>
                <p class="dark:group-scope-hover:text-oswapDark-gray group-scope-hover:text-gray-100">MAX</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="flex justify-between items-center mb-0.5 h-9">
        <!-- back to Stats -->
        <div @click="setPool()" class="flex h-9 items-center space-x-2 ss:space-x-0 sm:space-x-2 md:space-x-0 xl:space-x-2 rounded-full group bg-gray-100 hover:bg-gray-200 dark:bg-oswapDark-gray dark:hover:bg-gray-900 pr-3 ss:pr-0 sm:pr-3 md:pr-0 xl:pr-3 cursor-pointer">
          <i class="las la-arrow-left text-lg p-2 text-gray-500 dark:text-oswapBlue-light rounded-full bg-gray-100 dark:bg-oswapDark-gray group-hover:bg-oswapGreen dark:group-hover:text-oswapDark-gray group-hover:text-gray-100 border-2 border-gray-200 dark:border-gray-700"></i>
          <p class="ss:hidden sm:block md:hidden xl:block text-sm text-gray-500 dark:text-oswapBlue-light">back</p>
        </div>
        
        <div class="flex items-center space-x-2">
          <div class="flex items-center w-28 h-full relative">
            <farmApprove :key="amount" :amount="this.amount" :pool="pool" @tellStake="setStakeState" />
          </div>
          <div class="flex items-center w-28 h-full relative">
            <farmStake :key="amount" :amount="this.amount" :pool="pool" :btnState="btnStake" @updateData="updateData"/>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  import openswap from "@/shared/openswap.js";

  import InputWithValidation from "@/components/InputWithValidation";
  import farmStake from "@/components/farm/FarmPair/farmStake";
  import farmApprove from "@/components/farm/FarmPair/farmApprove";

  export default {
    name: 'PoolStake',
    components: {
      InputWithValidation,
      farmApprove,
      farmStake
    },
    mixins: [openswap],
    props: {
      pool: Object,
      isOpen: Boolean,
      maxAmount: Object
    },
    data() {
      return {
        errors: {},
        amount: '0.1',
        btnStake: 'disabled'
      }
    },
    methods: {
      setMax() {
        this.amount = this.getEthUnits(this.maxAmount);
      },
      updateData(){
        this.$emit("updateData")
      },
      setPool() {
        // reset Input
        this.amount = '0.1'
        // reset errors
        this.errors = {}
        this.$emit('setPool', 'stats')
      },
      inputAmount(value) {
        this.amount = value;
        // Checking if the input is in the right format.
        // parseFloat seems to behave like this regex rule.
        if (!value.match(/^\d*\.?\d*$/)) {
          this.errors['format'] = 'Invalid format! e.g: 12345.678';
        } else {
          delete this.errors['format'];
        }
        if (value == '') {
          this.errors['blank'] = 'Amount can\'t be blank';
        } else {
          delete this.errors['blank'];
        }
        if (parseFloat(value) > parseFloat(this.getEthUnits(this.maxAmount))) {
          this.errors['exceed'] = 'Your input exceeds the amount you have available!';
        } else {
          delete this.errors['exceed'];
        }
      },
      setStakeState(value) {
        // check if it has amount
        if (this.maxAmount > 0) {
          this.btnStake = value
        } else {
          this.btnStake = 'disabled'
        }
      }
    }
  }
</script>