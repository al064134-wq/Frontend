<script setup>
import { ref, onMounted } from 'vue';
import { fetchData } from '@/utils/api';
import AdminSidebar from '@/components/admin/AdminSidebar.vue';

const invitations = ref([])
const generatedUrl = ref('')
const message = ref('')

const getPublicInvitationUrl = (url) => {
    try {
        const invitationUrl = new URL(url, window.location.origin)
        return `${window.location.origin}${invitationUrl.pathname}${invitationUrl.search}${invitationUrl.hash}`
    } catch {
        return url
    }
}

const fetchInvitations = async () => {
    const pendingInvitations = await fetchData('invitations/pending', 'GET')
    invitations.value = pendingInvitations.map((invitation) => ({
        ...invitation,
        url: getPublicInvitationUrl(invitation.url)
    }))
}

const createInvitation = async () => {
    message.value = ''
    generatedUrl.value = ''
    try {
        const response = await fetchData('invitations', 'POST', { role_id: 2 })
        generatedUrl.value = getPublicInvitationUrl(response.url)
    } catch (error) {
        if (error.code == 409) {
            message.value = error.message
        } else {
            throw error
        }
    }
    await fetchInvitations()
}

const copyUrl = async (url) => {
    await navigator.clipboard.writeText(url)
    message.value = 'Link copiado'
}

const cancelInvitation = async (id) => {
    await fetchData(`invitations/${id}`, 'DELETE')
    await fetchInvitations()
}

onMounted(fetchInvitations)
</script>

<template>
    <div class="flex flex-row h-[calc(100vh-10rem)]">
        <AdminSidebar />

        <div class="flex flex-col flex-1 px-16 pt-12 gap-y-8 overflow-auto">
            <div class="text-3xl font-semibold">Invitaciones</div>

            <form @submit.prevent="createInvitation" class="flex flex-col gap-y-4 max-w-xl">
                <button
                    type="submit"
                    :disabled="invitations.length > 0"
                    class="bg-sky-600 text-white font-semibold py-2 rounded-lg hover:bg-sky-700 transition disabled:bg-gray-400 disabled:cursor-not-allowed">
                    Generar invitación
                </button>
                <div v-if="invitations.length > 0" class="text-sm text-gray-600">
                    Ya existe una invitación pendiente. Cancélala o espera a que se registre el usuario para crear otra.
                </div>
            </form>

            <div v-if="generatedUrl" class="flex flex-col gap-y-3 max-w-3xl">
                <div class="font-semibold text-sky-700">URL generada</div>
                <div class="flex gap-x-3">
                    <input :value="generatedUrl" class="border rounded-lg p-2 flex-1 bg-gray-100" readonly>
                    <button @click="copyUrl(generatedUrl)" class="bg-blue-600 text-white px-4 py-2 rounded-lg">
                        Copiar
                    </button>
                </div>
            </div>

            <div v-if="message" class="text-sky-700 font-semibold">{{ message }}</div>

            <div class="flex flex-col gap-y-3">
                <div class="text-xl font-semibold">Invitaciones pendientes</div>

                <div v-for="invitation in invitations" :key="invitation.id" class="flex flex-col gap-y-2 border rounded-lg p-4 max-w-3xl">
                    <div class="font-semibold">Link de registro activo</div>
                    <div class="text-sm text-gray-600">Expira: {{ invitation.expires_at }}</div>
                    <div class="flex gap-x-3">
                        <input :value="invitation.url" class="border rounded-lg p-2 flex-1 bg-gray-100" readonly>
                        <button @click="copyUrl(invitation.url)" class="bg-blue-600 text-white px-4 py-2 rounded-lg">
                            Copiar
                        </button>
                        <button @click="cancelInvitation(invitation.id)" class="bg-red-600 text-white px-4 py-2 rounded-lg">
                            Cancelar
                        </button>
                    </div>
                </div>

                <div v-if="invitations.length === 0" class="text-gray-600">No hay invitaciones pendientes.</div>
            </div>
        </div>
    </div>
</template>
