<script lang="ts">
import { defineComponent } from 'vue'
import { loadClerkJsScript } from '@clerk/shared/loadClerkJsScript'
import type { HeadlessBrowserClerk, BrowserClerk } from './types'
import type { ClientResource, Resources } from '@clerk/types'

export default defineComponent({
  data() {
    return {
      clerk: null as HeadlessBrowserClerk | BrowserClerk | null,
      isAuthenticated: false,
      resources: {
        client: {} as ClientResource,
        session: undefined,
        user: undefined,
        organization: undefined
      } as Resources,
    }
  },
  mounted() {
    this.loadClerk()
  },
  methods: {
    async loadClerk() {
      await loadClerkJsScript({
        publishableKey: import.meta.env.VITE_CLERK_PUBLISHABLE_KEY,
      })

      if (!window.Clerk) {
        throw new Error('Clerk script failed to load');
      }

      this.clerk = window.Clerk
      await this.clerk.load()

      if (this.clerk.user) {
        this.isAuthenticated = true
      } else {
        this.clerk.mountSignIn(this.$refs.signIn as HTMLDivElement)
      }

      this.clerk.addListener((payload) => {
        this.resources = payload;
      });
    }
  }
})
</script>

<template>
  <div>
    <div v-if="isAuthenticated">
      <p>Signed in as {{ resources.user?.id }}</p>
    </div>
    <div v-else>
      <div ref="signIn"></div>
    </div>
  </div>
</template>