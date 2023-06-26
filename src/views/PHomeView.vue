<template>
  <div class="container flex flex-col h-full items-center justify-center pt-[120px]">
    <div>
      <img src="/public/logo.svg" alt="">
    </div>
    <form @submit.prevent="onSubmit" class="p-8 bg-white rounded-xl w-[315px] mt-12">
      <p class="font-bold text-blue-400 text-2xl">Kirish</p>
      <div class="mt-11">
        <div class="flex flex-col">
          <label for="login" class="uppercase font-bold text-xs mb-2">Login</label>
          <input
              v-model="login"
              name="login"
              type="text"
              minlength="5"
              required
              placeholder="admin"
              class="bg-gray-200/20 text-[15px] rounded-md border-2 border-[#E0E7FF] outline-0 p-2 ring-0"
          />
        </div>
        <div class="flex flex-col mt-4">
          <label for="password" class="uppercase font-bold text-xs mb-2">Parol</label>
          <div
              class="flex justify-between items-center bg-gray-200/20 rounded-md border-2 border-[#E0E7FF] p-2 max-h-[42px]"
          >
            <input
                name="password"
                id="password"
                type="password"
                v-model="password"
                ref="inputPass"
                placeholder="********"
                minlength="8"
                required
                class="password-input text-[15px] w-full !bg-transparent outline-0"
            />
            <div class="hover:cursor-pointer" v-if="password" @click="togglePassword">
              <div v-if="!isVisible" class="flex">
                <i class="icon icon-eye text-2xl text-black"></i>
              </div>
              <div v-else class="flex">
                <i class="icon icon-eye-closed text-2xl text-black"></i>
              </div>
            </div>
          </div>
        </div>
        <div class="w-[300px] mt-5">
          <VueRecaptcha
              @verify="verify"
              aria-required="true"
              class="!scale-[0.83] !origin-[0_0]"
              sitekey="6LeWjrMmAAAAAH5tJmvHyNTslkTrXVDYKu0BB5Be"
          />
        </div>
        <VButton
            label="Kirish"
            class="w-full flex items-center justify-center hover:cursor-pointer"
            :class="{ disabled: !recaptchaToken }"
            :disabled="!recaptchaToken"
        >
          <div v-if="loading" class="spinner"></div>
        </VButton>
      </div>
    </form>
  </div>
</template>

<script setup lang="ts">
import {ref} from 'vue'
import {VueRecaptcha} from 'vue-recaptcha'
import axios from 'axios'
import VButton from '../components/Button/VButton.vue'
import {useAuthStore} from '@/stores'
import {useRouter} from 'vue-router'
import {useApi} from "@/helpers/axois";

const store = useAuthStore()
const router = useRouter()
let recaptchaToken = ref('')

function verify() {
  console.log(event)
  recaptchaToken.value = event?.data
}

const inputPass = ref(null)

const loading = ref(false)
const login = ref('')
const password = ref('')
const isVisible = ref(false)

async function onSubmit() {
  if (recaptchaToken.value !== '') {
    loading.value = true
    try {
      console.log('try')
      const res = await useApi.post('/auth/login/', {
        username: login.value,
        password: password.value
      })
      store.login({
        accessToken: res.access,
        refreshToken: res.refresh,
      })
      window.location.reload()
    } catch (error) {
      console.log(error)
    } finally {
      loading.value = false
    }
  }
}

function togglePassword() {
  isVisible.value = !isVisible.value
  const inputType = inputPass.value.type
  const changer = {
    password: 'text',
    text: 'password'
  }
  inputPass.value.type = changer[inputType]
}
</script>

<style scoped>
input:-webkit-autofill,
input:-webkit-autofill:hover,
input:-webkit-autofill:focus,
input:-webkit-autofill:active {
  transition: background-color 5000s ease-in-out 0s;
  -webkit-text-fill-color: #000 !important;
}

.spinner {
  position: absolute;
  backdrop-filter: blur(10px);
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: radial-gradient(farthest-side, #1f203b 94%, #0000) top/3.8px 3.8px no-repeat,
  conic-gradient(#0000 30%, #0d0d1a);
  -webkit-mask: radial-gradient(farthest-side, #0000 calc(100% - 3.8px), #000 0);
  animation: spinner-c7wet2 0.6s infinite linear;
}

@keyframes spinner-c7wet2 {
  100% {
    transform: rotate(1turn);
  }
}

.disabled {
  border: 1px solid transparent;
  cursor: not-allowed;
  background-color: #555555;
  color: white;
}
</style>