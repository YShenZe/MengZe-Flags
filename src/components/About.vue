<template>
  <v-container class="mt-4">
    <v-card class="about-card">
      <v-card-title class="bg-indigo-darken-3 text-white">
        <v-icon icon="mdi-information-outline" class="mr-2"></v-icon>
        关于本网站
      </v-card-title>

      <v-card-text class="pa-4">
        <div class="text-body-1 mb-6">
          {{ aboutText }}
        </div>

        <v-list density="compact" class="mb-4">
          <v-list-subheader>主要特性</v-list-subheader>
          <v-list-item
            v-for="(feature, i) in features"
            :key="i"
            :value="feature"
          >
            <template v-slot:prepend>
              <v-icon icon="mdi-check-circle-outline" color="success"></v-icon>
            </template>
            <v-list-item-title>{{ feature }}</v-list-item-title>
          </v-list-item>
        </v-list>

        <v-expansion-panels>
          <v-expansion-panel>
            <v-expansion-panel-title>
              <v-icon icon="mdi-code-tags" class="mr-2"></v-icon>
              技术栈
            </v-expansion-panel-title>
            <v-expansion-panel-text>
              <v-chip-group>
                <v-chip
                  v-for="(tech, index) in technologies"
                  :key="index"
                  variant="outlined"
                  color="primary"
                >
                  {{ tech }}
                </v-chip>
              </v-chip-group>
            </v-expansion-panel-text>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup>
import { defineProps } from 'vue'

const props = defineProps({
  aboutText: String,
})

const features = [
  '基于社区最佳实践的JVM参数推荐',
  '针对不同硬件配置的优化选项',
  '简单的图形化界面操作',
  '支持多种服务器类型'
]

const technologies = [
  'Vue3',
  'Vuetify',
  'Vite'
]
</script>
