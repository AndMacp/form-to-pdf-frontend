<template>
  <div class="flex flex-col items-center space-y-20">
    <h3 class="text-2xl font-semibold">
      Enter data below, submit and download agreement
    </h3>

    <UForm
      :state="state"
      class="space-y-4 max-w-[250px] bg-gray-400 p-10 rounded"
      @submit="onSubmit"
    >
      <UFormGroup label="Name" name="name">
        <UInput v-model="state.name" class="bg-white rounded-lg" required />
      </UFormGroup>

      <UFormGroup label="Surname" name="surname">
        <UInput v-model="state.surname" class="bg-white rounded-lg" required />
      </UFormGroup>

      <UFormGroup label="Age" name="age">
        <UInput
          v-model="state.age"
          type="number"
          class="bg-white rounded-lg"
          required
        />
      </UFormGroup>

      <UFormGroup label="Country" name="country">
        <UInput v-model="state.country" class="bg-white rounded-lg" required />
      </UFormGroup>

      <UFormGroup label="City" name="city">
        <UInput v-model="state.city" class="bg-white rounded-lg" required />
      </UFormGroup>

      <UButton type="submit">Submit</UButton>
      <div v-if="downloadLinkActive">
        <p class="mb-4">Your download is ready</p>
        <UButton @click="openPdf">Download</UButton>
      </div>
    </UForm>
  </div>
</template>

<script setup lang="ts">
import type { State } from '~/types/types'
const toast = useToast()
const pdfLink = ref('')
const downloadLinkActive = ref(false)
const apiBaseUrl = 'https://form-to-pdf-backend-production-46e9.up.railway.app'

const state = ref<State>({
  name: '',
  surname: '',
  age: null,
  country: '',
  city: '',
})

async function onSubmit() {
  try {
    const usersData = (await $fetch(`${apiBaseUrl}/api/agreements`, {
      method: 'POST',
      body: {
        data: {
          name: state.value.name,
          surname: state.value.surname,
          age: state.value.age,
          country: state.value.country,
          city: state.value.city,
        },
      },
    })) as any

    const userId = usersData.data.documentId

    const getUserData = (await $fetch(
      `${apiBaseUrl}/api/agreements/${userId}?populate=*`
    )) as any

    pdfLink.value = `${apiBaseUrl}${getUserData.data.agreement[0].url}`

    toast.add({
      description:
        'Your data has been submitted, your download will be ready shortly',
    })

    await new Promise((resolve) => setTimeout(resolve, 2000))

    if (pdfLink.value != '') {
      downloadLinkActive.value = true
    }

    state.value = {
      name: '',
      surname: '',
      age: null,
      country: '',
      city: '',
    }
  } catch (err) {
    toast.add({
      description: 'There was an error submitting data, please try again later',
    })
  }
}

const openPdf = () => {
  window.open(pdfLink.value, '_blank')
}
</script>
