<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import BaseButton from '@/components/ui/BaseButton.vue'
import BaseInput from '@/components/ui/BaseInput.vue'
import { useUserStore } from '@/stores/user'

const router = useRouter()
const userStore = useUserStore()

const isLogin = ref(true)
const username = ref('')
const password = ref('')
const cardCode = ref('')
const error = ref('')
const success = ref('')
const loading = ref(false)

async function handleSubmit() {
  loading.value = true
  error.value = ''
  success.value = ''

  try {
    if (isLogin.value) {
      const result = await userStore.login(username.value, password.value)
      if (result.ok) {
        router.push('/')
      }
      else {
        error.value = result.error || '登录失败'
      }
    }
    else {
      if (!cardCode.value) {
        error.value = '请输入卡密'
        loading.value = false
        return
      }
      const result = await userStore.register(username.value, password.value, cardCode.value)
      if (result.ok) {
        success.value = '注册成功，请登录'
        isLogin.value = true
        cardCode.value = ''
      }
      else {
        error.value = result.error || '注册失败'
      }
    }
  }
  catch (e: any) {
    error.value = e.response?.data?.error || e.message || '操作异常'
  }
  finally {
    loading.value = false
  }
}

function toggleMode() {
  isLogin.value = !isLogin.value
  error.value = ''
  success.value = ''
}
</script>

