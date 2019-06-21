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
    <!-- max and min length -->
    <div class="field">
      <label for="minmaxText">This text must be between 2-8 characters</label>
      <input type="text"
      name="minmaxText"
      id="minmaxText"
      minlength="2"
      maxlength="8"
      @blur="handleBlur"
      :aria-invalid="messages.minmaxText">
      <p class="messages" v-if="messages.minmaxText">There is an error</p>
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
      messages: {}
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