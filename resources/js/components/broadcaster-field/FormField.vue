<template>
  <default-field :field="field" :errors="errors">
    <template slot="field">
      <input
          ref="input"
          :id="field.name"
          :type="this.field.type"
          class="w-full form-control form-input form-input-bordered"
          :class="errorClasses"
          :placeholder="field.name"
          :value="value | moneyFormat(field.numberFormat)"
          @input="setFieldAndMessage"
      />
    </template>
  </default-field>
</template>

<script>
import {FormField, HandlesValidationErrors} from "laravel-nova";
import numeral from "numeral";

export default {
  mixins: [FormField, HandlesValidationErrors],

  props: ["resourceName", "resourceId", "field"],

  mounted() {
    this.$nextTick(() => {
      if (Array.isArray(this.field.broadcastTo)) {
        this.emitParsedValue(this.field.broadcastTo[0] + 'Created', this.value);
        this.emitParsedValue(this.field.broadcastTo[1] + 'Created', this.value);
      } else {
        this.emitParsedValue(this.field.broadcastTo + 'Created', this.value);
      }
    });
  },

  methods: {
    setFieldAndMessage(el) {
      console.log(el);
      const rawValue = el.target ? el.target.value : el.value;
      let parsedValue = rawValue;

      if (this.field.type === "number") {
        parsedValue = Number(rawValue);
      }

      if (Array.isArray(this.field.broadcastTo)) {
        this.emitParsedValue(this.field.broadcastTo[0], parsedValue);
        this.emitParsedValue(this.field.broadcastTo[1], parsedValue);
      } else {
        this.emitParsedValue(this.field.broadcastTo, parsedValue);
      }

      this.value = parsedValue;
    },

    emitParsedValue(broadcast, value) {
      Nova.$emit(broadcast, {
        field_name: this.field.attribute,
        value,
      });
    },

    /*
     * Set the initial, internal value for the field.
     */
    setInitialValue() {
      this.value = this.field.value || "";
    },

    /**
     * Fill the given FormData object with the field's internal value.
     */
    fill(formData) {
      formData.append(this.field.attribute, this.value || "");
    },

    /**
     * Update the field's internal value.
     */
    handleChange(value) {
      this.value = value;
    }
  },
  filters: {
    moneyFormat(number, format) {
      if (!format) {
        return number;
      }
      return numeral(number).format(format);
    }
  }
};
</script>