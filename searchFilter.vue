<template>
  <form
    class="product__filter"
    ref="searchForm"
    @change="updateProducts"
    @submit.prevent="updateProducts"
    @keydown.enter.prevent="updateProducts"
  >
    <div class="product__filter-main">
      <div class="product__filter-top flex flex-justify">
        <span>{{ txtResult }}</span>
        <button class="product__filter-reload" @click.prevent="reload">
          {{ txtErase }}
        </button>
        <input-text
          :type="'fulltext'"
          :value="fulltext"
          :placeholder="filtersPlaceholder"
        ></input-text>
      </div>

      <div class="product__filter-top-tags">
        <ul v-if="tags.length != 0" class="product__filter-tags">
          <li class="product__filter-tag" v-for="tag in tags" :key="tag.index">
            {{ tag }}
            <button @click.prevent="removeTag(tag)" class="remove-tag"></button>
          </li>
        </ul>
      </div>
    </div>

    <accordion
      v-for="filter in filters"
      ref="accordion"
      :key="filter.type"
      :type="filter.type"
      :title="filter.title"
      :has-child="filter.subfilters ? true : false"
    >
      <accordion
        v-if="filter.subfilters"
        v-for="(subfilters, index) in filter.subfilters"
        ref="accordion"
        :key="subfilters.type + index"
        :type="subfilters.type"
        :title="subfilters.title"
        :has-child="false"
      >
        <input-checkbox
          v-for="input in subfilters.filters"
          ref="inputChild"
          v-on:change="check($event)"
          :key="input.code"
          :type="subfilters.type"
          :value="input.code"
          :label="input.libelle"
        />
      </accordion>
      <input-date v-if="filter.type === 'datepublished'"></input-date>
      <input-checkbox
        v-else
        v-for="input in filter.filters"
        ref="inputChild"
        v-on:change="check($event)"
        :key="input.code"
        :type="filter.type"
        :value="input.code"
        :label="input.libelle"
      />
    </accordion>
  </form>
</template>

<script>
import {
  combineKeyData,
  getAllUrlParams,
  getTranslation,
  showOverlay,
  hideOverlay
} from "../../../webpack.resolve/js/main.js";

import accordion from "../../accordion/webpack.module/accordion.vue";
import inputCheckbox from "../../inputCheckbox/webpack.module/inputCheckbox.vue";
import inputText from "../../inputText/webpack.module/inputText.vue";
import inputDate from "../../inputDate/webpack.module/inputDate.vue";

