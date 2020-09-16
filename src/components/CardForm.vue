<template>
  <div class="card-form" :style=cssProps>
    <div class="card-list">
      <Card
        :fields="fields"
        :cardName="cardName"
        :cardNumber="cardNumber"
        :cardNumberNotMask="cardNumberNotMask"
        :cardMonth="cardMonth"
        :cardYear="cardYear"
        :cardCvv="cardCvv"
        :cardPostcode="cardPostcode"
        :isCardNumberMasked="isCardNumberMasked"
        :cardBackground="cardBackground"
      />
    </div>
    <div :class="['card-form__inner', cardFormClass]">
      <div class="card-input">
        <label for="cardNumber"
          class="card-input__label"
          :class="{
            '-error': !isCardNumberValid
          }"
        >
          Card Number
        </label>
        <input
          type="tel"
          :id="fields.cardNumber"
          @input="changeNumber"
          @focus="focusCardNumber"
          @blur="blurCardNumber"
          class="card-input__input"
          :class="{
            'card-input__error': !isCardNumberValid
          }"
          :value="cardNumber"
          :maxlength="cardNumberMaxLength"
          data-card-field
          autocomplete="off"
        />
        <button
          class="card-input__eye"
          :class="{ '-active' : !isCardNumberMasked }"
          title="Show/Hide card number"
          tabindex="-1"
          :disabled="cardNumber === ''"
          @click="toggleMask"
        ></button>
      </div>
      <div class="card-input">
        <label for="cardName" class="card-input__label">Card Name</label>
        <input
          type="text"
          :id="fields.cardName"
          v-letter-only
          @input="changeName"
          class="card-input__input"
          :value="cardName"
          data-card-field
          autocomplete="off"
        />
      </div>
      <div class="card-form__row">
        <div class="card-form__col">
          <div class="card-form__group">
            <label for="cardMonth" class="card-input__label">Expiration Date</label>
            <select
              class="card-input__input -select"
              :id="fields.cardMonth"
              v-model="cardMonth"
              @change="changeMonth"
              data-card-field
            >
              <option value disabled selected>Month</option>
              <option
                v-bind:value="n"
                v-for="n in 12"
                v-bind:disabled="n < minCardMonth"
                v-bind:key="n"
              >{{generateMonthValue(n)}}</option>
            </select>
            <select
              class="card-input__input -select"
              :id="fields.cardYear"
              v-model="cardYear"
              @change="changeYear"
              data-card-field
            >
              <option value disabled selected>Year</option>
              <option
                v-bind:value="$index + minCardYear"
                v-for="(n, $index) in 12"
                v-bind:key="n"
              >{{$index + minCardYear}}</option>
            </select>
          </div>
        </div>
      </div>
      <div class="card-form__row">
        <div class="card-form__col">
          <div class="card-form__group">
            <div class="card-form__item">
              <label for="cardPostcode" class="card-input__label">Postcode</label>
              <input
                type="text"
                class="card-input__input"
                maxlength="7"
                :value="cardPostcode"
                @input="changePostcode"
                data-card-field
                autocomplete="off"
              />
            </div>
            <div class="card-form__item">
              <label for="cardCvv" class="card-input__label">CVV</label>
              <input
                type="tel"
                class="card-input__input"
                v-number-only
                :id="fields.cardCvv"
                maxlength="4"
                :value="cardCvv"
                @input="changeCvv"
                data-card-field
                autocomplete="off"
              />
            </div>
          </div>
        </div>
      </div>

      <button class="card-form__button"
        :disabled="!cardFormValid || loading"
        @click="onSubmit"
      >
        <slot v-if="loading" name="loader">
            <span>Loading ...</span>
        </slot>
        <span v-else>{{submitButtonText}}</span>
      </button>
    </div>
  </div>
</template>

