<template>
  <v-container class="pa-4">
    <v-card max-width="800" class="mx-auto" elevation="2">
      <v-card-title class="bg-primary text-white">
        <h2 class="text-h5">MengZe2 Flags Generator</h2>
      </v-card-title>

      <v-card-text class="pa-6">
        <!-- Memory Section -->
        <v-row>
          <v-col cols="12">
            <h3 class="text-h6 mb-4">内存配置</h3>
            <v-row align="center">
              <v-col cols="12" sm="4">
                <v-checkbox
                  v-model="syncMemory"
                  label="同步内存"
                  color="primary"
                  density="compact"
                  aria-label="Toggle synchronized memory"
                ></v-checkbox>
              </v-col>
              <v-col cols="12" sm="8">
                <v-btn-toggle
                  v-model="unit"
                  mandatory
                  density="compact"
                  color="primary"
                  class="mb-2"
                  stacked
                >
                  <v-btn value="GB">GB</v-btn>
                  <v-btn value="MB">MB</v-btn>
                </v-btn-toggle>
              </v-col>
            </v-row>
          </v-col>

          <v-col cols="12">
            <v-row align="center">
              <v-col cols="12" sm="6" v-if="syncMemory">
                <v-text-field
                  v-model.number="memoryDisplay"
                  label="内存"
                  type="number"
                  :min="unit === 'GB' ? 1 : 1024"
                  :max="unit === 'GB' ? 32 : 32768"
                  density="compact"
                  :suffix="unit"
                  variant="outlined"
                  hide-details="auto"
                  :rules="[v => (v >= (unit === 'GB' ? 1 : 1024) && v <= (unit === 'GB' ? 32 : 32768)) || '内存必须在有效范围内']"
                  @update:modelValue="validateMemory"
                >
                  <template #append>
                    <v-tooltip text="内存分配给服务器的总内存量">
                      <template #activator="{ props }">
                        <v-icon v-bind="props">mdi-information</v-icon>
                      </template>
                    </v-tooltip>
                  </template>
                </v-text-field>
              </v-col>
              <v-col cols="12" sm="6" v-else>
                <v-row align="center">
                  <v-col cols="5">
                    <v-text-field
                      v-model.number="minMemoryDisplay"
                      label="最小内存"
                      type="number"
                      :min="unit === 'GB' ? 1 : 1024"
                      :max="maxPossibleDisplay"
                      density="compact"
                      :suffix="unit"
                      variant="outlined"
                      hide-details="auto"
                      :rules="[v => (v >= (unit === 'GB' ? 1 : 1024) && v <= maxMemoryDisplay) || '最小内存无效']"
                      @update:modelValue="validateMinMemory"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="2" class="text-center">
                    <span class="text-body-1 font-weight-bold">至</span>
                  </v-col>
                  <v-col cols="5">
                    <v-text-field
                      v-model.number="maxMemoryDisplay"
                      label="最大内存"
                      type="number"
                      :min="minPossibleDisplay"
                      :max="unit === 'GB' ? 32 : 32768"
                      density="compact"
                      :suffix="unit"
                      variant="outlined"
                      hide-details="auto"
                      :rules="[v => (v >= minMemoryDisplay && v <= (unit === 'GB' ? 32 : 32768)) || '最大内存无效']"
                      @update:modelValue="validateMaxMemory"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-col>
            </v-row>
          </v-col>

          <v-col cols="12">
            <v-slider
              v-if="syncMemory"
              v-model="memoryDisplay"
              :min="unit === 'GB' ? 1 : 1024"
              :max="unit === 'GB' ? 32 : 32768"
              :step="unit === 'GB' ? 1 : 256"
              thumb-label="focus"
              :color="memoryMB < 4096 ? 'error' : 'primary'"
            >
              <template #thumb-label="{ modelValue }">
                {{ modelValue }}{{ unit }}
              </template>
            </v-slider>
          </v-col>

          <v-col cols="12" v-if="(syncMemory ? memoryMB : maxMemoryMB) < 4096">
            <v-alert type="warning" density="compact" variant="tonal">
              建议至少分配4GB（4096MB）内存给服务器！
            </v-alert>
          </v-col>
        </v-row>

        <v-divider class="my-4"></v-divider>

        <!-- Garbage Collector Section -->
        <v-row>
          <v-col cols="12">
            <v-radio-group
              v-model="gcType"
              label="垃圾回收器类型"
              aria-label="Select garbage collector type"
            >
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
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12">
            <v-checkbox
              v-model="aikarFlags"
              label="启用 Aikar's 优化参数"
              color="primary"
              aria-label="Enable Aikar's optimization flags"
            ></v-checkbox>
          </v-col>
        </v-row>

        <v-divider class="my-4"></v-divider>

        <!-- Advanced Options -->
        <v-row>
          <v-col cols="12">
            <v-expansion-panels>
              <v-expansion-panel>
                <v-expansion-panel-title>高级选项</v-expansion-panel-title>
                <v-expansion-panel-text>
                  <p class="text-body-2 mb-4">高级设置适用于优化JVM性能，谨慎修改。</p>
                  <v-row>
                    <v-col cols="12">
                      <v-checkbox
                        v-model="advanced.useStringDeduplication"
                        label="启用字符串去重 (-XX:+UseStringDeduplication)"
                        aria-label="Enable string deduplication"
                      >
                        <template #append>
                          <v-tooltip text="减少字符串对象的内存占用">
                            <template #activator="{ props }">
                              <v-icon v-bind="props">mdi-information</v-icon>
                            </template>
                          </v-tooltip>
                        </template>
                      </v-checkbox>
                    </v-col>
                    <v-col cols="12">
                      <v-checkbox
                        v-model="advanced.parallelGCThreads"
                        label="自定义并行GC线程数"
                        aria-label="Enable custom parallel GC threads"
                      ></v-checkbox>
                    </v-col>
                    <v-col cols="12" sm="6" v-if="advanced.parallelGCThreads">
                      <v-text-field
                        v-model.number="advanced.threadCount"
                        type="number"
                        label="GC线程数"
                        min="1"
                        max="32"
                        suffix="线程"
                        variant="outlined"
                        hide-details="auto"
                        :rules="[v => (v >= 1 && v <= 32) || '线程数必须在1到32之间']"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-expansion-panel-text>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-col>
        </v-row>

        <v-divider class="my-4"></v-divider>

        <!-- Generated Flags -->
        <v-row>
          <v-col cols="12">
            <v-textarea
              v-model="generatedFlags"
              label="生成参数"
              readonly
              disabled
              auto-grow
              rows="3"
              variant="outlined"
              hint="复制后粘贴到服务器启动脚本中"
              persistent-hint
              aria-label="Generated JVM flags"
            ></v-textarea>
          </v-col>
          <v-col cols="12">
            <v-btn
              color="primary"
              variant="flat"
              :loading="copying"
              :disabled="!generatedFlags"
              @click="copyToClipboard"
              aria-label="Copy generated flags to clipboard"
            >
              <v-icon start icon="mdi-content-copy"></v-icon>
              复制参数
            </v-btn>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>

    <!-- Snackbar for Copy Feedback -->
    <v-snackbar v-model="snackbar" timeout="2000" color="success">
      参数已复制到剪贴板！
      <template #actions>
        <v-btn color="white" variant="text" @click="snackbar = false">关闭</v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useClipboard } from '@vueuse/core'

