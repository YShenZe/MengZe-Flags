<template>
  <v-container>
    <v-card>
      <v-card-title class="bg-blue-darken-2 text-white">
        <HelloWorld msg="MengZe2 Flags Generator" />
      </v-card-title>

      <v-card-text>
        <div class="mb-6">
          <div class="d-flex align-center mb-4">
            <h3 class="text-h6" style="min-width: 80px">内存设置</h3>
            
            <v-checkbox
              v-model="syncMemory"
              label="同步分配"
              color="primary"
              density="compact"
              class="mr-4 ml-2"
            ></v-checkbox>

            <v-btn-toggle
              v-model="unit"
              mandatory
              density="compact"
              class="mr-4 elevation-1"
            >
              <v-btn value="GB" size="small" variant="flat">GB</v-btn>
              <v-btn value="MB" size="small" variant="flat">MB</v-btn>
            </v-btn-toggle>

            <div class="d-flex align-center">
              <template v-if="syncMemory">
                <v-text-field
                  v-model.number="memoryDisplay"
                  type="number"
                  :min="unit === 'GB' ? 1 : 1024"
                  :max="unit === 'GB' ? 32 : 32768"
                  density="compact"
                  style="width: 120px"
                  :suffix="unit"
                  variant="outlined"
                  hide-details
                  @update:modelValue="validateMemory"
                ></v-text-field>
              </template>
              
              <template v-else>
                <v-text-field
                  v-model.number="minMemoryDisplay"
                  type="number"
                  :min="unit === 'GB' ? 1 : 1024"
                  :max="maxPossibleDisplay"
                  density="compact"
                  style="width: 120px"
                  :suffix="unit"
                  variant="outlined"
                  hide-details
                  @update:modelValue="validateMinMemory"
                ></v-text-field>
                <span class="mx-2 text-caption">至</span>
                <v-text-field
                  v-model.number="maxMemoryDisplay"
                  type="number"
                  :min="minPossibleDisplay"
                  :max="unit === 'GB' ? 32 : 32768"
                  density="compact"
                  style="width: 120px"
                  :suffix="unit"
                  variant="outlined"
                  hide-details
                  @update:modelValue="validateMaxMemory"
                ></v-text-field>
              </template>
            </div>
          </div>

          <v-slider
            v-if="syncMemory"
            v-model="memoryDisplay"
            :min="unit === 'GB' ? 1 : 1024"
            :max="unit === 'GB' ? 32 : 32768"
            :step="unit === 'GB' ? 1 : 1024"
            thumb-label="always"
            thumb-size="28"
            track-color="grey-lighten-2"
            :color="memoryMB < 4096 ? 'orange-darken-2' : 'blue-darken-2'"
            class="mt-4"
          >
            <template #thumb-label="{ modelValue }">
              <span class="text-caption">{{ modelValue }}{{ unit }}</span>
            </template>
          </v-slider>

          <v-alert
            v-if="(syncMemory ? memoryMB : maxMemoryMB) < 4096"
            type="warning"
            density="compact"
            variant="tonal"
            class="mt-3"
          >
            <template #prepend>
              <v-icon icon="mdi-alert-circle-outline"></v-icon>
            </template>
            建议至少分配4GB（4096MB）内存给服务器！
          </v-alert>
        </div>

        <v-divider class="my-6"></v-divider>

        <v-radio-group
          v-model="gcType"
          label="垃圾回收器类型"
          class="mb-4"
          density="compact"
        >
          <v-radio
            value="G1GC"
            label="G1GC（Java 8+ 推荐）"
            color="blue-darken-2"
          ></v-radio>
          <v-radio
            value="ZGC"
            label="ZGC（Java 17+ 高性能）"
            color="blue-darken-2"
          ></v-radio>
          <v-radio
            value="Shenandoah"
            label="Shenandoah（低延迟）"
            color="blue-darken-2"
          ></v-radio>
        </v-radio-group>

        <v-checkbox
          v-model="aikarFlags"
          label="启用 Aikar's 优化参数"
          color="blue-darken-2"
          density="compact"
          class="mb-4"
        ></v-checkbox>

        <v-expansion-panels class="elevation-1">
          <v-expansion-panel>
            <v-expansion-panel-title expand-icon="mdi-chevron-down">
              <v-icon icon="mdi-tune" class="mr-2"></v-icon>
              高级选项
            </v-expansion-panel-title>
            <v-expansion-panel-text>
              <v-checkbox
                v-model="advanced.useStringDeduplication"
                label="启用字符串去重"
                color="blue-darken-2"
                density="compact"
                hint="-XX:+UseStringDeduplication"
                persistent-hint
              ></v-checkbox>
              
              <div class="d-flex align-center">
                <v-checkbox
                  v-model="advanced.parallelGCThreads"
                  label="自定义GC线程数"
                  color="blue-darken-2"
                  density="compact"
                  class="mr-4"
                ></v-checkbox>
                <v-text-field
                  v-if="advanced.parallelGCThreads"
                  v-model.number="advanced.threadCount"
                  type="number"
                  density="compact"
                  style="width: 150px"
                  variant="outlined"
                  min="1"
                  max="32"
                  suffix="线程"
                  hide-details
                ></v-text-field>
              </div>
            </v-expansion-panel-text>
          </v-expansion-panel>
        </v-expansion-panels>

        <v-textarea
          v-model="generatedFlags"
          label="生成参数"
          readonly
          auto-grow
          rows="3"
          class="mt-6"
          variant="outlined"
          bg-color="grey-lighten-4"
          persistent-hint
          hint="复制后粘贴到服务器启动脚本中"
        ></v-textarea>

        <div class="d-flex justify-end mt-4">
          <v-btn
            color="blue-darken-2"
            variant="flat"
            @click="copyToClipboard"
            :disabled="!generatedFlags"
            prepend-icon="mdi-content-copy"
          >
            复制参数
          </v-btn>
        </div>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useClipboard } from '@vueuse/core'
