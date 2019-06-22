<template>
<form novalidate @submit.prevent="handleSubmit">
  <fieldset>
    <!-- required -->
    <div class="field">
      <label for="requiredText">This text is required</label>
      <input type="text"
      name="requiredText"
      id="requiredText"
      required
      @blur="handleBlur"
      :aria-invalid="messages.requiredText">
      <p class="messages" v-if="messages.requiredText">There is an error</p>
    </div>
    <!-- max and min length. Here we use pattern validation, as minlength/maxlength will constrain the input. -->
    <div class="field">
      <label for="minmaxText">This text must be between 2-8 characters</label>
      <input type="text"
      name="minmaxText"
      id="minmaxText"
      pattern="^(.{2,8})$"
      @blur="handleBlur"
      :aria-invalid="messages.minmaxText">
      <p class="messages" v-if="messages.minmaxText">There is an error</p>
    </div>
        <!-- required + max and min length. Combination of the above -->
    <div class="field">
      <label for="requiredMinmaxText">This text is required and must also be between 2-8 characters</label>
      <input type="text"
      name="requiredMinmaxText"
      id="requiredMinmaxText"
      pattern="^(.{2,8})$"
      required
      @blur="handleBlur"
      :aria-invalid="messages.requiredMinmaxText">
      <p class="messages" v-if="messages.requiredMinmaxText">There is an error</p>
    </div>
    <!-- max and min number. -->
    <div class="field">
      <label for="minmaxNumber">This number must be between 2-8</label>
      <input type="number"
      name="minmaxNumber"
      id="minmaxNumber"
      pattern="^[0-9]$"
      min="2"
      max="8"
      @blur="handleBlur"
      :aria-invalid="messages.minmaxNumber">
      <p class="messages" v-if="messages.minmaxNumber">There is an error</p>
    </div>
    <!-- date - must not be in the past. We need to set a sensible-ish max as well. Without this, using the down arrow on the year field will give 275760 (on chrome)-->
    <div class="field">
      <label for="futureDate">This date must be today or later</label>
      <input type="date"
      name="futureDate"
      id="futureDate"
      :min="today"
      mmax="2099-12-31"
      @blur="handleBlur"
      :aria-invalid="messages.futureDate">
      <p class="messages" v-if="messages.futureDate">There is an error</p>
    </div>
  </fieldset>
  <button type="submit">Submit</button>
</form>
</template>

<script>
/**
 * Desired behaviour:
 * - No validation errors shown initially
 * - Check a field's validity on blur
 * - check all fields' validity on submit
 * - do not prevent user from entering invalid data (no constraint)
 * - form submission will be blocked if any fields are invalid
 * - All error messages to be displayed below the field
 * ref: https://www.w3.org/WAI/WCAG21/working-examples/aria-invalid-data-format/
 */
export default {
  name: 'Form',
  data: ()=> {
    return {
      messages: {},
      today: new Date().toISOString().slice(0, 10)
    };
  },
  methods: {
    runValidation: function(elements) {
      this.messages = (elements || []).reduce((result, element) => {
        return {...result, [element.name]: !element.checkValidity() ? element.validity : null};
      }, this.messages);
    },
    handleBlur: function(event) {
      this.runValidation([event.target]);
    },
    handleSubmit: function(event) {
      const formInputElements = [...event.target.querySelectorAll('input')];
      this.runValidation(formInputElements);
    }
  }
}
</script>

<style scoped>
[aria-invalid] {
  border: 1px solid red;
}
.messages {
  color: red;
}
</style>
