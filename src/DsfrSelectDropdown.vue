<template>
  <div>
    <p class="fr-label fr-mb-1w" aria-hidden="true">{{ label }}
      <span class="fr-hint-text" v-show="hint">{{ hint }}</span>
    </p>
    <ul class="fr-tags-group" aria-label="Filtres appliqués">
      <li v-for="tag in selectedOptions" :key="tag">
        <button type="button" class="fr-tag fr-tag--sm fr-tag--dismiss" @click="removeTag(tag)"
          :aria-label="'Supprimer le filtre ' + getOptionLabel(tag)">
          {{ getOptionLabel(tag) }}
        </button>
      </li>
    </ul>
    <div class="fr-input-group__relative" :aria-labelledby="dropdownId + '-label'">
      <button @click="toggleDropdown" class="fr-select text-left" :aria-expanded="isDropdownOpen"
        :aria-controls="dropdownId" :aria-label="label + ' ' + hint + ' ' + selectedOptionsLabel" ref="selectButtonRef">
        {{ selectedOptionsLabel }}
      </button>
      <div v-if="isDropdownOpen" :id="dropdownId">
        <div class="fr-collapse fr-dropdown-shadow" :class="{ 'fr-collapse--expanded': isDropdownOpen }">
          <div class="fr-menu__alt fr-mb-2w">
            <div class="fr-btns-group" v-show="showSelectAllBtn">
              <button class="fr-btn fr-btn--secondary fr-btn--sm fr-mb-2w" @click="selectAll">
                Tout {{ allSelected ? "désélectionner" : "sélectionner" }}
              </button>
            </div>
            <div class="fr-input-group">
              <input type="search" aria-label="Filtre des options affichées" title="Filtre des options affichées"
                v-model="query" placeholder="Rechercher une option..." class="fr-input" />
            </div>
            <p :id="'search-options-' + dropdownId" role="status" aria-atomic="true" class="fr-sr-only">
              {{ filteredOptions.length }} options affichées
            </p>
            <fieldset class="fr-fieldset">
              <legend class="fr-fieldset__legend fr-fieldset__legend--regular">{{ fieldsetLegend }}
                <span class="fr-hint-text" v-show="fieldsetLegendHint">{{ fieldsetLegendHint }}</span>
              </legend>
              <div class="fr-fieldset__element" v-for="option in filteredOptions" :key="option.value">
                <div class="fr-checkbox-group">
                  <input type="checkbox" :id="option.value" :value="option.value" v-model="selectedOptions">
                  <label class="fr-label" :for="option.value">{{ option.label }}</label>
                </div>
              </div>
            </fieldset>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, PropType, onMounted, onBeforeUnmount } from 'vue';

interface Option {
  label: string;
  value: string;
}

export default defineComponent({
  name: 'DsfrSelectDropdown',
  props: {
    options: {
      type: Array as PropType<Option[]>,
      required: true,
    },
    label: {
      type: String,
      required: true,
    },
    hint: {
      type: String,
      default: '',
    },
    fieldsetLegend: {
      type: String,
      default: 'Choix de l\'option',
    },
    fieldsetLegendHint: {
      type: String,
      default: '',
    },
    showSelectAllBtn: {
      type: Boolean,
      default: false,
    },
  },
  setup(props) {
    const isDropdownOpen = ref(false);
    const query = ref('');
    const selectedOptions = ref<string[]>([]);
    const selectButtonRef = ref<HTMLElement | null>(null);

    const filteredOptions = computed(() => {
      return props.options.filter(option =>
        option.label.toLowerCase().includes(query.value.toLowerCase())
      );
    });

    const selectedOptionsLabel = computed(() => {
      if (selectedOptions.value.length === 1) return "Une option sélectionnée";
      if (selectedOptions.value.length > 1) return `${selectedOptions.value.length} options sélectionnées`;
      return "Sélectionner";
    });

    const dropdownId = `dropdown-${Math.random().toString(36).substr(2, 9)}`;

    const allSelected = computed(() => {
      return (
        filteredOptions.value.length > 0 &&
        filteredOptions.value.every(option => selectedOptions.value.includes(option.value))
      );
    });

    const toggleDropdown = () => {
      isDropdownOpen.value = !isDropdownOpen.value;
    };

    const selectAll = () => {
      if (allSelected.value) {
        selectedOptions.value = [];
      } else {
        selectedOptions.value = filteredOptions.value.map(option => option.value);
      }
    };

    const removeTag = (tag: string) => {
      const index = selectedOptions.value.indexOf(tag);
      if (index !== -1) selectedOptions.value.splice(index, 1);
    };

    const getOptionLabel = (value: string) => {
      const option = props.options.find(option => option.value === value);
      return option ? option.label : '';
    };

    const handleKeydown = (event: KeyboardEvent) => {
      if (event.key === 'Escape' && isDropdownOpen.value) {
        isDropdownOpen.value = false;
        selectButtonRef.value?.focus();
      }
    };

    onMounted(() => {
      window.addEventListener('keydown', handleKeydown);
    });

    onBeforeUnmount(() => {
      window.removeEventListener('keydown', handleKeydown);
    });

    return {
      isDropdownOpen,
      query,
      selectedOptions,
      filteredOptions,
      selectedOptionsLabel,
      dropdownId,
      allSelected,
      toggleDropdown,
      selectAll,
      removeTag,
      getOptionLabel,
      selectButtonRef
    };
  },
});
</script>

<style scoped>
.fr-menu__alt {
  padding: 1rem;
  min-height: 300px;
  overflow-y: auto;
}

.fr-dropdown-shadow {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.text-left {
  text-align: left;
}
</style>