import HelloWorld from '/src/components/HelloWorld.vue'

// 内存相关状态
const syncMemory = ref(true)
const unit = ref('GB')
const memoryMB = ref(4096) // 默认4GB
const minMemoryMB = ref(4096)
const maxMemoryMB = ref(4096)

// 功能选项
const gcType = ref('G1GC')
const aikarFlags = ref(true)

// 高级选项
const advanced = ref({
  useStringDeduplication: false,
  parallelGCThreads: false,
  threadCount: 4
})

// 显示值计算属性
const memoryDisplay = computed({
  get() {
    return unit.value === 'GB' ? memoryMB.value / 1024 : memoryMB.value
  },
  set(value) {
    const numValue = Math.round(Number(value))
    memoryMB.value = unit.value === 'GB' 
      ? Math.max(1, Math.min(32, numValue)) * 1024
      : Math.max(1024, Math.min(32768, numValue))
  }
})

const minMemoryDisplay = computed({
  get() {
    return unit.value === 'GB' ? minMemoryMB.value / 1024 : minMemoryMB.value
  },
  set(value) {
    const numValue = Math.round(Number(value))
    const newVal = unit.value === 'GB' 
      ? Math.max(1, Math.min(32, numValue)) * 1024
      : Math.max(1024, Math.min(32768, numValue))
    
    minMemoryMB.value = Math.min(newVal, maxMemoryMB.value)
    if (newVal > maxMemoryMB.value) {
      maxMemoryMB.value = newVal
    }
  }
})

const maxMemoryDisplay = computed({
  get() {
    return unit.value === 'GB' ? maxMemoryMB.value / 1024 : maxMemoryMB.value
  },
  set(value) {
    const numValue = Math.round(Number(value))
    const newVal = unit.value === 'GB' 
      ? Math.max(1, Math.min(32, numValue)) * 1024
      : Math.max(1024, Math.min(32768, numValue))
    
    maxMemoryMB.value = Math.max(newVal, minMemoryMB.value)
    if (newVal < minMemoryMB.value) {
      minMemoryMB.value = newVal
    }
  }
})

