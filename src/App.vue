<script lang="ts">
import { defineComponent } from 'vue'
import { loadClerkJsScript } from '@clerk/shared/loadClerkJsScript'
import type { HeadlessBrowserClerk, BrowserClerk } from './types'
import type { ClientResource, Resources } from '@clerk/types'

export default defineComponent({
  data() {
    return {
      isLoading: false,
      clerk: null as HeadlessBrowserClerk | BrowserClerk | null,
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
      this.isLoading = true

      await loadClerkJsScript({
        publishableKey: import.meta.env.VITE_CLERK_PUBLISHABLE_KEY,
      })

      this.isLoading = false

      if (!window.Clerk) {
        throw new Error('Clerk script failed to load');
      }

      this.clerk = window.Clerk
      await this.clerk.load()

      if (!this.clerk.user) {
        this.clerk.mountSignIn(this.$refs.signIn as HTMLDivElement)
      }

      this.clerk.addListener((payload) => {
        this.resources = payload;
      });
    },
    async signOut() {
      await this.clerk?.signOut()
    }
  }
})
</script>

<template>
  <div>
    <div v-if="isLoading">Loading Clerk...</div>
    <div v-else-if="resources.user">
      <p>Signed in as {{ resources.user?.id }}</p>
      <button @click="signOut">Sign out</button>
    </div>
    <div v-else ref="signIn"></div>
  </div>
</template>