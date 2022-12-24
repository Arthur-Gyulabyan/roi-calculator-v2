<script lang="ts">
import type { Ref } from 'vue'
import { ref, toRefs, watch } from 'vue'

interface Props {
  APY: number;
  SOL_USD: number;
}

interface Data {
  total: { sol: string, usd: string },
  perDay: { sol: string, usd: string },
  perMonth: { sol: string, usd: string },
  perYear: { sol: string, usd: string },
}

export default {
  props: ['APY', 'SOL_USD'],
  setup(props: Props, context: any) {
    const {APY, SOL_USD} = toRefs(props);

    const periodMap = {day: 365, month: 12, year: 20}

    const amount: Ref<string> = ref('');
    const number: Ref<string> = ref('');
    const period: Ref<'day' | 'month' | 'year'> = ref('day');
    const data: Ref<Data> = ref({
      total: {sol: '-', usd: '-'},
      perDay: {sol: '-', usd: '-'},
      perMonth: {sol: '-', usd: '-'},
      perYear: {sol: '-', usd: '-'},
    });
    const numberMax: Ref<number> = ref(periodMap[period.value]);
    const amountError: Ref<boolean> = ref(false);
    const numberError: Ref<boolean> = ref(false);

    const not = (value: any) => !value;

    const solToUsd = (value: number): number => value * SOL_USD.value;

    const numberInRange = (value: string): string => {
      const minValue = 1;
      const maxValue = periodMap[period.value];
      return Math.min(maxValue, Math.max(minValue, +value)).toString()
    }

    const toReadableMode = (value: string): string => {
      return isNaN(+value) ? value.toString() : (+value).toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }

    const addUsdSymbol = (value: string): string => {
      return value && value !== '-' ? `$ ${toReadableMode(value)}` : value
    }

    const keyDownAmountHandler = (event: KeyboardEvent) => {
      amountError.value = false;
      const {key, keyCode, ctrlKey, metaKey} = event;
      const value = (event.target as HTMLInputElement).value;

      if (
          not(ctrlKey) &&
          not(metaKey) &&
          not(/^[0-9\.]+$/.test(key)) &&
          not([8, 9, 37, 38, 39, 40].includes(keyCode)) ||
          (value.includes('.') && event.key === '.')
      ) {
        event.preventDefault()
      }
    }

    const keyUpNumberHandler = (event: KeyboardEvent) => {
      numberError.value = false;
      const value = (event.target as HTMLInputElement).value;
      const inRangeValue = numberInRange(value);
      if (value !== inRangeValue) {
        number.value = inRangeValue;
      }
    }

    const validateFields = () => {
      amountError.value = !amount.value;
      numberError.value = !number.value;
    }

    const calculate = () => {
      validateFields();
      if (amountError.value || numberError.value) {
        return;
      }

      const periodMap = {day: +number.value / 365, month: +number.value / 12, year: +number.value}
      const p = periodMap[period.value];

      const totalSol = (+amount.value * ((1 + APY.value / 100) ** p)).toString();
      const total = {sol: totalSol, usd: (solToUsd(+totalSol)).toString()}

      const map = {day: +number.value, month: +number.value * 365 / 12, year: +number.value * 365}
      const perDaySol = ((+totalSol - +amount.value) / (map[period.value])).toString();
      const perMonthSol = (+perDaySol * 365 / 12).toString();
      const perYearSol = (+perDaySol * 365).toString();
      data.value = {
        total,
        perDay: {sol: perDaySol, usd: (solToUsd(+perDaySol)).toString()},
        perMonth: {sol: perMonthSol, usd: (solToUsd(+perMonthSol)).toString()},
        perYear: {sol: perYearSol, usd: (solToUsd(+perYearSol)).toString()},
      }
    }

    watch(period, () => {
      const inRangeValue = numberInRange(number.value);
      if (number.value !== inRangeValue) {
        number.value = inRangeValue;
      }
    })

    return {
      APY,
      SOL_USD,
      data,
      amount,
      number,
      numberMax,
      period,
      amountError,
      numberError,
      calculate,
      addUsdSymbol,
      toReadableMode,
      keyDownAmountHandler,
      keyUpNumberHandler,
    }
  }
}
</script>

<template>
  <div class="calculator">
    <div class="calculator__header">
      <p class="calculator__title">Solana APY Interest Calculator</p>
    </div>
    <div class="calculator__body">
      <div class="calculator__form">
        <div class="rate mb-12">
          <p class="rate__title">Current APY Interest Rate</p>
          <p class="rate__value">≈{{ APY }}%</p>
        </div>

        <div class="control mb-12">
          <label class="control__label">SOL Investment Amount</label>
          <input
              type="text"
              class="control__field"
              :class="{'field-error': amountError}"
              v-model="amount"
              @keydown="keyDownAmountHandler"
          >
        </div>

        <div class="control mb-12">
          <label class="control__label">Investment Tenure</label>
          <div class="d-flex gap-6">
            <input
                type="number"
                class="control__field control__field_number"
                :class="{'field-error': numberError}"
                v-model="number"
                @keyup="keyUpNumberHandler"
            >
            <select class="control__field flex-grow-1" v-model="period">
              <option value="day">Day</option>
              <option value="month">Month</option>
              <option value="year">Year</option>
            </select>
          </div>
        </div>

        <button class="app-button app-button_primary app-button_block" @click="calculate">
          CALCULATE
        </button>
      </div>
      <div class="calculator__info">
        <table class="app-table app-table_block">
          <tr>
            <th>Total Return SOL</th>
            <th>{{ toReadableMode(data.total.sol) }}</th>
            <th>{{ addUsdSymbol(data.total.usd) }}</th>
          </tr>
          <tr>
            <td>Interest Per Day</td>
            <td>{{ toReadableMode(data.perDay.sol) }}</td>
            <th>{{ addUsdSymbol(data.perDay.usd) }}</th>
          </tr>
          <tr>
            <td>Interest Per Month</td>
            <td>{{ toReadableMode(data.perMonth.sol) }}</td>
            <th>{{ addUsdSymbol(data.perMonth.usd) }}</th>
          </tr>
          <tr>
            <td>Interest Per Year</td>
            <td>{{ toReadableMode(data.perYear.sol) }}</td>
            <th>{{ addUsdSymbol(data.perYear.usd) }}</th>
          </tr>
          <tr>
            <td></td>
            <td></td>
            <td></td>
          </tr>
        </table>
        <div>
          <div class="how-to-calculate">
            <p class="how-to-calculate__title">How to calculate APY</p>
            <p class="how-to-calculate__value">Current Price (USD) 1 SOL = {{ SOL_USD }} $</p>
          </div>
          <div>
            <p class="description">
              Annual percentage yield (APY) is the effective annual rate, or real rate, of return of an investment if
              the interest earned each period is compounded. APY considers the effects of compounding, since advertised
              rates are typically the rates of return for simple interest.
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.how-to-calculate {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}

.how-to-calculate__title {
  font-size: 11px;
  font-weight: 400;
  color: var(--app-black-1);
}

.how-to-calculate__value {
  font-weight: 400;
  font-size: 10px;
  line-height: 12px;
  color: var(--app-gray-2);
}

.description {
  font-size: 10px;
  line-height: 15px;
  font-weight: 300;
  color: var(--app-black-1);
}
</style>
