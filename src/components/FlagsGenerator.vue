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
            <v-checkbox
              v-model="syncMemory"
              label="同步"
              color="primary"
              density="compact"
              class="mr-2"
            ></v-checkbox>
            <v-btn-toggle
              v-model="unit"
              mandatory
              density="compact"
              class="mr-4"
            >
              <v-btn value="GB" size="small">GB</v-btn>
              <v-btn value="MB" size="small">MB</v-btn>
            </v-btn-toggle>

            <template v-if="syncMemory">
              <v-text-field
                v-model.number="memoryDisplay"
                type="number"
                :min="unit === 'GB' ? 1 : 1024"
                :max="unit === 'GB' ? 32 : 32768"
                density="compact"
                style="max-width: 120px"
                :suffix="unit"
                hide-details
                variant="outlined"
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
                style="max-width: 120px"
                :suffix="unit"
                hide-details
                variant="outlined"
                @update:modelValue="validateMinMemory"
              ></v-text-field>
              <span class="mx-2">至</span>
              <v-text-field
                v-model.number="maxMemoryDisplay"
                type="number"
                :min="minPossibleDisplay"
                :max="unit === 'GB' ? 32 : 32768"
                density="compact"
                style="max-width: 120px"
                :suffix="unit"
                hide-details
                variant="outlined"
                @update:modelValue="validateMaxMemory"
              ></v-text-field>
            </template>
          </div>

          <v-slider
            v-if="syncMemory"
            v-model="memoryDisplay"
            :min="unit === 'GB' ? 1 : 1024"
            :max="unit === 'GB' ? 32 : 32768"
            :step="unit === 'GB' ? 1 : 1024"
            thumb-label="always"
            :color="memoryMB < 4096 ? 'red' : 'primary'"
          >
            <template #thumb-label="{ modelValue }">
              {{ modelValue }}{{ unit }}
            </template>
          </v-slider>

          <v-alert
            v-if="(syncMemory ? memoryMB : maxMemoryMB) < 4096"
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

// 内存相关状态
const syncMemory = ref(true)
const unit = ref('GB')
const memoryMB = ref(4096) // 默认4GB
const minMemoryMB = ref(4096)
const maxMemoryMB = ref(4096)

// 垃圾回收器选项
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
    const numValue = Number(value)
    memoryMB.value = unit.value === 'GB' ? numValue * 1024 : numValue
  }
})

const minMemoryDisplay = computed({
  get() {
    return unit.value === 'GB' ? minMemoryMB.value / 1024 : minMemoryMB.value
  },
  set(value) {
    const numValue = Number(value)
    minMemoryMB.value = unit.value === 'GB' ? numValue * 1024 : numValue
    if (minMemoryMB.value > maxMemoryMB.value) {
      maxMemoryMB.value = minMemoryMB.value
    }
  }
})

const maxMemoryDisplay = computed({
  get() {
    return unit.value === 'GB' ? maxMemoryMB.value / 1024 : maxMemoryMB.value
  },
  set(value) {
    const numValue = Number(value)
    maxMemoryMB.value = unit.value === 'GB' ? numValue * 1024 : numValue
    if (maxMemoryMB.value < minMemoryMB.value) {
      minMemoryMB.value = maxMemoryMB.value
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
  const max = unit.value === 'GB' ? 32 : 32768
  minMemoryDisplay.value = Math.min(Math.max(numValue, min), Math.min(max, maxMemoryDisplay.value))
}

const validateMaxMemory = (value) => {
  const numValue = Number(value)
  const min = unit.value === 'GB' ? 1 : 1024
  const max = unit.value === 'GB' ? 32 : 32768
  maxMemoryDisplay.value = Math.max(Math.min(numValue, max), Math.max(min, minMemoryDisplay.value))
}

// 同步状态变化处理
watch(syncMemory, (newVal) => {
  if (newVal) {
    minMemoryMB.value = memoryMB.value
    maxMemoryMB.value = memoryMB.value
  }
})

// 内存值同步处理
watch(memoryMB, (newVal) => {
  if (syncMemory.value) {
    minMemoryMB.value = newVal
    maxMemoryMB.value = newVal
  }
})

// 单位变化处理
watch(unit, (newUnit) => {
  if (syncMemory.value) {
    memoryDisplay.value = newUnit === 'GB' 
      ? memoryMB.value / 1024 
      : memoryMB.value
  } else {
    minMemoryDisplay.value = newUnit === 'GB' 
      ? minMemoryMB.value / 1024 
      : minMemoryMB.value
    maxMemoryDisplay.value = newUnit === 'GB' 
      ? maxMemoryMB.value / 1024 
      : maxMemoryMB.value
  }
})

// 参数生成
const generatedFlags = computed(() => {
  const flags = []
  
  // 内存参数
  if (syncMemory.value) {
    const value = unit.value === 'GB' ? memoryMB.value / 1024 : memoryMB.value
    flags.push(`-Xms${value}${unit.value === 'GB' ? 'G' : 'M'}`)
    flags.push(`-Xmx${value}${unit.value === 'GB' ? 'G' : 'M'}`)
  } else {
    const xms = unit.value === 'GB' ? minMemoryMB.value / 1024 : minMemoryMB.value
    const xmx = unit.value === 'GB' ? maxMemoryMB.value / 1024 : maxMemoryMB.value
    flags.push(`-Xms${xms}${unit.value === 'GB' ? 'G' : 'M'}`)
    flags.push(`-Xmx${xmx}${unit.value === 'GB' ? 'G' : 'M'}`)
  }

  // 垃圾回收器
  flags.push(`-XX:+Use${gcType.value}`)

  // Aikar's 参数
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
  margin: 0 auto;
}
.v-slider {
  min-width: 300px;
}
.memory-warning {
  border-left: 4px solid #ff9800;
  padding-left: 12px;
  margin-top: 8px;
}
</style>