<template>
  <div class="login-container">
    <!-- 背景装饰 -->
    <div class="bg-decoration">
      <!-- 太阳 -->
      <div class="sun" />
      <!-- 云朵 -->
      <div class="cloud cloud-1" />
      <div class="cloud cloud-2" />
      <div class="cloud cloud-3" />
      <!-- 草地 -->
      <div class="grass" />
      <!-- 植物装饰 -->
      <div class="plant plant-1">
        🌱
      </div>
      <div class="plant plant-2">
        🌻
      </div>
      <div class="plant plant-3">
        🌾
      </div>
      <div class="plant plant-4">
        🌿
      </div>
      <div class="plant plant-5">
        🥕
      </div>
      <div class="plant plant-6">
        🍅
      </div>
    </div>

    <!-- 登录卡片 -->
    <div class="login-card">
      <!-- Logo 区域 -->
      <div class="logo-area">
        <div class="logo-icon">
          <span class="text-5xl">🌾</span>
        </div>
        <h1 class="logo-title">
          QQ农场智能助手
        </h1>
        <p class="logo-subtitle">
          {{ isLogin ? '欢迎回来，开始你的农场之旅' : '加入我们，开启农场新生活' }}
        </p>
      </div>

      <!-- 表单区域 -->
      <form class="form-area" @submit.prevent="handleSubmit">
        <div class="form-group">
          <label class="form-label">
            <span class="label-icon">👤</span>
            用户名
          </label>
          <BaseInput
            id="username"
            v-model="username"
            type="text"
            placeholder="请输入用户名"
            required
          />
        </div>

        <div class="form-group">
          <label class="form-label">
            <span class="label-icon">🔒</span>
            密码
          </label>
          <BaseInput
            id="password"
            v-model="password"
            type="password"
            placeholder="请输入密码"
            required
          />
        </div>

        <div v-if="!isLogin" class="form-group">
          <label class="form-label">
            <span class="label-icon">🎫</span>
            卡密
          </label>
          <BaseInput
            id="cardCode"
            v-model="cardCode"
            type="text"
            placeholder="请输入卡密"
            :required="!isLogin"
          />
          <p class="form-hint">
            💡 请输入有效的时间卡密进行注册
          </p>
        </div>

        <div v-if="error" class="message error-message">
          <span class="message-icon">⚠️</span>
          {{ error }}
        </div>

        <div v-if="success" class="message success-message">
          <span class="message-icon">✅</span>
          {{ success }}
        </div>

        <BaseButton
          type="submit"
          variant="primary"
          block
          :loading="loading"
          class="submit-btn"
        >
          <span v-if="!loading">{{ isLogin ? '🚀 立即登录' : '🎉 立即注册' }}</span>
        </BaseButton>
      </form>

      <!-- 切换区域 -->
      <div class="switch-area">
        <button
          type="button"
          class="switch-btn"
          @click="toggleMode"
        >
          {{ isLogin ? '🌱 没有账号？立即注册' : '🌿 已有账号？立即登录' }}
        </button>
      </div>

      <!-- 底部装饰 -->
      <div class="card-footer">
        <span>🌻 愿你的农场丰收满满 🌻</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.login-container {
  min-height: 100vh;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(180deg, #87ceeb 0%, #98d8c8 50%, #7cb342 100%);
  position: relative;
  overflow: hidden;
}

/* 背景装饰 */
.bg-decoration {
  position: absolute;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
}

/* 太阳 */
.sun {
  position: absolute;
  top: 40px;
  right: 80px;
  width: 80px;
  height: 80px;
  background: radial-gradient(circle, #ffd700 0%, #ffa500 100%);
  border-radius: 50%;
  box-shadow: 0 0 60px 20px rgba(255, 215, 0, 0.4);
  animation: sunPulse 4s ease-in-out infinite;
}

@keyframes sunPulse {
  0%,
  100% {
    transform: scale(1);
    box-shadow: 0 0 60px 20px rgba(255, 215, 0, 0.4);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 0 80px 30px rgba(255, 215, 0, 0.5);
  }
}

/* 云朵 */
.cloud {
  position: absolute;
  background: white;
  border-radius: 50px;
  opacity: 0.9;
}

.cloud::before,
.cloud::after {
  content: '';
  position: absolute;
  background: white;
  border-radius: 50%;
}

.cloud-1 {
  top: 60px;
  left: 10%;
  width: 100px;
  height: 40px;
  animation: cloudFloat 20s linear infinite;
}

.cloud-1::before {
  width: 50px;
  height: 50px;
  top: -25px;
  left: 15px;
}

.cloud-1::after {
  width: 35px;
  height: 35px;
  top: -15px;
  right: 15px;
}

.cloud-2 {
  top: 120px;
  left: 60%;
  width: 80px;
  height: 32px;
  animation: cloudFloat 25s linear infinite;
  animation-delay: -5s;
}

.cloud-2::before {
  width: 40px;
  height: 40px;
  top: -20px;
  left: 10px;
}

.cloud-2::after {
  width: 28px;
  height: 28px;
  top: -12px;
  right: 10px;
}

.cloud-3 {
  top: 200px;
  left: 30%;
  width: 60px;
  height: 24px;
  animation: cloudFloat 30s linear infinite;
  animation-delay: -10s;
}

.cloud-3::before {
  width: 30px;
  height: 30px;
  top: -15px;
  left: 8px;
}

.cloud-3::after {
  width: 22px;
  height: 22px;
  top: -10px;
  right: 8px;
}

@keyframes cloudFloat {
  0% {
    transform: translateX(-100px);
  }
  100% {
    transform: translateX(calc(100vw + 100px));
  }
}

/* 草地 */
.grass {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 120px;
  background: linear-gradient(180deg, #7cb342 0%, #558b2f 100%);
  border-radius: 100% 100% 0 0;
}

.grass::before {
  content: '';
  position: absolute;
  top: -20px;
  left: 0;
  right: 0;
  height: 40px;
  background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 20'%3E%3Cpath fill='%237CB342' d='M0 20 Q25 0 50 20 Q75 0 100 20 V0 H0Z'/%3E%3C/svg%3E")
    repeat-x;
  background-size: 100px 20px;
}

/* 植物装饰 */
.plant {
  position: absolute;
  font-size: 2rem;
  animation: plantSway 3s ease-in-out infinite;
}

.plant-1 {
  bottom: 100px;
  left: 5%;
  animation-delay: 0s;
}
.plant-2 {
  bottom: 80px;
  left: 15%;
  animation-delay: 0.5s;
  font-size: 2.5rem;
}
.plant-3 {
  bottom: 110px;
  left: 25%;
  animation-delay: 1s;
}
.plant-4 {
  bottom: 90px;
  right: 25%;
  animation-delay: 1.5s;
}
.plant-5 {
  bottom: 100px;
  right: 15%;
  animation-delay: 2s;
}
.plant-6 {
  bottom: 85px;
  right: 5%;
  animation-delay: 2.5s;
  font-size: 2.5rem;
}

@keyframes plantSway {
  0%,
  100% {
    transform: rotate(-5deg);
  }
  50% {
    transform: rotate(5deg);
  }
}

/* 登录卡片 */
.login-card {
  width: 100%;
  max-width: 420px;
  margin: 20px;
  padding: 40px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 24px;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.15),
    0 0 0 1px rgba(255, 255, 255, 0.5);
  position: relative;
  z-index: 10;
  backdrop-filter: blur(10px);
}

/* Logo 区域 */
.logo-area {
  text-align: center;
  margin-bottom: 32px;
}

.logo-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, #7cb342 0%, #558b2f 100%);
  border-radius: 20px;
  margin-bottom: 16px;
  box-shadow: 0 8px 20px rgba(124, 179, 66, 0.3);
  animation: logoBounce 2s ease-in-out infinite;
}

@keyframes logoBounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
}