// 输入验证
const validateMemory = (value) => {
  const numValue = Number(value)
  const min = unit.value === 'GB' ? 1 : 1024
  const max = unit.value === 'GB' ? 32 : 32768
  memoryDisplay.value = Math.min(Math.max(numValue, min), max)
}

const validateMinMemory = (value) => {
  const numValue = Number(value)
  const min = unit.value === 'GB' ? 1 : 1024
  const currentMax = unit.value === 'GB' 
    ? maxMemoryMB.value / 1024 
    : maxMemoryMB.value
  minMemoryDisplay.value = Math.min(
    Math.max(numValue, min),
    Math.min(currentMax, unit.value === 'GB' ? 32 : 32768)
  )
}

const validateMaxMemory = (value) => {
  const numValue = Number(value)
  const max = unit.value === 'GB' ? 32 : 32768
  const currentMin = unit.value === 'GB' 
    ? minMemoryMB.value / 1024 
    : minMemoryMB.value
  maxMemoryDisplay.value = Math.max(
    Math.min(numValue, max),
    Math.max(currentMin, unit.value === 'GB' ? 1 : 1024)
  )
}

// 状态同步处理
watch(syncMemory, (newVal) => {
  if (newVal) {
    minMemoryMB.value = memoryMB.value
    maxMemoryMB.value = memoryMB.value
  } else {
    memoryMB.value = minMemoryMB.value
  }
})

watch(memoryMB, (newVal) => {
  if (syncMemory.value) {
    minMemoryMB.value = newVal
    maxMemoryMB.value = newVal
  }
})

// 单位切换处理
watch(unit, (newUnit) => {
  const converter = newUnit === 'GB' ? 1/1024 : 1024
  if (syncMemory.value) {
    memoryDisplay.value = memoryMB.value * converter
  } else {
    minMemoryDisplay.value = minMemoryMB.value * converter
    maxMemoryDisplay.value = maxMemoryMB.value * converter
  }
})

// 参数生成
const generatedFlags = computed(() => {
  const flags = []
  
  // 内存参数
  if (syncMemory.value) {
    const value = unit.value === 'GB' 
      ? memoryMB.value / 1024 
      : memoryMB.value
    flags.push(`-Xms${value}${unit.value === 'GB' ? 'G' : 'M'}`)
    flags.push(`-Xmx${value}${unit.value === 'GB' ? 'G' : 'M'}`)
  } else {
    const xms = unit.value === 'GB' 
      ? minMemoryMB.value / 1024 
      : minMemoryMB.value
    const xmx = unit.value === 'GB' 
      ? maxMemoryMB.value / 1024 
      : maxMemoryMB.value
    flags.push(`-Xms${xms}${unit.value === 'GB' ? 'G' : 'M'}`)
    flags.push(`-Xmx${xmx}${unit.value === 'GB' ? 'G' : 'M'}`)
  }

  // 垃圾回收器
  flags.push(`-XX:+Use${gcType.value}`)

  // Aikar's 优化参数
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

  // ZGC 特殊参数
  if (gcType.value === 'ZGC') {
    flags.push('-XX:+ZUncommit', '-XX:+ZProactive')
  }

  // 高级选项
  if (advanced.value.useStringDeduplication) {
    flags.push('-XX:+UseStringDeduplication')
  }

  if (advanced.value.parallelGCThreads) {
    flags.push(`-XX:ParallelGCThreads=${advanced.value.threadCount}`)
  }

  return flags.join(' ')
})

// 复制功能
const { copy } = useClipboard()
const copyToClipboard = async () => {
  await copy(generatedFlags.value)
  alert('参数已复制到剪贴板！')
}
</script>

<style scoped>
.v-card {
  max-width: 800px;
  margin: 20px auto;
  border-radius: 12px;
  overflow: hidden;
}

.v-slider {
  min-width: 300px;
}

.v-text-field {
  font-feature-settings: "tnum";
  font-variant-numeric: tabular-nums;
}

.v-expansion-panel {
  border-radius: 8px !important;
}
</style>