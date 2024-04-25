<template>
  <main>
    <div class="info">
      <h1 class="info-title">
        {{ title }} <span>{{ span }}</span>
      </h1>
      <p class="info-subtitle">
        {{ subtitle }}
      </p>
      <!-- <img class="info-img" src="/src/assets/Group.png" alt="" /> -->
    </div>
    <div class="form">
      <Form />
    </div>
  </main>
</template>

<script setup>
import Form from './components/Form.vue';

const title = "Обмен криптовалюты";
const span = "без ограничений";
const subtitle = "Обменивайте Bitcoin, Litecoin, Ethereum, Tether и более 50 альткоинов быстрым, простым и безопасным способом.";
</script>

<style scoped>
.info {
  width: 57.3%;
  display: flex;
  flex-direction: column;
  gap: 1.7vw;
}

.info-title {
  width: 27vw;
  font-size: clamp(24px, 2.5vw, 36px);
  font-style: normal;
  font-weight: 700;
  line-height: normal;
  letter-spacing: 0.36px;
}

span {
  color: #4A40C6;
}

.info-subtitle {
  width: 27.85vw;
  font-size: clamp(14px, 1.25vw, 18px);
  font-style: normal;
  font-weight: 300;
  line-height: 133.333%;
}

.info-img {
  margin-left: -4.6vw;
  transform: rotate(10.874deg);
}

.form {
  width: 32vw;
}

@media (max-width: 950px) {

  .info,
  .info-title,
  .info-subtitle {
    width: 100%;
  }

  .form {
    width: 70%;
  }
}

@media (max-width: 680px) {
  .form {
    width: 100%;
  }
}
</style>