.logo-title {
  font-size: 1.75rem;
  font-weight: 700;
  color: #2e7d32;
  margin-bottom: 8px;
  text-shadow: 0 2px 4px rgba(46, 125, 50, 0.1);
}

.logo-subtitle {
  font-size: 0.9rem;
  color: #66bb6a;
  font-weight: 500;
}

/* 表单区域 */
.form-area {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-label {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.875rem;
  font-weight: 600;
  color: #37474f;
}

.label-icon {
  font-size: 1rem;
}

.form-hint {
  font-size: 0.75rem;
  color: #66bb6a;
  margin-top: 4px;
}

/* 消息提示 */
.message {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  border-radius: 12px;
  font-size: 0.875rem;
}

.message-icon {
  font-size: 1rem;
}

.error-message {
  background: linear-gradient(135deg, #ffebee 0%, #ffcdd2 100%);
  color: #c62828;
  border: 1px solid #ef9a9a;
}

.success-message {
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  color: #2e7d32;
  border: 1px solid #a5d6a7;
}

/* 提交按钮 */
.submit-btn {
  margin-top: 8px;
  height: 48px;
  font-size: 1rem;
  font-weight: 600;
  border-radius: 12px;
  background: linear-gradient(135deg, #7cb342 0%, #558b2f 100%);
  box-shadow: 0 4px 15px rgba(124, 179, 66, 0.4);
  transition: all 0.3s ease;
}

.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(124, 179, 66, 0.5);
}

.submit-btn:active {
  transform: translateY(0);
}

/* 切换区域 */
.switch-area {
  text-align: center;
  margin-top: 24px;
}

.switch-btn {
  background: none;
  border: none;
  color: #66bb6a;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  padding: 8px 16px;
  border-radius: 20px;
  transition: all 0.3s ease;
}

.switch-btn:hover {
  background: rgba(102, 187, 106, 0.1);
  color: #43a047;
}

/* 卡片底部 */
.card-footer {
  text-align: center;
  margin-top: 24px;
  padding-top: 20px;
  border-top: 1px solid rgba(102, 187, 106, 0.2);
  color: #81c784;
  font-size: 0.8rem;
}

/* 暗色模式适配 */
@media (prefers-color-scheme: dark) {
  .login-container {
    background: linear-gradient(180deg, #1a3a2a 0%, #1e4d2b 50%, #0d2818 100%);
  }

  .login-card {
    background: rgba(30, 60, 40, 0.95);
    box-shadow:
      0 20px 60px rgba(0, 0, 0, 0.4),
      0 0 0 1px rgba(102, 187, 106, 0.2);
  }

  .logo-title {
    color: #81c784;
  }

  .logo-subtitle {
    color: #66bb6a;
  }

  .form-label {
    color: #a5d6a7;
  }

  .card-footer {
    border-top-color: rgba(102, 187, 106, 0.3);
    color: #66bb6a;
  }
}

/* 响应式适配 */
@media (max-width: 480px) {
  .login-card {
    margin: 10px;
    padding: 30px 24px;
    border-radius: 20px;
  }

  .logo-icon {
    width: 70px;
    height: 70px;
  }

  .logo-title {
    font-size: 1.5rem;
  }

  .sun {
    width: 60px;
    height: 60px;
    top: 20px;
    right: 40px;
  }

  .plant {
    font-size: 1.5rem;
  }

  .plant-2,
  .plant-6 {
    font-size: 2rem;
  }
}
</style>
