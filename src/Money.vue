<template>
    <span v-if="label">
        {{ money }}
    </span>
    <input v-model="money"
        :disabled="disabled"
        :readonly="readonly"
        :placeholder="placeholder"
        type="tel"
        @blur="focus = false; update($event)"
        @keydown.enter.prevent="update"
        @focus="focus = true; money = modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        v-else>
</template>

<script>
import accounting from 'accounting-js';

export default {
    name: 'Money',

    props: {
        modelValue: {
            type: [Number, String],
            default: null,
        },
        label: {
            type: Boolean,
            default: false,
        },
        readonly: {
            type: Boolean,
            default: false,
        },
        disabled: {
            type: Boolean,
            default: false,
        },
        placeholder: {
            type: String,
            default: null,
        },
        symbol: {
            type: String,
            default: '$',
        },
        decimal: {
            type: String,
            default: '.',
            validator: val => ['.', ','].includes(val),
        },
        thousand: {
            type: String,
            default: ',',
            validator: val => [' ', '.', ','].includes(val),
        },
        precision: {
            type: Number,
            default: 2,
            validator: val => val >= 0,
        },
        positive: {
            type: String,
            default: '%s %v',
        },
        negative: {
            type: String,
            default: '%s (%v)',
        },
        zero: {
            type: String,
            default: '%s --',
        },
    },

    emits: ['update:modelValue'],

    data: () => ({
        money: null,
        focus: false,
    }),

    watch: {
        modelValue: 'format',
    },

    created() {
        this.format();
    },

    methods: {
        format() {
            if (this.focus) {
                return;
            }

            this.money = accounting.formatMoney(this.modelValue, {
                symbol: this.symbol,
                precision: this.precision,
                thousand: this.thousand,
                decimal: this.decimal,
                format: {
                    pos: this.positive,
                    neg: this.negative,
                    zero: this.zero,
                },
            });
        },
        update(event) {
            let value = event.target.value.split(this.decimal).join('.');
            value = parseFloat(value);

            if (isNaN(value)) {
                value = null;
            } else {
                value = this.round(value);
            }

            this.$emit('update:modelValue', value);
            this.format();
        },
        round(value) {
            const factor = 10 ** this.precision;
            return Math.round(value * factor) / factor;
        },
    },
};
</script>
