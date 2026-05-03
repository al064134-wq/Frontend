<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { API_BASE_URL } from '@/utils/apiBase';

const router = useRouter()
const error = ref('')
const success = ref(false)
const form = ref({
    name: '',
    email: '',
    password: '',
    password_confirmation: '',
    description: ''
})

const submitRequest = async () => {
    error.value = ''
    try {
        const response = await fetch(`${API_BASE_URL}/account-requests`, {
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
</script>

<template>
    <div class="min-h-[calc(100vh-7rem)] flex items-center justify-center bg-gray-100 px-4 py-4">
        <div class="bg-white border rounded-xl shadow-xl px-6 py-5 w-full max-w-md max-h-[calc(100vh-9rem)] overflow-auto">
            <h1 class="text-xl font-bold text-gray-800 mb-4 text-center">Petición de cuenta</h1>

            <div v-if="success" class="flex flex-col gap-y-4 text-center">
                <div class="text-sky-700 font-semibold">Tu petición fue enviada. Un administrador la revisará.</div>
                <button @click="router.push({ name: 'login' })" class="bg-sky-600 text-white py-1.5 rounded-lg">
                    Regresar al login
                </button>
            </div>

            <form v-else @submit.prevent="submitRequest" class="flex flex-col gap-y-2.5">
                <div class="flex flex-col">
                    <label class="font-medium text-sm">Nombre:</label>
                    <input v-model="form.name" class="border rounded-lg px-2 py-1.5" required>
                </div>
                <div class="flex flex-col">
                    <label class="font-medium text-sm">Correo:</label>
                    <input v-model="form.email" type="email" class="border rounded-lg px-2 py-1.5" required>
                </div>
                <div class="flex flex-col">
                    <label class="font-medium text-sm">Contraseña:</label>
                    <input v-model="form.password" type="password" class="border rounded-lg px-2 py-1.5" required>
                </div>
                <div class="flex flex-col">
                    <label class="font-medium text-sm">Confirmar contraseña:</label>
                    <input v-model="form.password_confirmation" type="password" class="border rounded-lg px-2 py-1.5" required>
                </div>
                <div class="flex flex-col">
                    <label class="font-medium text-sm">Descripción:</label>
                    <textarea v-model="form.description" class="border rounded-lg px-2 py-1.5 min-h-20" required></textarea>
                </div>

                <div v-if="error" class="text-red-600 text-sm">{{ error }}</div>

                <button type="submit" class="bg-sky-600 text-white font-semibold py-1.5 rounded-lg hover:bg-sky-700 transition">
                    Enviar petición
                </button>
            </form>
        </div>
    </div>
</template>
