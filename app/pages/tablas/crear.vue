<template>
    <DefaultSection>
        <HeadingH1>{{ getTituloForm() }}</HeadingH1>

        <FormDestinosCreate 
            v-if="isDestinosForm()" 
            :tipo="route.query.tipo"
            @success="handleSuccess"
            @cancel="handleCancel"
        />



        <FormProductosCreate 
            v-else-if="isProductosForm()" 
            @success="handleSuccess"
            @cancel="handleCancel"
        />

        <FormLayout @submit="handleSubmit" v-else-if="tabla" class="flex flex-col gap-5">
            <template v-for="(chunk, chunkIndex) in columnChunks" :key="`chunk-${chunkIndex}`">
                <FormFieldsContainer>
                    <template v-for="column in chunk" :key="column.key">
                        <FormTextField v-if="column.type === 'text'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required"
                            :error="errors[column.key]" :placeholder="`Ingresa ${column.label.toLowerCase()}`" />

                        <FormTextField v-else-if="column.type === 'number'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required"
                            :error="errors[column.key]" type="number"
                            :placeholder="`Ingresa ${column.label.toLowerCase()}`" />

                        <FormTextField v-else-if="column.type === 'date'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required"
                            :error="errors[column.key]" type="date" />

                        <FormTextField v-else-if="column.type === 'datetime'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required"
                            :error="errors[column.key]" type="datetime-local" />

                        <FormTextField v-else-if="column.type === 'currency'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required"
                            :error="errors[column.key]" type="number" step="0.01"
                            :placeholder="`Ingresa ${column.label.toLowerCase()}`" />

                        <FormSwitchField v-else-if="column.type === 'boolean'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required"
                            :error="errors[column.key]" />

                        <FormSelectField v-else-if="column.type === 'select'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required" 
                            :error="errors[column.key]" :options="selectOptions[column.key] || []" 
                            :loading="loadingOptions[column.key]" :placeholder="`Seleccionar ${column.label.toLowerCase()}`" />

                        <FormSelectField v-else-if="column.type === 'badge'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required" 
                            :error="errors[column.key]" :options="badgeOptions" 
                            :placeholder="`Seleccionar ${column.label.toLowerCase()}`" />

                        <FormImageField v-else-if="column.type === 'image'" :id="`field-${column.key}`"
                            v-model="formData[column.key]" :label="column.label" :required="column.required"
                            :error="errors[column.key]" />
                    </template>
                </FormFieldsContainer>
            </template>

            <!-- Botones de acción -->
            <div class="flex justify-center flex-wrap items-center gap-5 mt-8">
                <ButtonPrimary 
                    @click="handleCancel"
                    type="button"
                    class="!bg-gray-mid !text-dark"
                >
                    Cancelar
                </ButtonPrimary>
                
                <ButtonPrimary 
                    type="submit"
                    :disabled="isSubmitting"
                    class=""
                >
                    {{ isSubmitting ? 'Creando...' : 'Crear' }}
                </ButtonPrimary>
            </div>

        </FormLayout>

        <div v-else class="text-center py-8">
            <p class="text-dark">No se pudo cargar la configuración de la tabla.</p>
            <NuxtLink :to="ROUTE_NAMES.HOME" class="text-primary underline">
                Volver a la Home
            </NuxtLink>
        </div>
    </DefaultSection>
</template>

<script setup>
import { useDynamicForm } from '~/composables/useDynamicForm'
import { ButtonPrimary } from '#components'
import { ROUTE_NAMES } from '~/constants/ROUTE_NAMES'

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

const isDestinosForm = () => {
    return tablaSlug === 'destinos' || route.query.tipo === 'region' || route.query.tipo === 'pais'
}

const isProductosForm = () => {
    return tablaSlug === 'productos'
}

const getTituloForm = () => {
    if (isDestinosForm()) {
        const tipo = route.query.tipo
        if (tipo === 'region') return 'Crear nueva región'
        if (tipo === 'pais') return 'Crear nuevo país'
        return 'Crear nuevo destino'
    }
    if (isProductosForm()) return 'Crear nuevo producto'
    return botonTexto.value || 'Crear nuevo elemento'
}

const handleSuccess = () => {
    if (isDestinosForm()) {
        const tipo = route.query.tipo
        if (tipo === 'region') {
            router.push(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.DESTINOS_REGIONES}`)
        } else if (tipo === 'pais') {
            router.push(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.DESTINOS_PAISES}`)
        } else {
            router.push(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.DESTINOS}`)
        }
    } else if (isProductosForm()) {
        router.push(`${ROUTE_NAMES.TABLAS}${ROUTE_NAMES.PRODUCTOS}`)
    } else {
        router.push(`${ROUTE_NAMES.TABLAS}/${tablaSlug}`)
    }
}

const handleCancel = () => {
    handleSuccess()
}

const handleSubmit = async () => {
    if (!validateForm()) return

    isSubmitting.value = true

    try {
        const dataToSubmit = prepareDataForSubmit()
        // POST

        await router.push(`${ROUTE_NAMES.TABLAS}/${tablaSlug}`)

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