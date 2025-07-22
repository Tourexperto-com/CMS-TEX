<template>
    <h1>Crear</h1>
</template>

<script setup>
import { useDynamicForm } from '~/composables/useDynamicForm'

const route = useRoute()
const router = useRouter()

const tablaSlug = route.query.tabla

const {
    formData,
    errors,
    selectOptions,
    loadingOptions,
    isSubmitting,
    tabla,
    badgeOptions,
    columnChunks,
    initializeFormData,
    loadSelectOptions,
    validateForm,
    prepareDataForSubmit
} = useDynamicForm(tablaSlug)

const botonTexto = computed(() => tabla?.botonTexto || 'Crear nuevo elemento')

const handleSubmit = async () => {
    if (!validateForm()) return

    isSubmitting.value = true

    try {
        const dataToSubmit = prepareDataForSubmit()
        console.log('Datos a enviar:', dataToSubmit)

        await router.push(`/tablas/${tablaSlug}`)

    } catch (error) {
        console.error('Error al crear:', error)
    } finally {
        isSubmitting.value = false
    }
}

onMounted(async () => {
    if (tabla) {
        initializeFormData()
        await loadSelectOptions()
    }
})
</script>