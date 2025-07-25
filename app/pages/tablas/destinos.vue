<template>
    <DefaultSection class="lg:!gap-8">
        <HeadingH1>Destinos</HeadingH1>
        <div class="w-full max-w-4xl grid grid-cols-1 md:grid-cols-3 gap-6">
            <div v-for="card in destinationCards" :key="card.id"
                class="flex flex-col items-center gap-4 bg-white rounded-lg shadow-sm border p-6">
                <div class="flex flex-col items-center gap-4 text-center">
                    <div :class="`w-16 h-16 flex items-center justify-center ${card.bgColor} rounded-full`">
                        <Icon :name="`tabler:${card.icon}`" :class="`w-8 h-8 ${card.iconColor}`" />
                    </div>
                    <HeadingH2>{{ card.title }}</HeadingH2>
                    <ButtonPrimary @click="card.action" class="w-full">
                        {{ card.buttonText }}
                    </ButtonPrimary>
                </div>
            </div>
        </div>

        <div class="w-full max-w-2xl">
            <div class="flex flex-col gap-1">
                <FormLabel id="search">Busca un destino</FormLabel>
                <div class="relative">
                    <FormTextField id="search" v-model="searchQuery" type="text" placeholder="Europa" @input="handleSearch" />
                    <div class="flex items-center absolute top-1/2 right-0 transform -translate-y-1/2 pr-3">
                        <Icon name="tabler:search" class="w-5 h-5 text-gray-dark" />
                    </div>
                </div>
            </div>

            <!-- REVISAR -->
            <div v-if="searchQuery && searchResults.length > 0"
                class="bg-white border border-gray-200 rounded-lg shadow-sm max-h-64 overflow-y-auto mt-4">
                <div v-for="result in searchResults" :key="`${result.type}-${result.id}`"
                    class="border-b border-gray-100 last:border-b-0 hover:bg-gray-50 transition-colors duration-200 cursor-pointer px-4 py-3"
                    @click="goToResult(result)">
                    <div class="flex justify-between items-center">
                        <div>
                            <span class="font-medium">{{ result.nombre }}</span>
                            <span class="ml-2 text-xs px-2 py-1 rounded"
                                :class="result.type === 'destino' ? 'bg-blue-100 text-blue-800' : 'bg-green-100 text-green-800'">
                                {{ result.type === 'destino' ? 'Destino' : 'Ciudad' }}
                            </span>
                        </div>
                        <div class="text-sm text-gray-500">
                            {{ result.type === 'destino' ? (result.regionId ? 'País' : 'Región') : `País ID:
                            ${result.paises_id}` }}
                        </div>
                    </div>
                </div>
            </div>

            <div v-else-if="searchQuery && searchResults.length === 0" class="bg-gray-50 rounded-lg mt-4 p-4">
                <p class="text-gray-500 text-center">No se encontraron resultados para "{{ searchQuery }}"</p>
            </div>
        </div>
    </DefaultSection>
</template>

<script setup>
import destinosData from '~/shared/destinos/destinos.js'
import ciudadesData from '~/shared/ciudades/ciudades.js'
import { ROUTE_NAMES } from '~/constants/ROUTE_NAMES'

const searchQuery = ref('')
const searchResults = ref([])

const handleSearch = () => {
    if (!searchQuery.value.trim()) {
        searchResults.value = []
        return
    }

    const query = searchQuery.value.toLowerCase()
    const results = []

    destinosData.forEach(destino => {
        if (destino.nombre.toLowerCase().includes(query) ||
            destino.txt_search.toLowerCase().includes(query)) {
            results.push({
                ...destino,
                type: 'destino'
            })
        }
    })

    ciudadesData.forEach(ciudad => {
        if (ciudad.nombre.toLowerCase().includes(query)) {
            results.push({
                ...ciudad,
                type: 'ciudad'
            })
        }
    })

    searchResults.value = results.slice(0, 10)
}

const goToResult = (result) => {
    if (result.type === 'destino') {
        if (result.regionId) {
            navigateTo(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.DESTINOS_PAISES}`)
        } else {
            navigateTo(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.DESTINOS_REGIONES}`)
        }
    } else {
        navigateTo(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.CIUDADES}`)
    }
}

const goToRegiones = () => {
    navigateTo(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.DESTINOS_REGIONES}`)
}

const goToPaises = () => {
    navigateTo(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.DESTINOS_PAISES}`)
}

const goToCiudades = () => {
    navigateTo(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.CIUDADES}`)
}

const destinationCards = [
    {
        id: 'regiones',
        title: 'Regiones',
        description: null,
        icon: 'world',
        bgColor: 'bg-blue-100',
        iconColor: 'text-blue-600',
        buttonText: 'Ver Regiones',
        action: goToRegiones
    },
    {
        id: 'paises',
        title: 'Países',
        description: 'Gestionar países y destinos nacionales',
        icon: 'map-pin',
        bgColor: 'bg-green-100',
        iconColor: 'text-green-600',
        buttonText: 'Ver Países',
        action: goToPaises
    },
    {
        id: 'ciudades',
        title: 'Ciudades',
        description: 'Gestionar ciudades y destinos urbanos',
        icon: 'building',
        bgColor: 'bg-purple-100',
        iconColor: 'text-purple-600',
        buttonText: 'Ver Ciudades',
        action: goToCiudades
    }
]
</script>