// Memory-related states
const syncMemory = ref(true)
const unit = ref('GB')
const memoryMB = ref(4096) // Default 4GB
const minMemoryMB = ref(4096)
const maxMemoryMB = ref(4096)

// Garbage collector options
const gcType = ref('G1GC')
const aikarFlags = ref(true)

// Advanced options
const advanced = ref({
  useStringDeduplication: false,
  parallelGCThreads: false,
  threadCount: 4
})

// Snackbar and copy state
const copying = ref(false)
const snackbar = ref(false)

// Display value computed properties
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

const minPossibleDisplay = computed(() => {
  return unit.value === 'GB' ? minMemoryMB.value / 1024 : minMemoryMB.value
})

const maxPossibleDisplay = computed(() => {
  return unit.value === 'GB' ? maxMemoryMB.value / 1024 : maxMemoryMB.value
})

// Input validation
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

// Sync state handling
watch(syncMemory, (newVal) => {
  if (newVal) {
    minMemoryMB.value = memoryMB.value
    maxMemoryMB.value = memoryMB.value
  }
})

// Memory value sync
watch(memoryMB, (newVal) => {
  if (syncMemory.value) {
    minMemoryMB.value = newVal
    maxMemoryMB.value = newVal
  }
})

// Unit change handling
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

// Generated flags
const generatedFlags = computed(() => {
  const flags = []
  
  // Memory parameters
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

  // Garbage collector
  flags.push(`-XX:+Use${gcType.value}`)

  // Aikar's flags
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

  // ZGC-specific flags
  if (gcType.value === 'ZGC') {
    flags.push('-XX:+ZUncommit', '-XX:+ZProactive')
  }

  // Advanced options
  if (advanced.value.useStringDeduplication) {
    flags.push('-XX:+UseStringDeduplication')
  }

  if (advanced.value.parallelGCThreads) {
    flags.push(`-XX:ParallelGCThreads=${advanced.value.threadCount}`)
  }

  return flags.join(' ')
})

// Copy functionality
const { copy } = useClipboard()
const copyToClipboard = async () => {
  copying.value = true
  await copy(generatedFlags.value)
  snackbar.value = true
  copying.value = false
}
</script>

<style scoped>
.v-card {
  margin: 0 auto;
}
</style>