<script>
import Card from '@/components/Card'
export default {
  name: 'CardForm',
  directives: {
    'number-only': {
      bind (el) {
        function checkValue (event) {
          event.target.value = event.target.value.replace(/[^0-9]/g, '')
          if (event.charCode >= 48 && event.charCode <= 57) {
            return true
          }
          event.preventDefault()
        }
        el.addEventListener('keypress', checkValue)
      }
    },
    'letter-only': {
      bind (el) {
        function checkValue (event) {
          if (event.charCode >= 48 && event.charCode <= 57) {
            event.preventDefault()
          }
          return true
        }
        el.addEventListener('keypress', checkValue)
      }
    }
  },
  props: {
    onSubmit: {
      type: Function,
      default: () => console.log('Submit clicked')
    },
    cardFormClass: String,
    primaryColor: {
      type: String,
      default: () => '#2364d2' // Blue,
    },
    typographyColor: {
      type: String,
      default: () => '#000' // Black,
    },
    submitButtonText: {
      type: String,
      default: () => 'Submit'
    },
    cardBackground: {
      type: String,
      default: () => `blue`
    },
    loading: {
      type: Boolean,
      default: false
    }
  },
  components: {
    Card
  },
  data () {
    return {
      cardName: '',
      cardNumber: '',
      cardMonth: null,
      cardYear: null,
      cardCvv: '',
      cardPostcode: '',
      fields: {
        cardNumber: 'v-card-number',
        cardName: 'v-card-name',
        cardMonth: 'v-card-month',
        cardYear: 'v-card-year',
        cardCvv: 'v-card-cvv',
        cardPostcode: 'v-card-postcode'
      },
      cardNumberNotMask: '',
      minCardYear: new Date().getFullYear(),
      isCardNumberMasked: true,
      mainCardNumber: this.cardNumber,
      cardNumberMaxLength: 19
    }
  },
  computed: {
    minCardMonth () {
      if (this.cardYear === this.minCardYear) return new Date().getMonth() + 1
      return 1
    },
    cssProps () {
      return {
        '--primary-color': this.primaryColor,
        '--typography-color': this.typographyColor,
        '--card-background': this.cardBackground
      }
    },
    isCardNumberValid () {
      let number = this.cardNumberNotMask
      if (number === '') return true
      if (!number) return false

      // The Luhn Algorithm. It's so pretty.
      let nCheck = 0
      let bEven = false
      let value = number.replace(/\D/g, '')

      for (let n = value.length - 1; n >= 0; n--) {
        let cDigit = value.charAt(n)
        let nDigit = parseInt(cDigit, 10)

        if (bEven && (nDigit *= 2) > 9) nDigit -= 9

        nCheck += nDigit
        bEven = !bEven
      }

      let a = (nCheck % 10) === 0
      return a
    },
    cardFormValid () {
      return this.cardNumber !== '' &&
        this.cardName !== '' &&
        this.cardMonth &&
        this.cardYear &&
        this.cardCvv !== '' &&
        this.cardPostcode !== '' &&
        this.isCardNumberValid
    }
  },
  mounted () {
    this.maskCardNumber()
  },
  methods: {
    generateMonthValue (n) {
      return n < 10 ? `0${n}` : n
    },
    changeName (e) {
      this.cardName = e.target.value
      this.$emit('input-card-name', this.cardName)
    },
    changeNumber (e) {
      this.cardNumber = e.target.value
      let value = this.cardNumber.replace(/\D/g, '')
      // american express, 15 digits
      if ((/^3[47]\d{0,13}$/).test(value)) {
        this.cardNumber = value.replace(/(\d{4})/, '$1 ').replace(/(\d{4}) (\d{6})/, '$1 $2 ')
        this.cardNumberMaxLength = 17
      } else if ((/^3(?:0[0-5]|[68]\d)\d{0,11}$/).test(value)) { // diner's club, 14 digits
        this.cardNumber = value.replace(/(\d{4})/, '$1 ').replace(/(\d{4}) (\d{6})/, '$1 $2 ')
        this.cardNumberMaxLength = 16
      } else if ((/^\d{0,16}$/).test(value)) { // regular cc number, 16 digits
        this.cardNumber = value.replace(/(\d{4})/, '$1 ').replace(/(\d{4}) (\d{4})/, '$1 $2 ').replace(/(\d{4}) (\d{4}) (\d{4})/, '$1 $2 $3 ')
        this.cardNumberMaxLength = 19
      }

      if (e.inputType === 'deleteContentBackward') {
        let lastChar = this.cardNumber.substring(
          this.cardNumber.length,
          this.cardNumber.length - 1
        )
        if (lastChar === ' ') {
          this.cardNumber = this.cardNumber
            .substring(0, this.cardNumber.length - 1)
        }
      }
      this.$emit('input-card-number', this.cardNumber)
    },
    changeMonth () {
      this.$emit('input-card-month', this.cardMonth)
    },
    changeYear () {
      this.$emit('input-card-year', this.cardYear)
    },
    changeCvv (e) {
      this.cardCvv = e.target.value
      this.$emit('input-card-cvv', this.cardCvv)
    },
    changePostcode (e) {
      this.cardPostcode = e.target.value.toUpperCase()
      this.$emit('input-card-postcode', this.cardPostcode)
    },
    blurCardNumber () {
      if (this.isCardNumberMasked) {
        this.maskCardNumber()
      }
    },
    maskCardNumber () {
      this.cardNumberNotMask = this.cardNumber
      this.mainCardNumber = this.cardNumber
      let arr = this.cardNumber.split('')
      arr.forEach((element, index) => {
        if (index > 4 && index < 14 && element.trim() !== '') {
          arr[index] = '*'
        }
      })
      this.cardNumber = arr.join('')
    },
    unMaskCardNumber () {
      this.cardNumber = this.mainCardNumber
    },
    focusCardNumber () {
      this.unMaskCardNumber()
    },
    toggleMask () {
      this.isCardNumberMasked = !this.isCardNumberMasked
      if (this.isCardNumberMasked) {
        this.maskCardNumber()
      } else {
        this.unMaskCardNumber()
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.card-form {
  max-width: 570px;
  margin: auto;
  width: 100%;

  & * {
    box-sizing: border-box;
    &:focus {
      outline: none;
    }
  }

  @media screen and (max-width: 576px) {
    margin: 0 auto;
  }

  &__inner {
    background: #fff;
    border-radius: 10px;
    padding: 35px;
    padding-top: 180px;

    @media screen and (max-width: 480px) {
      padding: 25px;
      padding-top: 165px;
    }
    @media screen and (max-width: 360px) {
      padding: 15px;
      padding-top: 165px;
    }
  }

  &__row {
    display: flex;
    align-items: flex-start;
    @media screen and (max-width: 480px) {
      flex-wrap: wrap;
    }
  }

  &__col {
    flex: auto;
    margin-right: 35px;

    &:last-child {
      margin-right: 0;
    }

    @media screen and (max-width: 480px) {
      margin-right: 0;
      flex: unset;
      width: 100%;
      margin-bottom: 20px;

      &:last-child {
        margin-bottom: 0;
      }
    }

    &.-cvv {
      max-width: 150px;
      @media screen and (max-width: 480px) {
        max-width: initial;
      }
    }
  }

  &__group {
    margin-bottom: 20px;
    display: flex;
    align-items: flex-start;
    flex-wrap: wrap;

    .card-input__input, .card-form__item {
      flex: 1;
      margin-right: 15px;

      &:last-child {
        margin-right: 0;
      }
    }

  }

  button[disabled] {
    background-color: #dddddd;
  }

  &__button {
    width: 100%;
    height: 55px;
    border: none;
    border-radius: 5px;
    font-size: 22px;
    font-weight: 500;
    font-family: "Source Sans Pro", sans-serif;
    color: #fff;
    margin-top: 20px;
    cursor: not-allowed;

    @media screen and (max-width: 480px) {
      margin-top: 10px;
    }


    &:not([disabled]) {
      background: var(--primary-color);
      cursor: pointer;
    }
  }
}

.card-list {
  margin-bottom: -130px;

  @media screen and (max-width: 480px) {
    margin-bottom: -120px;
  }
}

.card-input {
  margin-bottom: 20px;
  position: relative;

  &__label {
    font-size: 14px;
    margin-bottom: 5px;
    font-weight: 500;
    color: var(--typography-color);
    width: 100%;
    display: block;
    user-select: none;
    &.-error {
      color: red;
    }
  }
  &__input {
    width: 100%;
    height: 50px;
    border-radius: 5px;
    box-shadow: none;
    border: 1px solid #ced6e0;
    transition: all 0.3s ease-in-out;
    font-size: 18px;
    padding: 5px 15px;
    background: none;
    color: #1a3b5d;
    font-family: "Source Sans Pro", sans-serif;

    &:hover,
    &:focus {
      border-color: var(--primary-color);
    }

    &:focus {
      box-shadow: 0px 10px 20px -13px rgba(32, 56, 117, 0.35);
    }
    &.-select {
      -webkit-appearance: none;
      background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAeCAYAAABuUU38AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAUxJREFUeNrM1sEJwkAQBdCsngXPHsQO9O5FS7AAMVYgdqAd2IGCDWgFnryLFQiCZ8EGnJUNimiyM/tnk4HNEAg/8y6ZmMRVqz9eUJvRaSbvutCZ347bXVJy/ZnvTmdJ862Me+hAbZCTs6GHpyUi1tTSvPnqTpoWZPUa7W7ncT3vK4h4zVejy8QzM3WhVUO8ykI6jOxoGA4ig3BLHcNFSCGqGAkig2yqgpEiMsjSfY9LxYQg7L6r0X6wS29YJiYQYecemY+wHrXD1+bklGhpAhBDeu/JfIVGxaAQ9sb8CI+CQSJ+QmJg0Ii/EE2MBiIXooHRQhRCkBhNhBcEhLkwf05ZCG8ICCOpk0MULmvDSY2M8UawIRExLIQIEgHDRoghihgRIgiigBEjgiFATBACAgFgghEwSAAGgoBCBBgYAg5hYKAIFYgHBo6w9RRgAFfy160QuV8NAAAAAElFTkSuQmCC');
      background-size: 12px;
      background-position: 90% center;
      background-repeat: no-repeat;
      padding-right: 30px;
    }
  }
  &__error {
    border-color: red;
    &:hover {
      border-color: red;
    }
  }
  &__eye {
    display: inline-flex;
    position: absolute;
    width: 1em;
    height: 1em;
    font-size: 24px;
    border-radius: 50%;
    top: 35px;
    right: 10px;
    opacity: .75;
    color: #8c9cae;
    cursor: pointer;
    padding: 0;
    background: none;
    display: inline-flex;
    border: 2px solid currentColor;
    box-shadow: none;
    transition: all .3s ease-in-out;

    &:before {
      content: '';
      position: absolute;
      background: white;
      width: 0.35em;
      height: 0.35em;
      top: 6px;
      left: 6px;
      z-index: 2;
      border-radius: 50%;
      transform: scale(.1);
      opacity: 0;
      transition: all .3s ease-in-out;
      transition-delay: .1s;
    }

    &:after {
      content: '';
      position: absolute;
      top: 3px;
      left: 3px;
      background: currentColor;
      width: 0.6em;
      height: 0.6em;
      border-radius: 50%;
      transform: scale(.1);
      opacity: 0;
      transition: all .3s ease-in-out;
    }

    &:hover:not(:disabled), &.-active:not(:disabled) {
      color: var(--primary-color);
      opacity: 1;
    }

    &.-active {
      &::before, &::after {
        transform: scale(1);
        opacity: 1;
      }
    }

    &:disabled {
      cursor: not-allowed;
      opacity: .4;
    }
  }
}
.slide-fade-up-enter-active {
  transition: all 0.25s ease-in-out;
  transition-delay: 0.1s;
  position: relative;
}
.slide-fade-up-leave-active {
  transition: all 0.25s ease-in-out;
  position: absolute;
}
.slide-fade-up-enter {
  opacity: 0;
  transform: translateY(15px);
  pointer-events: none;
}
.slide-fade-up-leave-to {
  opacity: 0;
  transform: translateY(-15px);
  pointer-events: none;
}

.slide-fade-right-enter-active {
  transition: all 0.25s ease-in-out;
  transition-delay: 0.1s;
  position: relative;
}
.slide-fade-right-leave-active {
  transition: all 0.25s ease-in-out;
  position: absolute;
}
.slide-fade-right-enter {
  opacity: 0;
  transform: translateX(10px) rotate(45deg);
  pointer-events: none;
}
.slide-fade-right-leave-to {
  opacity: 0;
  transform: translateX(-10px) rotate(45deg);
  pointer-events: none;
}
</style>
