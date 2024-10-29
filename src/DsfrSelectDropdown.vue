<template>
  <div>
    <ul class="fr-tags-group" aria-label="Filtres appliqués">
      <li v-for="(tag, index) in selectedOptions" :key="tag">
        <button type="button" class="fr-tag fr-tag--sm fr-tag--dismiss" @click="removeTag(tag)"
          :aria-label="'Supprimer le filtre ' + getOptionLabel(tag)">
          {{ getOptionLabel(tag) }}
        </button>
      </li>
    </ul>
    <div class="fr-input-group__relative">
      <button @click="toggleDropdown" class="fr-select" :aria-expanded="isDropdownOpen" :aria-controls="dropdownId">
        {{ selectedOptionsLabel }}
      </button>
      <div v-if="isDropdownOpen" :id="dropdownId">
        <div class="fr-collapse fr-dropdown-shadow" :class="{ 'fr-collapse--expanded': isDropdownOpen }">
          <div class="fr-menu__alt fr-my-2w">
            <div class="fr-toggle fr-mb-2w">
              <input type="checkbox" class="fr-toggle__input" id="toggle-selection" @click="selectAll">
              <label class="fr-toggle__label" for="toggle-selection">Sélectionner toutes les options</label>
            </div>
            <div class="fr-input-group">
              <input type="search" aria-label="Filtre des options affichées" title="Filtre des options affichées"
                v-model="query" placeholder="Rechercher une option..." class="fr-input" />
            </div>
            <p :id="'search-options-' + dropdownId" role="status" aria-atomic="true" class="fr-sr-only">
              {{ filteredOptions.length }} options affichées</p>

            <fieldset class="fr-fieldset">
              <legend class="fr-fieldset__legend">{{ fieldsetLegend }}</legend>
              <div class="fr-fieldset__element" v-for="(option, index) in filteredOptions" :key="option.value">
                <div class="fr-checkbox-group">
                  <input type="checkbox" :id="option.value" :value="option.value" v-model="selectedOptions">
                  <label class="fr-label" :for="option.value"> {{ option.label }} </label>
                </div>
              </div>
            </fieldset>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    options: {
      type: Array,
      required: true
    },
    fieldsetLegend: {
      type: String,
      default: 'Choix de l\'option'
    }
  },
  data() {
    return {
      isDropdownOpen: false,
      query: '',
      selectedOptions: [],
    };
  },
  computed: {
    filteredOptions() {
      return this.options.filter(option =>
        option.label.toLowerCase().includes(this.query.toLowerCase())
      );
    },
    selectedOptionsLabel() {
      if (this.selectedOptions.length === 1) {
        return "Une option sélectionnée";
      } else if (this.selectedOptions.length > 1) {
        return this.selectedOptions.length + " options sélectionnées";
      } else {
        return "Sélectionner";
      }
    },
    dropdownId() {
      return `dropdown-${Math.random().toString(36).substr(2, 9)}`
    },
    allSelected() {
      return this.filteredOptions.length > 0 && this.filteredOptions.every(option =>
        this.selectedOptions.includes(option.value)
      );
    }
  },
  methods: {
    toggleDropdown() {
      this.isDropdownOpen = !this.isDropdownOpen;
    },
    selectAll() {
      if (this.allSelected) {
        this.selectedOptions = [];
      } else {
        this.selectedOptions = this.filteredOptions.map(option => option.value);
      }
    },
    removeTag(tag) {
      const index = this.selectedOptions.indexOf(tag);
      if (index !== -1) {
        this.selectedOptions.splice(index, 1);
      }
    },
    getOptionLabel(value) {
      const option = this.options.find(option => option.value === value);
      return option ? option.label : '';
    }
  },
};
</script>

<style scoped>
.fr-menu__alt {
  padding: 1rem;
  min-height: 300px;
}

.fr-dropdown-shadow {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
</style>
