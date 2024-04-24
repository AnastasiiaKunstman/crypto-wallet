<template>
    <div class="currency-converter">
        <h2 class="form_title">{{ title }}</h2>
        <div class="input-container">
            <label>Вы отдаете</label>
            <div class="input">
                <input min="0" type="text" v-model="inputValue" placeholder="Введите сумму"
                    @input="handleInput('input', $event.target.value)" @focus="clearCurrencySymbolInput" />
                <DropDown v-model="inputCurrency" :options="currencies" labelKey="name" valueKey="code"
                    :change="handleDropdownChange" />
            </div>
        </div>
        <div class="info">
            <div class="info-box">
                <div class="info-text">{{ exchangeRate }}</div>
                <img src="/src/assets/Line.svg" alt="" class="info-img" />
            </div>
            <button @click="swapCurrencies" class="converter-btn">
                <img src="/src/assets/icon.svg" class="form_icon" />
            </button>
        </div>
        <div class="input-container">
            <label>Вы получаете</label>
            <div class="input">
                <input min="0" type="text" v-model="outputValue" placeholder="Введите сумму"
                    @input="handleInput('output', $event.target.value)" @focus="clearCurrencySymbolOutput">
                <DropDown v-model="outputCurrency" :options="currencies" labelKey="name" valueKey="code"
                    :change="handleDropdownChange" />
            </div>
        </div>
        <button @click="calculate" class="convert-btn" :disabled="loading">
            {{ loading ? 'Загрузка...' : 'Продолжить' }}
        </button>
        <p class="error" v-if="error !== ''">{{ error }}</p>
    </div>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted, watch } from 'vue';
import DropDown from './DropDown.vue';

// Состояние компонента
const inputValue = ref('');
const inputCurrency = ref('');
const outputValue = ref('');
const outputCurrency = ref('');
const currencies = ref([]);
const exchangeRate = ref('');

const axiosInstance = axios.create();
let cancelToken = null;
const error = ref('');
const loading = ref(false);

const title = "Введите сумму обмена и выберите валюты";

// Методы компонента
async function fetchCurrencies() {
    try {
        const response = await axiosInstance.get('https://api-exchange.cryptocloud.plus/exchange/all_currency');
        currencies.value = response.data.map(currency => ({
            code: currency.code,
            name: currency.name,
            icon: currency.icon,
            full_name: currency.full_name,
            full_network: currency.full_network
        }));

        if (currencies.value.length > 0) {
            inputCurrency.value = currencies.value[0].code;
            outputCurrency.value = currencies.value[1].code;
        }

    } catch (error) {
        console.error('Error fetching currencies:', error);
    }
}

function handleInput(type, value) {
    value = value.replace(',', '.');

    if (/^00/.test(value)) {
        value = value.slice(1);
    }

    const currencySymbol = type === 'input' ? inputCurrency.value : outputCurrency.value;
    value = value ? value : `${value} ${currencySymbol}`;

    // Устанавливаем значение в соответствующий инпут
    if (type === 'input') {
        inputValue.value = value;
    } else {
        outputValue.value = value;
    }
}

function clearCurrencySymbolInput() {
    inputValue.value = '';
}

function clearCurrencySymbolOutput() {
    outputValue.value = '';
}

function showError(message) {
    error.value = message;

    setTimeout(() => {
        error.value = '';
    }, 5000);
}

function handleDropdownChange(value) {
    console.log('Selected option:', value);
}

async function calculate() {
    loading.value = true;

    if (cancelToken) {
        cancelToken.cancel('Operation canceled by the user.');
    }

    cancelToken = axios.CancelToken.source();

    try {
        if (!inputCurrency.value || !outputCurrency.value) {
            showError('Выберите валюту в обоих полях');
            return;
        }

        if (inputCurrency.value == outputCurrency.value) {
            showError('Выберите разную валюту');
            return;
        }

        if (!inputValue.value && !outputValue.value) {
            showError('Введите сумму для обмена');
            return;
        }

        const amount = parseFloat(inputValue.value ? inputValue.value : outputValue.value);
        const from_float = inputValue.value ? inputCurrency.value : outputCurrency.value;
        const to_float = inputValue.value ? outputCurrency.value : inputCurrency.value;

        const response = await axiosInstance.post(
            'https://api-exchange.cryptocloud.plus/exchange/profit_amount',
            { amount, from_float, to_float },
            { cancelToken: cancelToken.token }
        );

        const result = response.data.profit_rate.toFixed(2);

        if (inputValue.value) {
            outputValue.value = result.toString() + " " + outputCurrency.value;
        } else {
            inputValue.value = result.toString() + " " + inputCurrency.value;
        }

        error.value = '';
    } catch (error) {
        if (axios.isCancel(error)) {
            console.log('Request canceled:', error.message);
        } else {
            console.error('Error calculating:', error);
            error.value = 'Ошибка при вычислении: ' + error.message;
        }
    } finally {
        loading.value = false;
    }
}

