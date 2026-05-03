<script setup>
import { computed, onMounted, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { API_BASE_URL } from '@/utils/apiBase';

const route = useRoute()
const router = useRouter()

const invitation = ref(null)
const loading = ref(true)
const error = ref('')
const success = ref(false)
const form = ref({
    name: '',
    email: '',
    password: '',
    password_confirmation: ''
})

const token = computed(() => route.params.token)

const loadInvitation = async () => {
    try {
        const response = await fetch(`${API_BASE_URL}/invitations/${token.value}`)
        const data = await response.json()
        if (!response.ok) {
            throw new Error(data.message)
        }
        invitation.value = data
    } catch (e) {
        error.value = e.message
    } finally {
        loading.value = false
    }
}

const submitRegister = async () => {
    error.value = ''
    try {
        const response = await fetch(`${API_BASE_URL}/invitations/${token.value}/register`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Accept': 'application/json'
            },
            body: JSON.stringify(form.value)
        })
        const data = await response.json()
        if (!response.ok) {
            throw new Error(data.message)
        }
        success.value = true
    } catch (e) {
        error.value = e.message
    }
}

onMounted(loadInvitation)
</script>

<template>
    <div class="min-h-[calc(100vh-7rem)] flex items-center justify-center bg-gray-100 px-6">
        <div class="bg-white border rounded-xl shadow-xl p-8 w-full max-w-md">
            <h1 class="text-2xl font-bold text-gray-800 mb-6 text-center">Registro por invitación</h1>

            <div v-if="loading" class="text-center text-gray-600">Cargando invitación...</div>

            <div v-else-if="success" class="flex flex-col gap-y-5 text-center">
                <div class="text-sky-700 font-semibold">Cuenta creada correctamente.</div>
                <button @click="router.push({ name: 'login' })" class="bg-sky-600 text-white py-2 rounded-lg">
                    Ir a iniciar sesión
                </button>
            </div>

            <form v-else-if="invitation" @submit.prevent="submitRegister" class="flex flex-col gap-y-4">
                <div class="flex flex-col">
                    <label class="font-medium">Nombre:</label>
                    <input v-model="form.name" class="border rounded-lg p-2" required>
                </div>
                <div class="flex flex-col">
                    <label class="font-medium">Correo:</label>
                    <input v-model="form.email" type="email" class="border rounded-lg p-2" required>
                </div>
                <div class="flex flex-col">
                    <label class="font-medium">Contraseña:</label>
                    <input v-model="form.password" type="password" class="border rounded-lg p-2" required>
                </div>
                <div class="flex flex-col">
                    <label class="font-medium">Confirmar contraseña:</label>
                    <input v-model="form.password_confirmation" type="password" class="border rounded-lg p-2" required>
                </div>

                <div v-if="error" class="text-red-600 text-sm">{{ error }}</div>

                <button type="submit" class="bg-sky-600 text-white font-semibold py-2 rounded-lg hover:bg-sky-700 transition">
                    Crear cuenta
                </button>
            </form>

            <div v-else class="text-red-600 text-center">{{ error }}</div>
        </div>
    </div>
</template>
