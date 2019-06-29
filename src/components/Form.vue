<template>
  <form novalidate @submit.prevent="handleSubmit" @reset="handleReset">
    <fieldset>
      <!-- required -->
      <div class="field">
        <input
          type="text"
          name="requiredText"
          id="requiredText"
          required
          @blur="handleBlur"
          :aria-invalid="reportedMessages.requiredText"
        >
        <label for="requiredText">This text is required</label>
        <p class="messages" v-if="reportedMessages.requiredText">{{reportedMessages.requiredText}}</p>
      </div>
      <!-- max and min length. Here we use pattern validation, as minlength/maxlength will constrain the input. -->
      <div class="field">
        <input
          type="text"
          name="minmaxText"
          id="minmaxText"
          pattern="^(.{2,8})$"
          @blur="handleBlur"
          :aria-invalid="reportedMessages.minmaxText"
        >
        <label for="minmaxText">This text must be between 2-8 characters</label>
        <p class="messages" v-if="reportedMessages.minmaxText">{{reportedMessages.minmaxText}}</p>
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
          :aria-invalid="reportedMessages.requiredMinmaxText"
        >
        <label
          for="requiredMinmaxText"
        >This text is required and must also be between 2-8 characters</label>
        <p
          class="messages"
          v-if="reportedMessages.requiredMinmaxText"
        >{{reportedMessages.requiredMinmaxText}}</p>
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
          :aria-invalid="reportedMessages.minmaxNumber"
        >
        <label for="minmaxNumber">This number must be between 2-8</label>
        <p class="messages" v-if="reportedMessages.minmaxNumber">{{reportedMessages.minmaxNumber}}</p>
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
          :aria-invalid="reportedMessages.futureDate"
        >
        <label for="futureDate">This date must be today or later, but is not required</label>
        <p class="messages" v-if="reportedMessages.futureDate">{{reportedMessages.futureDate}}</p>
      </div>
      <div class="field">
        <input
          type="date"
          name="requiredDate"
          id="requiredDate"
          required
          @blur="handleBlur"
          :aria-invalid="reportedMessages.requiredDate"
        >
        <label for="requiredDate">This date is required</label>
        <p class="messages" v-if="reportedMessages.requiredDate">{{reportedMessages.requiredDate}}</p>
      </div>
      <div class="field">
        <input
          type="time"
          name="requiredTime"
          id="requiredTime"
          required
          @blur="handleBlur"
          :aria-invalid="reportedMessages.requiredTime"
        >
        <label for="requiredTime">This time is required</label>
        <p class="messages" v-if="reportedMessages.requiredTime">{{reportedMessages.requiredTime}}</p>
      </div>
      <!-- required + max and min length. This time in a textarea -->
      <!-- TODO
The textarea appears not to support the pattern attribute in Chrome, so we cannot directly use that approach to validating length. We will need to do something more complex.
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
          :aria-invalid="reportedMessages.requiredMinmaxTextArea"
        ></textarea>
        <label for="requiredMinmaxText">This text is required and has max 10 characters</label>
        <p
          class="messages"
          v-if="reportedMessages.requiredMinmaxTextArea"
        >{{reportedMessages.requiredMinmaxTextArea}}</p>
      </div>
      <!-- required select -->
      <div class="field">
        <select
          name="requiredSelect"
          id="requiredSelect"
          required
          @blur="handleBlur"
          :aria-invalid="reportedMessages.requiredSelect"
        >
          <option></option>
          <option value="1">Option 1</option>
          <option value="2">Option 2</option>
        </select>
        <label for="requiredSelect">This select is required</label>
        <p
          class="messages"
          v-if="reportedMessages.requiredSelect"
        >{{reportedMessages.requiredSelect}}</p>
      </div>
    </fieldset>
    <fieldset>
      <button type="submit">Submit</button>
      <button type="reset">Reset</button>
      <p v-if="formIsValid">No errors reported</p>
      <p class="messages form-invalid" v-if="!formIsValid">This form has some errors</p>
    </fieldset>
  </form>
</template>

<script>
/**
 * Desired behaviour:
 * - No validation errors shown initially
 * - Report a field's validity on blur
 * - Report all fields' validity on submit
 * - Show multiple messages simultaneously
 * - Do not prevent user from entering invalid data (no constraint)
 * - Replace default validation messages with custom messages
 * - Form submission will be blocked if any fields are invalid
 * - All error messages to be displayed below the field
 * ref: https://www.w3.org/WAI/WCAG21/working-examples/aria-invalid-data-format/
 */

/**
 * Creates validation message for each element
 * We use the underlying ValidationState as keys. This allows us to let the browser to do the validation where it can.
 * We can also combine messages if a field has multiple validation errors
 */
const setValidationMessages = elements => {
  // For now, the same map will be applied everywhere
  const messageMap = {
    badInput: () => "Please enter a valid value",
    patternMismatch: () => "Does not match regex pattern",
    rangeOverflow: () => "Too high",
    rangeUnderflow: element =>
      element.type === "date" ? "Cannot be in the past" : "Too low",
    stepMismatch: () => "Does not resolve to the given step size",
    tooLong: () => "Too long",
    tooShort: () => "Too short",
    typeMismatch: () => "Cannot be resolved to the required type",
    valueMissing: () => "Cannot be blank"
  };
  return elements.map(element => {
    const validationMessage = Object.entries(messageMap)
      .filter(entry => element.validity[entry[0]])
      .map(entry => entry[1](element));
    element.setCustomValidity(validationMessage.join("|"));
    return element;
  });
};

export default {
  name: "Form",
  data: () => {
    return {
      reportedMessages: {},
      today: new Date().toISOString().slice(0, 10)
    };
  },
  computed: {
    formIsValid: function() {
      return !Object.values(this.reportedMessages).length;
    },
    fieldsToValidate: function() {
      return this.$el && typeof this.$el.querySelectorAll === "function"
        ? [...this.$el.querySelectorAll("[name]")].filter(
            element => element.willValidate
          )
        : [];
    }
  },
  methods: {
    reportMessages: function(elements) {
      this.reportedMessages = setValidationMessages(elements).reduce(
        (result, element) => {
          result[element.name] = element.validationMessage
            ? element.validationMessage.split("|")
            : undefined;
          return result;
        },
        { ...this.reportedMessages }
      );
    },
    handleChange: function(event) {},
    handleBlur: function(event) {
      this.reportMessages([event.target]);
    },
    handleSubmit: function() {
      this.reportMessages(this.fieldsToValidate);
    },
    handleReset: function() {
      this.reportedMessages = {};
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

.field :invalid ~ .messages,
.messages.form-invalid {
  color: red;
}
</style>
