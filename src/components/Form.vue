<template>
  <form novalidate @submit.prevent="handleSubmit" @reset="handleReset">
    <fieldset>
      <legend>Text Inputs</legend>
      <!-- required -->
      <div class="field">
        <input
          type="text"
          name="requiredText"
          id="requiredText"
          required
          @blur="doValidation"
          :aria-invalid="reportedMessages.requiredText"
        />
        <label for="requiredText">This text is required</label>
        <p class="errors">{{ reportedMessages.requiredText }}</p>
      </div>
      <!-- max and min length. Here we use pattern validation, as minlength/maxlength will constrain the input. -->
      <div class="field">
        <input
          type="text"
          name="minmaxText"
          id="minmaxText"
          pattern="^(.{2,8})$"
          size="8"
          @blur="doValidation"
          :aria-invalid="reportedMessages.minmaxText"
        />
        <label for="minmaxText">This text must be between 2-8 characters</label>
        <p class="errors">{{ reportedMessages.minmaxText }}</p>
      </div>
      <!-- required + max and min length. Combination of the above -->
      <div class="field">
        <input
          type="text"
          name="requiredMinmaxText"
          id="requiredMinmaxText"
          pattern="^(.{2,8})$"
          required
          @blur="doValidation"
          :aria-invalid="reportedMessages.requiredMinmaxText"
        />
        <label for="requiredMinmaxText"
          >This text is required and must also be between 2-8 characters</label
        >
        <p class="errors">{{ reportedMessages.requiredMinmaxText }}</p>
      </div>
      <p class="errors">This fieldset is not yet complete</p>
    </fieldset>
    <fieldset>
      <legend>Other kinds of input</legend>
      <!-- max and min number. -->
      <div class="field">
        <input
          type="number"
          name="minmaxNumber"
          id="minmaxNumber"
          pattern="^[0-9]$"
          min="2"
          max="8"
          @blur="doValidation"
          :aria-invalid="reportedMessages.minmaxNumber"
        />
        <label for="minmaxNumber">This number must be between 2-8</label>
        <p class="errors">{{ reportedMessages.minmaxNumber }}</p>
      </div>
      <!-- date - must not be in the past. We need to set a sensible-ish max as well. Without this, using the down arrow on the year field will give 275760 (on chrome)-->
      <div class="field">
        <input
          type="date"
          name="futureDate"
          id="futureDate"
          :min="today"
          max="2099-12-31"
          @blur="doValidation"
          :aria-invalid="reportedMessages.futureDate"
        />
        <label for="futureDate"
          >This date must be today or later, but is not required</label
        >
        <p class="errors">{{ reportedMessages.futureDate }}</p>
      </div>
      <div class="field">
        <input
          type="date"
          name="requiredDate"
          id="requiredDate"
          required
          @blur="doValidation"
          :aria-invalid="reportedMessages.requiredDate"
        />
        <label for="requiredDate">This date is required</label>
        <p class="errors">{{ reportedMessages.requiredDate }}</p>
      </div>
      <div class="field">
        <input
          type="time"
          name="requiredTime"
          id="requiredTime"
          required
          @blur="doValidation"
          :aria-invalid="reportedMessages.requiredTime"
        />
        <label for="requiredTime">This time is required</label>
        <p class="errors">{{ reportedMessages.requiredTime }}</p>
      </div>
      <!-- required + max length. This time in a textarea -->
      <!-- textarea does not support the pattern attribute, so we use maxlength instead
      -->
      <div class="field">
        <textarea
          name="requiredMinmaxTextArea"
          id="requiredMinmaxTextArea"
          rows="10"
          pattern="^(.{0,10})$"
          maxlength="10"
          required
          @blur="doValidation"
          :aria-invalid="reportedMessages.requiredMinmaxTextArea"
        ></textarea>
        <label for="requiredMinmaxTextArea"
          >This text is required and has max 10 characters</label
        >
        <p class="errors">{{ reportedMessages.requiredMinmaxTextArea }}</p>
      </div>
      <!-- required select -->
      <div class="field">
        <select
          name="requiredSelect"
          id="requiredSelect"
          required
          @change="doValidation"
          :aria-invalid="reportedMessages.requiredSelect"
        >
          <option :value="null">No option selected</option>
          <option value="1">Option 1</option>
          <option value="2">Option 2</option>
        </select>
        <label for="requiredSelect">This select is required</label>
        <p class="errors">{{ reportedMessages.requiredSelect }}</p>
      </div>
      <p class="errors">This fieldset is not yet complete</p>
    </fieldset>
    <fieldset>
      <div class="field">
        <input
          type="date"
          name="startDate"
          v-model="startDate"
          v-bind:max="endDate"
          @blur="doValidation"
          :aria-invalid="reportedMessages.startDate"
        />
        <label for="startDate">Start date</label>
        <p class="errors">{{ reportedMessages.startDate }}</p>
      </div>
      <div class="field">
        <input
          type="date"
          name="endDate"
          v-model="endDate"
          v-bind:min="startDate"
          @blur="doValidation"
          :aria-invalid="reportedMessages.endDate"
        />
        <label for="endDate">End date</label>
        <p class="errors">{{ reportedMessages.endDate }}</p>
      </div>
    </fieldset>
    <p class="errors">This form is not yet complete</p>
    <fieldset>
      <button type="submit">Submit</button>
      <button type="reset">Reset</button>
    </fieldset>
  </form>
