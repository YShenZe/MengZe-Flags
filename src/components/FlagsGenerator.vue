<template>
  <v-container>
    <v-card>
      <v-card-title class="bg-blue-darken-2 text-white">
        <HelloWorld msg="MengZe2 Flags Generator" />
      </v-card-title>

      <v-card-text>
        <div class="mb-6">
          <div class="d-flex align-center mb-2">
            <h3 class="text-h6 mr-6 mb-6 mt-6">内存</h3>
            <v-btn-toggle
              v-model="unit"
              mandatory
              density="compact"
              class="mr-4"
            >
              <v-btn value="GB" size="small">GB</v-btn>
              <v-btn value="MB" size="small">MB</v-btn>
            </v-btn-toggle>

            <v-text-field
              v-model.number="memoryInput"
              type="number"
              :min="minMemory"
              :max="maxMemory"
              density="compact"
              style="max-width: 120px"
              :suffix="unit"
              hide-details
              variant="outlined"
              @update:modelValue="handleMemoryInput"
            ></v-text-field>
          </div>

          <v-slider
            v-model="memory"
            :min="minMemory"
            :max="maxMemory"
            step="1"
            thumb-label="always"
            :color="memory < 4 ? 'red' : 'primary'"
          >
            <template #thumb-label="{ modelValue }">
              {{ modelValue }}{{ unit }}
            </template>
          </v-slider>

          <v-alert
            v-if="memory < 4"
            type="warning"
            density="compact"
            variant="tonal"
          >
            建议至少分配4GB（4096MB）内存给服务器！
          </v-alert>
        </div>

        <v-radio-group v-model="gcType" label="垃圾回收器类型" class="mb-4">
          <v-radio
            value="G1GC"
            label="G1GC（Java 8+ 推荐）"
            color="primary"
          ></v-radio>
          <v-radio
            value="ZGC"
            label="ZGC（Java 17+ 高性能）"
            color="primary"
          ></v-radio>
          <v-radio
            value="Shenandoah"
            label="Shenandoah（低延迟）"
            color="primary"
          ></v-radio>
        </v-radio-group>

        <v-checkbox
          v-model="aikarFlags"
          label="启用 Aikar's 优化参数"
          color="primary"
          class="mb-4"
        ></v-checkbox>

        <v-expansion-panels>
          <v-expansion-panel>
            <v-expansion-panel-title>高级选项</v-expansion-panel-title>
            <v-expansion-panel-text>
              <v-checkbox
                v-model="advanced.useStringDeduplication"
                label="启用字符串去重 (-XX:+UseStringDeduplication)"
              ></v-checkbox>
              <v-checkbox
                v-model="advanced.parallelGCThreads"
                label="自定义并行GC线程数"
              ></v-checkbox>
              <v-text-field
                v-if="advanced.parallelGCThreads"
                v-model.number="advanced.threadCount"
                type="number"
                label="GC线程数"
                min="1"
                max="32"
                suffix="线程"
              ></v-text-field>
            </v-expansion-panel-text>
          </v-expansion-panel>
        </v-expansion-panels>

        <v-textarea
          v-model="generatedFlags"
          label="生成参数"
          readonly
          auto-grow
          rows="3"
          class="mt-4"
          variant="outlined"
          :persistent-hint="true"
          hint="复制后粘贴到服务器启动脚本中"
        ></v-textarea>

        <v-btn
          color="green-darken-2"
          variant="flat"
          @click="copyToClipboard"
          :disabled="!generatedFlags"
          class="mt-2"
        >
          <v-icon start icon="mdi-content-copy"></v-icon>
          复制参数
        </v-btn>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useClipboard } from '@vueuse/core'
import HelloWorld from '/src/components/HelloWorld.vue'

const unit = ref('GB')
const memory = ref(4)
const memoryInput = ref(4)
const minMemory = ref(1)
const maxMemory = ref(32)
const gcType = ref('G1GC')
const aikarFlags = ref(true)
const { copy } = useClipboard()

const advanced = ref({
  useStringDeduplication: false,
  parallelGCThreads: false,
  threadCount: 4
})

const actualMemory = computed(() => {
  return unit.value === 'GB' ? memory.value : memory.value * 1024
})

watch(unit, (newUnit) => {
  const converter = newUnit === 'GB' ? 1/1024 : 1024
  memory.value = Math.round(memory.value * converter)
  memoryInput.value = memory.value
  minMemory.value = newUnit === 'GB' ? 1 : 1024
  maxMemory.value = newUnit === 'GB' ? 32 : 32768
})

const handleMemoryInput = (value) => {
  const numValue = Number(value)
  if (numValue < minMemory.value) {
    memory.value = minMemory.value
  } else if (numValue > maxMemory.value) {
    memory.value = maxMemory.value
  } else {
    memory.value = numValue
  }
}

const generatedFlags = computed(() => {
  const flags = [
    `-Xms${actualMemory.value}${unit.value === 'GB' ? 'G' : 'M'}`,
    `-Xmx${actualMemory.value}${unit.value === 'GB' ? 'G' : 'M'}`,
    `-XX:+Use${gcType.value}`
  ]

  if (aikarFlags.value) {
    flags.push(
      '-XX:+ParallelRefProcEnabled',
      '-XX:MaxGCPauseMillis=200',
      '-XX:+UnlockExperimentalVMOptions',
      '-XX:+DisableExplicitGC',
      '-XX:+AlwaysPreTouch',
      '-XX:G1NewSizePercent=30',
      '-XX:G1MaxNewSizePercent=40',
      '-XX:G1HeapRegionSize=8M',
      '-XX:G1ReservePercent=20',
      '-XX:G1HeapWastePercent=5',
      '-XX:G1MixedGCCountTarget=4',
      '-XX:InitiatingHeapOccupancyPercent=15',
      '-XX:G1MixedGCLiveThresholdPercent=90',
      '-XX:G1RSetUpdatingPauseTimePercent=5',
      '-XX:SurvivorRatio=32',
      '-XX:+PerfDisableSharedMem',
      '-XX:MaxTenuringThreshold=1',
      '-Dusing.aikars.flags=https://mcflags.emc.gs'
    )
  }

  if (gcType.value === 'ZGC') {
    flags.push('-XX:+ZUncommit', '-XX:+ZProactive')
  }

  if (advanced.value.useStringDeduplication) {
    flags.push('-XX:+UseStringDeduplication')
  }

  if (advanced.value.parallelGCThreads) {
    flags.push(`-XX:ParallelGCThreads=${advanced.value.threadCount}`)
  }

  return flags.join(' ')
})

const copyToClipboard = async () => {
  await copy(generatedFlags.value)
  alert('参数已复制到剪贴板！')
}
</script>

<style scoped>
.v-card {
  max-width: 800px;
  margin: 0 auto;
}
.v-slider {
  min-width: 300px;
}
</style>