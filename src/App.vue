<template>
    <CardForm
      card-form-class="bordered"
      primary-color="#4ae00f"
      typography-color="#000"
      submit-button-text="Pay"
      :on-submit="clickHandler"
      @input-card-number="updateCardNumber"
      @input-card-name="updateCardName"
      @input-card-month="updateCardMonth"
      @input-card-year="updateCardYear"
      @input-card-cvv="updateCardCvv"
      card-background="blue"
    />
</template>

<script>
import CardForm from '@/components/CardForm'
export default {
  name: 'app',
  components: {
    CardForm
  },
  methods: {
    updateCardNumber (val) {
      console.log('Card Number', val)
    },
    updateCardName (val) {
      console.log('Card Name', val)
    },
    updateCardMonth (val) {
      console.log('Card Month', val)
    },
    updateCardYear (val) {
      console.log('Card Year', val)
    },
    updateCardCvv (val) {
      console.log('Card Cvv', val)
    },
    clickHandler () {
      console.log('Click from parent')
    }
  }
}
</script>
<style lang="scss">
  .bordered {
    border: 1px solid black;
  }
</style>
