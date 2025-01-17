<template>
  <div class="fee-selector btn-group btn-group-toggle" data-toggle="buttons">
    <label class="btn btn-option btn-option-lg"
        v-for="name in ['slow', 'average', 'fast']" :key="name"
        :class="{ active: (name === value)}"
        v-tooltip="{ content: getTooltip(name) }"
        v-on:click="$emit('input', name)">
        <input type="radio" name="fee" autocomplete="off" :checked="name === value"> {{name}}
    </label>
  </div>
</template>

<script>
import { getTxFee } from '@/utils/fees'
import BN from 'bignumber.js'
import { prettyFiatBalance } from '@/utils/coinFormatter'
import { getNativeAsset } from '@/utils/asset'
import cryptoassets from '@/utils/cryptoassets'
import { chains } from '@liquality/cryptoassets'

export default {
  props: [
    'asset',
    'value',
    'fees',
    'txTypes',
    'fiatRates'
  ],
  methods: {
    getTooltip (name) {
      let content = '<div class="text-right">'
      if (this.fees[name].wait) {
        content += `${this.fees[name].wait} sec<br />`
      }

      const nativeAsset = getNativeAsset(this.asset)
      if (this.txTypes) {
        const total = this.txTypes.reduce((accum, tx) => {
          return accum.plus(getTxFee(this.asset, tx, this.fees[name].fee))
        }, BN(0))
        const totalFiat = prettyFiatBalance(total, this.fiatRates[nativeAsset])
        content += `${BN(total).dp(6)} ${nativeAsset}`
        content += `<br />${totalFiat} USD`
      } else {
        const chainId = cryptoassets[this.asset].chain
        const { unit } = chains[chainId].fees
        content += `${this.fees[name].fee} ${unit}`
      }

      return `${content}</div>`
    }
  }
}
</script>

<style lang="scss">
.fee-selector.btn-group label.btn {
  text-transform: capitalize;
}
</style>
