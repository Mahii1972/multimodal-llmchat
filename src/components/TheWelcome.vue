<script setup lang="ts">
import WelcomeItem from './WelcomeItem.vue'
import DocumentationIcon from './icons/IconDocumentation.vue'
import ToolingIcon from './icons/IconTooling.vue'
import EcosystemIcon from './icons/IconEcosystem.vue'
import CommunityIcon from './icons/IconCommunity.vue'
import SupportIcon from './icons/IconSupport.vue'

const openReadmeInEditor = () => fetch('/__open-in-editor?file=README.md')
</script>

<template>
  <div class="space-y-6 max-w-3xl mx-auto">
    <WelcomeItem>
      <template #icon>
        <DocumentationIcon class="w-8 h-8 text-blue-500" />
      </template>
      <template #heading>Documentation</template>
      <p class="text-gray-700">
        Vue’s
        <a
          href="https://vuejs.org/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >official documentation</a
        >
        provides you with all information you need to get started.
      </p>
    </WelcomeItem>

    <WelcomeItem>
      <template #icon>
        <ToolingIcon class="w-8 h-8 text-green-500" />
      </template>
      <template #heading>Tooling</template>
      <p class="text-gray-700">
        This project is served and bundled with
        <a
          href="https://vite.dev/guide/features.html"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Vite</a
        >. The recommended IDE setup is
        <a
          href="https://code.visualstudio.com/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >VSCode</a
        >
        +
        <a
          href="https://github.com/vuejs/language-tools"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Vue - Official</a
        >. If you need to test your components and web pages, check out
        <a
          href="https://vitest.dev/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Vitest</a
        >
        and
        <a
          href="https://www.cypress.io/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Cypress</a
        >
        /
        <a
          href="https://playwright.dev/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Playwright</a
        >.
      </p>
      <p class="mt-2 text-gray-700">
        More instructions are available in
        <a
          href="javascript:void(0)"
          @click="openReadmeInEditor"
          class="text-blue-600 hover:text-blue-800"
          ><code>README.md</code></a
        >.
      </p>
    </WelcomeItem>

    <WelcomeItem>
      <template #icon>
        <EcosystemIcon class="w-8 h-8 text-purple-500" />
      </template>
      <template #heading>Ecosystem</template>
      <p class="text-gray-700">
        Get official tools and libraries for your project:
        <a
          href="https://pinia.vuejs.org/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Pinia</a
        >,
        <a
          href="https://router.vuejs.org/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Vue Router</a
        >,
        <a
          href="https://test-utils.vuejs.org/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Vue Test Utils</a
        >, and
        <a
          href="https://github.com/vuejs/devtools"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Vue Dev Tools</a
        >. If you need more resources, we suggest paying
        <a
          href="https://github.com/vuejs/awesome-vue"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Awesome Vue</a
        >
        a visit.
      </p>
    </WelcomeItem>

    <WelcomeItem>
      <template #icon>
        <CommunityIcon class="w-8 h-8 text-orange-500" />
      </template>
      <template #heading>Community</template>
      <p class="text-gray-700">
        Got stuck? Ask your question on
        <a
          href="https://chat.vuejs.org"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >Vue Land</a
        >
        (our official Discord server), or
        <a
          href="https://stackoverflow.com/questions/tagged/vue.js"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >StackOverflow</a
        >. You should also follow the official
        <a
          href="https://bsky.app/profile/vuejs.org"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >@vuejs.org</a
        >
        Bluesky account or the
        <a
          href="https://x.com/vuejs"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >@vuejs</a
        >
        X account for latest news in the Vue world.
      </p>
    </WelcomeItem>

    <WelcomeItem>
      <template #icon>
        <SupportIcon class="w-8 h-8 text-red-500" />
      </template>
      <template #heading>Support Vue</template>
      <p class="text-gray-700">
        As an independent project, Vue relies on community backing for its sustainability. You can
        help us by
        <a
          href="https://vuejs.org/sponsor/"
          target="_blank"
          rel="noopener"
          class="text-blue-600 hover:text-blue-800"
          >becoming a sponsor</a
        >.
      </p>
    </WelcomeItem>
  </div>
</template>
