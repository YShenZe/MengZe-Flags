<template>
  <v-app>
    <v-main>
      <FlagsGenerator />
      <About about-text="          本工具旨在帮助Minecraft服务器管理员快速生成优化的JVM启动参数，基于社区最佳实践和性能调优指南。 本站开发者为梦泽，开源地址：https://github.com/YShenZe/MengZe-Flags QQ群：994827275，欢迎加群一同交流！"/>
    </v-main>
  </v-app>
</template>

<script setup>
import FlagsGenerator from './components/FlagsGenerator.vue'
import About from './components/About.vue'
</script>

<style>
.v-card-title {
  font-size: 1.5rem;
  font-weight: bold;
  padding: 20px;
}
</style>