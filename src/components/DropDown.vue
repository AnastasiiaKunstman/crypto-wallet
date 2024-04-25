<template>
    <div class="dropdown-wrapper">
        <div class="dropdown-selected-option" @click="toggleDropDownVisibility">
            <div class="dropdown-selected-option__box">
                <img :src="selectedOption ? selectedOption.icon : ''" alt="" class="dropdown-selected-option__icon" />
                <p class="dropdown-selected-option__text">{{ mappedSelectedOption }}</p>
            </div>
            <img src="/src/assets/dropdown.svg" alt="" class="dropdown-selected-option__img" />
        </div>
        <div class="options-wrapper" v-if="isDropDownVisible" ref="optionsWrapper">
            <div class="options-header">
                <p class="options-header__title">Выберите криптовалюту</p>
                <img src="/src/assets/x.svg" alt="" @click="toggleDropDownVisibility" />
            </div>
            <div class="search-input-wrapper">
                <input type="search" placeholder="Название валюты..." class="search-input-wrapper__search"
                    v-model="searchText" />
                <img class="search-input-wrapper__img" src="/src/assets/search.svg" alt="" />
            </div>
            <div class="options-filters">
                <p class="options-filters__filter" v-for="filter in filters" :key="filter">{{ filter }}</p>
            </div>
            <div class="option" v-for="(option, index) in filteredOptions" :key="index"
                @click="handleOptionSelect(option)">
                <div class="option__box">
                    <img :src="option.icon" :alt="option.name" class="option__img" />
                    <p>{{ option.name }}</p>
                </div>
                <div class="option__box2">
                    <p>{{ option.full_name }}</p>
                    <p class="option__network">{{ option.full_network !== option.full_name ? option.full_network : '' }}
                    </p>
                </div>
            </div>
            <p v-if="filteredOptions.length === 0" class="no-results">Ничего не найдено</p>
        </div>
    </div>
</template>

<script setup>
import { ref, defineProps, computed, defineEmits, onMounted, onUnmounted, nextTick } from 'vue';

const isDropDownVisible = ref(false);
const searchText = ref('');
const emit = defineEmits(['update:modelValue'])
const optionsWrapper = ref('');

const props = defineProps({
    modelValue: String,
    options: Array,
    labelKey: String,
    valueKey: String,
    change: Function
});

const toggleDropDownVisibility = () => {
    isDropDownVisible.value = !isDropDownVisible.value;
};

const handleOptionSelect = (option) => {
    props.change(option[props.valueKey]);
    emit('update:modelValue', option[props.valueKey])
    isDropDownVisible.value = false;
};

const selectedOption = computed(() => {
    if (!props.options || props.options.length === 0 || !props.modelValue) {
        return null;
    }
    return props.options.find(option => option[props.valueKey] === props.modelValue);
});

const mappedSelectedOption = computed(() => {
    return selectedOption.value ? selectedOption.value[props.labelKey] : '';
});

const filters = ['Все', 'Стейблкоины', 'Классика'];

const filteredOptions = computed(() => {
    return props.options.filter(option =>
        option[props.labelKey].toLowerCase().includes(searchText.value.toLowerCase()) && option[props.valueKey] !== selectedOption.value
    );
});

// Закрываем выпадающий список при клике за его пределами
onMounted(() => {
    document.addEventListener('click', closeDropDown);
});

onUnmounted(() => {
    document.removeEventListener('click', closeDropDown);
});

const closeDropDown = (event) => {
    if (!event.target.closest('.dropdown-wrapper')) {
        isDropDownVisible.value = false;
    }
};
</script>

<style scoped>
.dropdown-wrapper {
    width: 28.4%;
    height: 48px;
    margin-left: -23.5%;
    border-radius: 6px;
    position: relative;
    background: #FFF;
}

.dropdown-selected-option {
    width: 100%;
    height: 100%;
    line-height: 44px;
    border: 1px solid #F0F0F0;
    border-radius: 6px;
    padding: 0.84vw 1.46vw 0.84vw 0.84vw;
    cursor: pointer;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 0.42vw;
}

.dropdown-selected-option__box {
    display: flex;
    gap: 0.42vw;
}

.dropdown-selected-option__img {
    max-width: 8px;
}

.dropdown-selected-option__text {
    font-size: clamp(12px, 0.97vw, 14px);
    font-style: normal;
    font-weight: 400;
}

.search-input-wrapper {
    width: 100%;
    position: relative;
}

.search-input-wrapper input:focus-visible {
    outline-color: #958EE8;
}

.search-input-wrapper input:focus::-webkit-input-placeholder {
    color: transparent;
}

.search-input-wrapper__img {
    position: absolute;
    right: 6px;
    top: 6px;
}

.options-wrapper {
    position: absolute;
    top: 100%;
    right: 0;
    width: 265px;
    height: 338px;
    overflow: auto;
    padding: 14px 12px 0;
    background-color: #fff;
    border: 1px solid #f1f3f9;
    border-radius: 6px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    z-index: 999;
}

.options-wrapper::-webkit-scrollbar {
    width: 2px;
    height: 50%;
}

.options-wrapper::-webkit-scrollbar-track {
    background: #f1f1f1;
}

.options-wrapper::-webkit-scrollbar-thumb {
    background: #BFBCDD;
    border-radius: 4px;
}

.options-wrapper::-webkit-scrollbar-thumb:hover {
    background: #555;
}

.options-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 12px;
}

.options-header__title {
    font-size: 14px;
    font-style: normal;
    font-weight: 400;
    line-height: 128.571%;
}

.search-input-wrapper__search {
    width: 241px;
    height: 32px;
    border-radius: 6px;
    border: 1px solid #F0F0F0;
    background: #FFF;
    padding: 8px 6px 8px 9px;
    margin-bottom: 10px;
}

.options-filters {
    display: flex;
    gap: 6px;
    margin-bottom: 10px;
}

.options-filters__filter {
    display: flex;
    width: 88px;
    height: 24px;
    padding: 4px 8px;
    justify-content: center;
    align-items: center;
    gap: 6px;
    border-radius: 50px;
    background: #F1F3F9;
    color: #5F5F5F;
    font-size: clamp(7px, 0.97vw, 10px);
    font-style: normal;
    font-weight: 400;
    line-height: 240%;
}

.options-filters__filter:first-child {
    width: 53px;
    border: 1px solid #958EE8;
}

.options-filters__filter:hover {
    border: 1px solid #958EE8;
    cursor: pointer;
}

.option {
    width: 100%;
    height: 44px;
    line-height: 44px;
    padding: 0 12px;
    cursor: pointer;
    border-radius: 6px;
    border: 1px solid #f1f3f9;
    margin-bottom: 6px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.option:active {
    border: 1px solid #958EE8;
}

.option:hover {
    border: 1px solid #958EE8;
}

.option:last-child {
    margin: 0;
}

.option__img {
    width: 20px;
    height: 20px;
}

.option__box {
    display: flex;
    gap: 6px;
    align-items: center;
    color: #3C3C3C;
    font-size: clamp(10px, 0.97vw, 14px);
    font-style: normal;
    font-weight: 400;
    line-height: 128.571%;
}

.option__box2 {
    display: flex;
    flex-direction: column;
    gap: 4px;
    color: #5F5F5F;
    text-align: right;
    font-size: clamp(8px, 0.7vw, 10px);
    font-style: normal;
    font-weight: 300;
    line-height: 100%;
}

.option__network {
    color: #9A9A9A;
    text-align: right;
    font-size: clamp(6px, 0.55vw, 8px);
    font-weight: 300;
    line-height: 125%;
}
</style>