export default {
  name: "searchFilter",
  components: {
    accordion,
    inputCheckbox,
    inputText,
    inputDate
  },
  data() {
    return {
      name: "searchFilter",
      tags: []
    };
  },

  async created() {},

  async mounted() {
    this.$store.dispatch("getFilters");
    localStorage.getItem("tags");

    await new Promise(checkedInputsFromURL =>
      setTimeout(this.checkedInputsFromURL, 500)
    );
  },

  methods: {
    updateProducts: function(e) {
      let form = "";
      if (e != undefined) {
        form = e.currentTarget;
      } else {
        form = this.$refs.searchForm;
      }
      let params = [];
      form = new FormData(form);

      for (let input of form.entries()) {
        const cle = input[0];
        const value = input[1];
        if (
          cle === "date_publication_start" ||
          cle === "date_publication_end" ||
          cle === "date_validation_start" ||
          cle === "date_validation_end"
        ) {
          value = Date.parse(value);
          if (isNaN(value)) {
            value = "";
          }
        }
        if (value != "") {
          params.push({ [cle]: value });
        }
      }

      params = combineKeyData(params, ",");
      showOverlay();
      this.$store.dispatch("getProduct", params);

      var url = window.location.pathname;
      var urlParams = Object.keys(params)
        .map(function(k) {
          return encodeURIComponent(k) + "=" + encodeURIComponent(params[k]);
        })
        .join("&");
      var finalURL = url + "?" + urlParams;
      window.history.pushState(null, null, decodeURIComponent(finalURL));

      delete params['lang']

      let paramsArray = Object.values(params);
      let keysArray = Object.keys(params);

      for (let item of this.$refs.inputChild) {
        if (paramsArray.includes(item.value)) {
          item.checked = true;
        }
      }

      for (let accordion of this.$refs.accordion) {
        if (keysArray.includes(accordion.type)) {
          accordion.open = true;
          accordion.$options.parent.open === true;
        } else {
          accordion.open = false;
        }
        if (accordion.hasChild === true) {
          for (let children of accordion.$children) {
            if (children.open === true) {
              children.$parent.open = true;
            }
          }
        }
      }
    },
    reload: function(e) {
      window.location = window.location.origin + window.location.pathname;
      localStorage.clear();
    },
    checkedInputsFromURL: function() {
      let params = [];

      if (getAllUrlParams(document.URL)) {
        params = getAllUrlParams(document.URL);

        let that = this;

        delete params['lang']

        let paramsArray = Object.values(params);
        let keysArray = Object.keys(params);

        setTimeout(() => {
          for (let item of that.$refs.inputChild) {
            if (paramsArray.includes(item.value)) {
              item.checked = true;
              that.tags.splice(1, 0, item.label);
            }
          }

          for (let accordion of that.$refs.accordion) {
            if (keysArray.includes(accordion.type)) {
              accordion.open = true;

              if (paramsArray.includes("all")) {
                accordion.open = false;
              }
            } else {
              accordion.open = false;
            }
            if (accordion.hasChild === true) {
              for (let children of accordion.$children) {
                if (children.open === true) {
                  children.$parent.open = true;
                }
              }
            }
          }
        }, 1500);
      }
    },
    check: function($event) {
      let InputValue = $event.target.labels[0].innerText;

      this.$refs.inputChild.checked = $event.target.checked;
      this.$emit("check", $event.target.checked);

      if (
        this.$refs.inputChild.checked === true &&
        this.tags.includes(InputValue) === false
      ) {
        this.tags.splice(1, 0, InputValue);
      } else {
        this.tags = this.tags.filter(tag => tag !== InputValue);
      }

      let parsed = JSON.stringify(this.tags);
      localStorage.setItem("tags", parsed);
    },
    removeTag: function(tagTitle) {
      new Promise(resolve => {
        for (let item of this.$refs.inputChild) {
          if (
            new String(item.label).valueOf() == new String(tagTitle).valueOf()
          ) {
            item.checked = false;
          }
        }

        this.tags = this.tags.filter(tag => tag !== tagTitle);

        let parsed = JSON.stringify(this.tags);
        localStorage.setItem("tags", parsed);

        resolve();
      }).then(() => {
        setTimeout(() => {
          this.updateProducts();
        }, 10);
      });
    }
  },
  computed: {
    filters() {
      return this.$store.state.filters;
    },
    filtersPlaceholder() {
      if (
        this.$store.state.dictionnary["filtersplaceholder"] !== undefined &&
        this.$store.state.lang !== undefined
      ) {
        return this.$store.state.dictionnary["filtersplaceholder"][
          `${this.$store.state.lang}`
        ];
      }
    },

    filterTitle() {
      if (
        this.$store.state.dictionnary["filter"] !== undefined &&
        this.$store.state.lang !== undefined
      ) {
        return this.$store.state.dictionnary["filter"][
          `${this.$store.state.lang}`
        ];
      }
    },
    fulltext() {
      return decodeURIComponent(this.$store.state.fulltext.replace(/\+/g, " "));
    },
    txtResult() {
      if (
        this.$store.state.dictionnary["Search result"] !== undefined &&
        this.$store.state.lang !== undefined
      ) {
        return this.$store.state.dictionnary["Search result"][
          `${this.$store.state.lang}`
        ];
      }
    },
    txtErase() {
      if (
        this.$store.state.dictionnary["erase"] !== undefined &&
        this.$store.state.lang !== undefined
      ) {
        return this.$store.state.dictionnary["erase"][
          `${this.$store.state.lang}`
        ];
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.product {
  &__filter {
    &-top {
      font-size: 11px;
      text-transform: uppercase;
      letter-spacing: 1px;
      margin: 1.5em 0;

      .input {
        &__text {
          width: 100%;
        }
      }
    }

    &-main {
      border-bottom: 1px solid rgba($black, 0.1);
      margin: 1.5em 0 1em 0;
    }

    &-reload {
      padding: 0;
      text-decoration: underline;
      font-size: 12px;
      letter-spacing: 1px;
      font-family: Arial, Helvetica, sans-serif;
    }

    &-tags {
      border-top: 1px solid rgba(0, 0, 0, 0.1);
      padding: 1.8em 0 1em 0;

      &-header {
        width: 100%;
        padding-bottom: 0.5em;
        margin-bottom: 1.5em;

        > span {
          text-transform: uppercase;
          letter-spacing: 1px;
          font-size: 11px;
        }
      }
    }

    &-tag {
      font-size: 0.8em;
      display: inline-block;
      margin: 0 0.5em 1em 0;
      padding: 3px 23px 3px 10px;
      background-color: #ffffff;
      position: relative;

      button.remove-tag {
        position: absolute;
        right: 4px;
        transform: translate3d(0, 1px, 0) rotate(-45deg);
        transform-origin: center;
        outline: none;
        cursor: pointer;

        &:before,
        &:after {
          content: "";
          display: block;
          height: 1px;
          width: 10px;
          background: #000000;
          position: absolute;
          right: 0;
          top: 50%;
          transform: translate3d(-50%, 0, 0);
          transition: 0.3s ease transform;
        }

        &:after {
          transform: translate3d(-50%, 0, 0) rotate(270deg);
        }
      }
    }
  }
}
</style>
