<template>
  <h2 class="m-2 text-center text-secondary">OTC trading tool</h2>

  <div class="form-group form-floating m-2">
    <select
      v-model.number="inputMarket"
      @change="updateMarketPrice()"
      id="inputMarket"
      class="form-select form-select-lg"
    >
      <option v-for="(market, id) in markets" :value="id">
        {{ market.name }} - {{ exchanges[market.exchange].name }}
      </option>
    </select>
    <label for="inputMarket">Market</label>
  </div>

  <div class="form-group form-floating m-2">
    <select
      v-model.number="inputFee"
      id="inputFee"
      class="form-select form-select-lg"
    >
      <option v-for="fee in fees" :value="fee">{{ fee }}%</option>
    </select>
    <label for="inputFee">Fee</label>
  </div>

  <div class="form-group form-floating m-2">
    <select
      v-model="inputConstant"
      id="inputConstant"
      class="form-select form-select-lg"
    >
      <option value="fiat">Fiat (EUR, USD, ...)</option>
      <option value="crypto">Crypto (BTC, ETH, USDT ...)</option>
    </select>
    <label for="inputConstant">Trade constant</label>
  </div>

  <div class="form-group form-floating m-2">
    <input
      type="text"
      id="inputAmount"
      v-model.number="inputAmount"
      type="number"
      class="form-control form-control-lg border-success"
    /><label for="inputAmount">Trade amount</label>
  </div>

  <div class="m-2 p-3 bg-light">
    <span class="side_label">Market price</span>
    <span class="price"
      ><span class="num" id="price">{{
        marketPrice.toFixed(markets[inputMarket].priceDecimal)
      }}</span></span
    >
    <br />
    <span class="side_label"></span>
    <span id="price_with_fee"
      >with <span class="num_small">{{ inputFee }}%</span> fee
      <span class="num_small">{{
        marketPriceWithFee().toFixed(markets[inputMarket].priceDecimal)
      }}</span></span
    >
    <br /><br />
    <span class="side_label">Trade fiat</span>
    <span class="num_big" id="trade_fiat">{{
      (inputAmount
        ? inputConstant == 'fiat'
          ? inputAmount
          : +inputAmount * +marketPriceWithFee()
        : 0
      ).toFixed(2)
    }}</span>
    <br /><br />
    <span class="side_label">Trade crypto</span>
    <span class="num_big" id="trade_crypto">{{
      (inputAmount
        ? inputConstant == 'crypto'
          ? inputAmount
          : +inputAmount / +marketPriceWithFee()
        : 0
      ).toFixed(markets[inputMarket].amountDecimal)
    }}</span>
    <br /><br />
    <span class="side_label"></span>
    <input
      type="checkbox"
      class="btn-check"
      id="btn-check-outlined"
      autocomplete="off"
      v-model="statusHold"
    />
    <label class="btn btn-lg btn-outline-info" for="btn-check-outlined">{{
      statusHold ? 'RELEASE' : 'FREEZE'
    }}</label>
  </div>
</template>

<script>
export default {
  name: 'OTCTool',
  props: {},
  data() {
    return {
      inputAmount: 0,
      inputMarket: 0,
      inputFee: 0,
      inputConstant: 'fiat',
      marketPrice: 0,
      statusHold: false,
      priceUpdateIntervalID: '',
      markets: [
        {
          name: 'BTC/EUR',
          exchange: 'binance',
          ticker: 'BTCEUR',
          priceDecimal: 2,
          amountDecimal: 8,
        },
        {
          name: 'BTC/EUR',
          exchange: 'bitstamp',
          ticker: 'btceur',
          priceDecimal: 2,
          amountDecimal: 8,
        },
        {
          name: 'ETH/EUR',
          exchange: 'bitstamp',
          ticker: 'etheur',
          priceDecimal: 2,
          amountDecimal: 8,
        },
        {
          name: 'USDT/EUR',
          exchange: 'bitstamp',
          ticker: 'usdteur',
          priceDecimal: 5,
          amountDecimal: 2,
        },
      ],
      fees: [
        10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0, -1, -2, -3, -4, -5, -6, -7, -8, -9,
        -10,
      ],
      exchanges: {
        bitstamp: {
          name: 'bitstamp.net',
          api_url: 'https://www.bitstamp.net/api/v2/ticker/{ticker}/',
          bidKey: 'bid',
          askKey: 'ask',
        },
        binance: {
          name: 'binance.com',
          api_url:
            'https://api.binance.com/api/v3/ticker/bookTicker?symbol={ticker}',
          bidKey: 'bidPrice',
          askKey: 'askPrice',
        },
      },
    };
  },
  methods: {
    updateMarketPrice: async function () {
      let exchange = this.exchanges[this.markets[this.inputMarket].exchange];
      const response = await fetch(
        exchange.api_url.replace(
          '{ticker}',
          this.markets[this.inputMarket].ticker
        )
      );
      const data = await response.json();
      this.marketPrice = (+data[exchange.askKey] + +data[exchange.bidKey]) / 2;
      console.log(this.marketPrice);
    },
    marketPriceWithFee: function () {
      return +this.marketPrice * ((100 + +this.inputFee) / 100);
    },
  },
  mounted() {
    this.updateMarketPrice();
    this.priceUpdateIntervalID = setInterval(() => {
      if (!this.statusHold) {
        this.updateMarketPrice();
      }
    }, 3000);
  },
  beforeDestroy() {
    clearInterval(this.priceUpdateIntervalID);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.price {
  display: inline-block;
  padding: 0 0 0 10px;
  line-height: 95px;
  height: 95px;
}
#inputAmount {
  background: #eeffed;
}
.num,
.num_small,
.num_big {
  font-family: monospace;
  font-weight: bold;
  font-size: 35px;
}
.num_small {
  font-size: 25px;
}
.num_big {
  font-size: 50px;
}

.side_label {
  display: inline-block;
  width: 150px;
  text-align: right;
  padding: 0 20px 0 0;
}
</style>
