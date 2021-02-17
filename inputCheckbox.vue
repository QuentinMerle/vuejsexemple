
<template>
  <div class="input__checkbox">
    <input type="checkbox" :name="type" :id="type + '-' + value" :value="value" :checked="IsChecked || checked" v-on="check" v-bind="$attrs">
    <label :for="type + '-' + value">{{label}}</label>
  </div>
</template>

<script>
export default {
  inheritAttrs: false,
  name: "inputCheckbox",
  data() {
    return {
      name: "inputCheckbox",
      checked: false
    };
  },
  props: {
    label: String,
    value: String,
    type: String,
    required: Boolean,
    IsChecked: Boolean
  },
  computed: {
    check: function(e) {
      var vm = this
      return Object.assign({},
        this.$listeners,
        {
          input: function (event) {
            vm.$emit('check', event.target.checked)
            vm.checked = event.target.checked
          }
        }
      )
    }
  }
};
</script>

<style lang="scss" scped>
.input {
  &__checkbox {
    min-height: 25px;
    margin-bottom: 9px;
    margin-top: 1px;
    position: relative;
    label {
      display: block;
      cursor: pointer;
      padding-left: 40px;
      line-height: 25px;

      &::first-letter{
        text-transform: uppercase;
      }

      &:before,
      &:after {
        position: absolute;
        left: 0;
        top: 50%;
        transform: translate3d(0, -50%, 0);
        content: "";
        display: block;
        height: 20px;
        width: 20px;
        border: 1px solid #c5c5c5;
        transition: 0.3s ease background;
      }

      &:after {
        background: #ffffff;
        border: none;
        transform-origin: 65% 1%;
        transform: scale(0.6) translate3d(0, -50%, 0);
      }

      &:hover {
        &:after {
          background: rgba(0, 0, 0, 0.3);
        }
      }
    }

    input:checked + label {
      &:after {
        background: #000000;
      }
    }
  }
}
</style>
