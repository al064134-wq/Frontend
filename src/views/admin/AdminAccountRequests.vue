<script setup>
import { ref, onMounted } from 'vue';
import { fetchData } from '@/utils/api';
import AdminSidebar from '@/components/admin/AdminSidebar.vue';

const requests = ref([])

const fetchRequests = async () => {
    requests.value = await fetchData('account-requests/pending', 'GET')
}

const acceptRequest = async (id) => {
    await fetchData(`account-requests/${id}/accept`, 'POST')
    await fetchRequests()
}

const rejectRequest = async (id) => {
    await fetchData(`account-requests/${id}/reject`, 'POST')
    await fetchRequests()
}

onMounted(fetchRequests)
</script>

<template>
    <div class="flex flex-row h-[calc(100vh-10rem)]">
        <AdminSidebar />

        <div class="flex flex-col flex-1 px-16 pt-12 gap-y-8 overflow-auto">
            <div class="text-3xl font-semibold">Solicitudes de cuenta</div>

            <div class="flex flex-col gap-y-4 max-w-4xl">
                <div v-for="request in requests" :key="request.id" class="border rounded-lg p-5 flex flex-col gap-y-3">
                    <div class="flex justify-between gap-x-6">
                        <div>
                            <div class="text-lg font-semibold">{{ request.name }}</div>
                            <div class="text-gray-600">{{ request.email }}</div>
                        </div>
                        <div class="text-sm text-gray-500">{{ request.created_at }}</div>
                    </div>

                    <p class="text-gray-700">{{ request.description }}</p>

                    <div class="flex gap-x-3">
                        <button @click="acceptRequest(request.id)" class="bg-green-600 text-white px-4 py-2 rounded-lg">
                            Aceptar
                        </button>
                        <button @click="rejectRequest(request.id)" class="bg-red-600 text-white px-4 py-2 rounded-lg">
                            Rechazar
                        </button>
                    </div>
                </div>

                <div v-if="requests.length === 0" class="text-gray-600">No hay solicitudes pendientes.</div>
            </div>
        </div>
    </div>
</template>