function swapCurrencies() {
    const tempValue = inputValue.value;
    const tempCurrency = inputCurrency.value;

    inputValue.value = outputValue.value;
    inputCurrency.value = outputCurrency.value;
    outputValue.value = tempValue;
    outputCurrency.value = tempCurrency;
}

async function calculateExchangeRate() {
    try {
        if (inputCurrency.value && outputCurrency.value) {
            const amount = 1; // Фиксированная сумма для расчета обменного курса
            const from_float = inputCurrency.value;
            const to_float = outputCurrency.value;

            const response = await axiosInstance.post(
                'https://api-exchange.cryptocloud.plus/exchange/profit_amount',
                { amount, from_float, to_float }
            );

            const result = response.data.profit_rate.toFixed(5);
            exchangeRate.value = `1 ${inputCurrency.value} ~ ${result} ${outputCurrency.value}`;
        }
    } catch (error) {
        console.error('Error calculating exchange rate:', error.message);
    }
}

watch([inputCurrency, outputCurrency], calculateExchangeRate);

onMounted(calculateExchangeRate);

onMounted(() => {
    fetchCurrencies();
});
</script>

<style scoped>
.currency-converter {
    width: 100%;
    height: 52.8vh;
    padding: 7.25% 7.25% 11.8%;
    border-radius: 20px;
    background: #FFF;
    box-shadow: 0px 5px 50px -5px rgba(15, 15, 15, 0.05);
}

.form_title {
    width: 85%;
    margin-bottom: 6.9%;
    color: #23262F;
    font-size: clamp(20px, 1.94vw, 28px);
    font-style: normal;
    font-weight: 600;
    line-height: normal;
    letter-spacing: 0.28px;
}

.input-container {
    display: flex;
    flex-direction: column;
    gap: 6px;
}

.input-container label {
    color: #5F5F5F;
    font-size: clamp(10px, 0.97vw, 14px);
    font-weight: 400;
    line-height: 171.429%;
}

.input {
    display: flex;
    align-items: center;
}

.input input {
    width: 100%;
    height: 5.05vh;
    border-radius: 6px;
    border: 1px solid #F0F0F0;
    background: #FFF;
    padding: 0.84vw;
    color: #3C3C3C;
    font-size: clamp(14px, 1.1vw, 16px);
    font-style: normal;
    font-weight: 400;
    line-height: 150%;
}

.input input:focus-visible {
    outline-color: #F0F0F0;
}

.input input:focus::-webkit-input-placeholder {
    color: transparent;
}

.amount-input {
    width: 100%;
}

.converter-btn {
    border: none;
    background-color: transparent;
    cursor: pointer;
}

.form_icon {
    max-width: 24px;
    max-height: 24px;
    cursor: pointer;
}

.convert-btn {
    width: 100%;
    height: 5.5vh;
    padding: 1.11vw 1.66vw;
    border-radius: 90px;
    background: #4A40C6;
    color: #FFF;
    border: none;
    cursor: pointer;
    margin-top: 3.47vw;
    font-size: clamp(14px, 1.1vw, 16px);
    font-style: normal;
    font-weight: 500;
    line-height: 150%;
}

.convert-btn:hover {
    opacity: 0.7;
}

.convert-btn:disabled {
    opacity: 0.5;
}

.info {
    display: flex;
    margin-top: 0.7vw;
    justify-content: space-between;
    height: 34px;
    flex-wrap: wrap;
    align-content: center;
}

.info-box {
    display: flex;
    flex-direction: column;
}

.info-text {
    height: 12px;
    margin-bottom: 0.42vw;
    color: #5F5F5F;
    font-size: clamp(8px, 0.8vw, 12px);
    line-height: normal;
}

.info-img {
    width: 7.8vw;
}

.currency-img {
    width: 1.7vw;
    height: 1.7vw;
    margin-right: 0.35vw;
}

.error {
    margin-top: 0.35vw;
    color: red;
    text-align: center;
}
</style>