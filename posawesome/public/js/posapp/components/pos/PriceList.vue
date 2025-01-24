<template>
  <div class="price-list-wrapper">
    <v-autocomplete
      dense
      flat
      clearable
      auto-select-first
      outlined
      hide-details
      color="primary"
      :label="frappe._('Price List')"
      v-model="price_list"
      :items="price_lists"
      item-text="price_list"
      item-value="name"
      background-color="white"
      :no-data-text="__('Price List not found')"
      :filter="customFilter"
      :disabled="readonly"
      @click:prepend-inner="edit_price_list"
      @change="logPriceList"
      class="custom-autocomplete"
    />
  </div>
</template>

<script>
import { evntBus } from '../../bus';

export default {
  data: () => ({
    pos_profile: '',
    price_lists: [],
    price_list: '', // v-model binds this
    readonly: false,
  }),

  methods: {
    get_price_list() {
      const vm = this;
      if (this.price_lists.length > 0) {
        return;
      }
      if (vm.pos_profile.posa_local_storage && localStorage.pl_storage) {
        vm.price_lists = JSON.parse(localStorage.getItem('pl'));
      }
      frappe.call({
        method: 'posawesome.posawesome.api.posapp.get_price_list',
        args: {
          pos_profile: this.pos_profile.pos_profile,
        },
        callback: function (r) {
          if (r.message) {
            vm.price_lists = r.message;
            if (vm.pos_profile.posa_local_storage) {
              localStorage.setItem('pl', '');
              localStorage.setItem(
                'pl',
                JSON.stringify(r.message)
              );
            }
          }
        },
      });
    },
    edit_price_list() {
      // Implement your edit logic here
    },
    logPriceList(selectedValue) {
      localStorage.setItem(
        'pl',
        selectedValue
      );
    },
  },

  created: function () {
    this.$nextTick(function () {
      evntBus.$on('register_pos_profile', (pos_profile) => {
        this.pos_profile = pos_profile;
        this.get_price_list();
      });
      evntBus.$on('payments_register_pos_profile', (pos_profile) => {
        this.pos_profile = pos_profile;
        this.get_price_list();
      });
    });
  },

  watch: {
    price_list() {
      evntBus.$emit('update_price_list', this.price_list);
    },
  },
};
</script>

<style scoped>
.price-list-wrapper {
  display: flex;
  align-items: center;
  max-width: 150px;
  margin: 0;
}

.custom-autocomplete .v-input {
  height: 30px; /* Set height to 30px */
}

.custom-autocomplete .v-input__control {
  min-height: 30px; /* Ensure control matches the specified height */
  padding: 0 6px; /* Adjust padding to fit the compact height */
  font-size: 12px; /* Reduce font size for better spacing */
}

.custom-autocomplete .v-icon {
  font-size: 16px; /* Scale down icons */
}
</style>