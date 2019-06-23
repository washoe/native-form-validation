<template>
  <form novalidate @submit.prevent="handleSubmit">
    <fieldset>
      <!-- required -->
      <div class="field">
        <input
          type="text"
          name="requiredText"
          id="requiredText"
          required
          @blur="handleBlur"
          :aria-invalid="messages.requiredText"
        >
        <label for="requiredText">This text is required</label>
        <p class="messages" v-if="messages.requiredText">There is an error</p>
      </div>
      <!-- max and min length. Here we use pattern validation, as minlength/maxlength will constrain the input. -->
      <div class="field">
        <input
          type="text"
          name="minmaxText"
          id="minmaxText"
          pattern="^(.{2,8})$"
          @blur="handleBlur"
          :aria-invalid="messages.minmaxText"
        >
        <label for="minmaxText">This text must be between 2-8 characters</label>
        <p class="messages" v-if="messages.minmaxText">There is an error</p>
      </div>
      <!-- required + max and min length. Combination of the above -->
      <div class="field">
        <input
          type="text"
          name="requiredMinmaxText"
          id="requiredMinmaxText"
          pattern="^(.{2,8})$"
          required
          @blur="handleBlur"
          :aria-invalid="messages.requiredMinmaxText"
        >
        <label
          for="requiredMinmaxText"
        >This text is required and must also be between 2-8 characters</label>
        <p class="messages" v-if="messages.requiredMinmaxText">There is an error</p>
      </div>
      <!-- max and min number. -->
      <div class="field">
        <input
          type="number"
          name="minmaxNumber"
          id="minmaxNumber"
          pattern="^[0-9]$"
          min="2"
          max="8"
          @blur="handleBlur"
          :aria-invalid="messages.minmaxNumber"
        >
        <label for="minmaxNumber">This number must be between 2-8</label>
        <p class="messages" v-if="messages.minmaxNumber">There is an error</p>
      </div>
      <!-- date - must not be in the past. We need to set a sensible-ish max as well. Without this, using the down arrow on the year field will give 275760 (on chrome)-->
      <div class="field">
        <input
          type="date"
          name="futureDate"
          id="futureDate"
          :min="today"
          max="2099-12-31"
          @blur="handleBlur"
          :aria-invalid="messages.futureDate"
        >
        <label for="futureDate">This date must be today or later</label>
        <p class="messages" v-if="messages.futureDate">There is an error</p>
      </div>
      <!-- required + max and min length. This time in a textarea -->
      <!--
The textarea appears not to support the pattern attribute in Chrome, so we cannot directly use that approach to validating length.

However we can use the setCustomValidity method, which we will call on change. See https://developer.mozilla.org/en-US/docs/Web/API/Constraint_validation
      -->
      <div class="field">
        <textarea
          name="requiredMinmaxTextArea"
          id="requiredMinmaxTextArea"
          rows="10"
          pattern="^(.{0,10})$"
          required
          @blur="handleBlur"
          @change="handleChange"
          :aria-invalid="messages.requiredMinmaxTextArea"
        ></textarea>
        <label for="requiredMinmaxText">This text is required and has max 10 characters</label>
        <p class="messages" v-if="messages.requiredMinmaxTextArea">There is an error</p>
      </div>
      <!-- required select -->
      <div class="field">
        <select
          name="requiredSelect"
          id="requiredSelect"
          required
          @blur="handleBlur"
          :aria-invalid="messages.requiredSelect"
        >
          <option></option>
          <option value="1">Option 1</option>
          <option value="2">Option 2</option>
        </select>
        <label for="requiredSelect">This select is required</label>
        <p class="messages" v-if="messages.requiredSelect">There is an error</p>
      </div>
    </fieldset>
    <button type="submit">Submit</button>
    <p v-if="formIsValid">No errors reported</p>
    <p class="messages" v-if="!formIsValid">This form has some errors</p>
  </form>
</template>

<script>
/**
 * Desired behaviour:
 * - No validation errors shown initially
 * - Report a field's validity on blur
 * - Report all fields' validity on submit
 * - Do not prevent user from entering invalid data (no constraint)
 * - Form submission will be blocked if any fields are invalid
 * - All error messages to be displayed below the field
 * ref: https://www.w3.org/WAI/WCAG21/working-examples/aria-invalid-data-format/
 */
export default {
  name: "Form",
  data: () => {
    return {
      messages: {},
      today: new Date().toISOString().slice(0, 10),
      formIsValid: true
    };
  },
  methods: {
    setFormIsValid: function() {
      this.$nextTick(() => {
        this.formIsValid = !document.querySelectorAll("form [aria-invalid]")
          .length;
      });
    },
    customValidate: function(element) {
      const patternValid =
        !element.attributes.pattern ||
        new RegExp(element.attributes.pattern.value).test(element.value); // true if valid
      element.setCustomValidity(patternValid ? "" : "Too damn long!");
    },
    handleChange: function(event) {
      this.customValidate(event.target);
    },
    runValidation: function(elements) {
      this.messages = (elements || []).reduce((result, element) => {
        return {
          ...result,
          [element.name]: !element.checkValidity() ? element.validity : null
        };
      }, this.messages);
      this.setFormIsValid();
    },
    handleBlur: function(event) {
      this.runValidation([event.target]);
    },
    handleSubmit: function(event) {
      [...event.target.querySelectorAll("textarea")].forEach(
        this.customValidate
      );
      const formInputElements = [
        ...event.target.querySelectorAll("input, textarea, select")
      ];
      this.runValidation(formInputElements);
    }
  }
};
</script>

<style scoped>
[aria-invalid] {
  border: 1px solid red;
}
.field {
  display: flex;
  flex-direction: column;
}
.field:not(:last-child) {
  margin-bottom: 1em;
}
.field .label {
  order: 1;
}
.field input,
.field textarea,
.field select {
  order: 2;
}
.field .messages {
  order: 3;
}
.field :required ~ label:after {
  content: " *";
  color: red;
}

.field :invalid ~ .messages {
  color: red;
}
</style>
