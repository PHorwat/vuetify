<template>
  <ExamplesUsageExample
    v-model="model"
    :code="code"
    :name="name"
    :options="options"
    :script="script"
  >
    <v-card class="my-n9" title="Activity log" border flat>
      <v-divider></v-divider>
      <template v-slot:text>
        <v-text-field v-model="keys" label="Hotkey command" variant="outlined"></v-text-field>

        <v-sheet
          class="pa-2 d-flex flex-column ga-2 overflow-y-auto"
          color="surface-light"
          height="212"
          rounded
        >
          <v-empty-state v-if="!logs.length" icon="mdi-calendar" text="No Events"></v-empty-state>
          <div
            v-for="(log, i) in logs.slice().reverse()"
            :key="i"
            class="log-entry pa-2 border rounded"
          >
            <span class="text-caption text-grey">{{ log.timestamp }}</span>
            <span class="ml-2">{{ log.message }}</span>
          </div>
        </v-sheet>
      </template>

      <template v-slot:actions>
        <span class="ps-2">Current hotkey:</span> <v-hotkey :display-mode="displayMode" :keys></v-hotkey>

        <v-spacer></v-spacer>

        <v-btn size="small" text="Clear" @click="logs = []"></v-btn>
      </template>
    </v-card>

    <template v-slot:configuration>
      <v-checkbox v-model="allowInputs" label="Allow inputs"></v-checkbox>
      <v-slider v-model="sequenceTimeout" label="Timeout"></v-slider>
    </template>
  </ExamplesUsageExample>
</template>

<script setup>
  // Utilities
  import { useHotkey } from 'vuetify'

  const name = 'hotkey'
  const model = ref('default')
  const options = ['text', 'symbol', 'icon']
  const logs = ref([])
  const keys = shallowRef('cmd+b')
  const binding = shallowRef(false)
  const allowInputs = shallowRef(false)
  const sequenceTimeout = shallowRef(0)

  const displayMode = computed(() => {
    return model.value === 'default' ? undefined : model.value
  })

  const and = computed(() => sequenceTimeout.value && allowInputs.value ? '\n  ' : '')

  const args = computed(() => {
    return sequenceTimeout.value || allowInputs.value ? `, {
  ${sequenceTimeout.value > 0 ? `sequenceTimeout: ${sequenceTimeout.value},` : ''}${allowInputs.value ? `${and.value}inputs: true,` : ''}
}` : ''
  })

  const code = `<template>
  <div>Hello world</div>
</template>`

  const script = computed(() => {
    return `<script setup>
  import { useHotkey } from 'vuetify'

  function onHotkey () {}

  const unwatch = useHotkey('${keys.value}', onHotkey${args.value})
<\\/script>`.replace('\\/', '/')
  })

  let unwatch = useHotkey(keys.value, onHotkey)
  let timeout = null

  watch(keys, val => {
    binding.value = true
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      unwatch?.()

      unwatch = useHotkey(val, onHotkey)

      binding.value = false
      logs.value.push({
        message: `⚙️ Hotkey updated`,
        timestamp: new Date().toLocaleTimeString(),
      })
    }, 500)
  })

  function onHotkey () {
    logs.value.push({
      message: `⌨️ Hotkey pressed`,
      timestamp: new Date().toLocaleTimeString(),
    })
  }
</script>