</template>

<script>
/**
 * Desired behaviour:
 * - No validation errors shown initially
 * - Report a field's validity on blur
 * - Report all fields' validity on submit
 * - Show multiple errors simultaneously
 * - Do not prevent user from entering invalid data (no constraint)
 * - Replace default validation errors with custom errors
 * - Form submission will be blocked if any fields are invalid
 * - All errors to be displayed below the field
 * ref: https://www.w3.org/WAI/WCAG21/working-examples/aria-invalid-data-format/
 */

/**
 * Creates validation message for each element
 * We use the underlying ValidationState as keys. This allows us to let the browser to do the validation where it can.
 * We can also combine errors if a field has multiple validation errors
 */
const setValidationMessages = (elements) => {
  // For now, the same map will be applied everywhere
  const defaultMessages = {
    badInput: () => "Please enter a valid value",
    patternMismatch: () => "Does not match regex pattern",
    rangeOverflow: (element) =>
      element.type === "date"
        ? `Cannot be after ${new Date(element.max).toLocaleDateString()}`
        : "Too high",
    rangeUnderflow: (element) =>
      element.type === "date"
        ? `Cannot be before ${new Date(element.min).toLocaleDateString()}`
        : "Too low",
    stepMismatch: () => "Does not resolve to the given step size",
    tooLong: () => "Too long",
    tooShort: () => "Too short",
    typeMismatch: () => "Cannot be resolved to the required type",
    valueMissing: () => "Cannot be blank",
  };

  const customMessagesByName = {
    minmaxText: {
      ...defaultMessages,
      patternMismatch: () => "Must be between 2 and 8 characters",
    },
  };
  return elements.map((element) => {
    const messages = customMessagesByName[element.name] || defaultMessages;
    const validationMessage = Object.entries(messages)
      .filter((entry) => element.validity[entry[0]])
      .map((entry) => entry[1](element));
    element.setCustomValidity(validationMessage.join("|"));
    return element;
  });
};

export default {
  name: "Form",
  data: () => {
    return {
      reportedMessages: {},
      today: new Date().toISOString().slice(0, 10),
      startDate: null,
      endDate: null,
    };
  },
  computed: {
    fieldsToValidate: function () {
      return this.$el && typeof this.$el.querySelectorAll === "function"
        ? [...this.$el.querySelectorAll("[name]")].filter(
            (element) => element.willValidate
          )
        : [];
    },
    isValid: function () {
      return !Object.values(this.reportedMessages).length;
    },
  },
  methods: {
    reportMessages: function (elements) {
      this.reportedMessages = setValidationMessages(elements).reduce(
        (result, element) => {
          result[element.name] = element.validationMessage
            ? element.validationMessage.split("|").join(", ")
            : undefined;
          return result;
        },
        { ...this.reportedMessages }
      );
    },
    doValidation: function (event) {
      this.reportMessages([event.target]);
    },
    handleSubmit: function (event) {
      this.reportMessages(this.fieldsToValidate);
      const formArray = Array.from(new FormData(event.target));
      console.table(formArray);
    },
    handleReset: function () {
      this.reportedMessages = {};
    },
  },
};
</script>

<style scoped>
fieldset {
  display: flex;
  flex-direction: column;
  gap: 16px;
}
/**
** Field layout
** By placing the input first in the html
** we can select and style the other elements (e.g. the label)
** order property allows us to change the order of appearance
** e.g. so the label appears above */
.field {
  display: grid;
  /* add columns for inline layout */
  /* grid-template-columns: 1fr 2fr 1fr; */
  gap: 4px;
}

.field .label {
  order: 1;
}
.field input,
.field textarea,
.field select {
  order: 2;
}
.field .errors {
  order: 3;
}

form {
  --error-color: red;
  --warn-color: orange;
  --success-color: green;
  --info-color: blue;
}

.field :required ~ label:after {
  content: " *";
  color: var(--warn-color);
}
.field :not(:required) ~ label:after {
  content: " (optional)";
  color: var(--info-color);
}

[aria-invalid] {
  border: 1px solid var(--error-color);
}

.field .errors,
fieldset > .errors,
form > .errors {
  display: none;
  margin: 0;
}

fieldset:valid > legend::after {
  content: " (complete)";
  color: var(--success-color);
}

fieldset:invalid > legend::after {
  content: " (incomplete)";
  color: var(--warn-color);
}

.field [aria-invalid] ~ .errors:not(:empty) {
  display: inherit;
  color: var(--error-color);
}

fieldset:invalid > .errors,
form:invalid > .errors {
  display: inherit;
  color: var(--error-color);
}

fieldset:valid {
  border-color: var(--success-color);
}
</style>
