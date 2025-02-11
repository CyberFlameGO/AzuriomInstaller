<template>
  <div>
    <h1 class="text-center">{{ $t('title') }}</h1>

    <div v-for="error in errors" :key="error" class="alert alert-danger" role="alert">
      {{ error }}

      <div v-if="error.startsWith('cURL error 60:')" v-html="$t('help.cUrl60', { path: phpIniPath ? phpIniPath : $t('unknown') })"/>
    </div>

    <transition name="fade" mode="out-in">
      <div v-if="preLoading" class="d-flex flex-column align-items-center justify-content-center" style="height: 300px">
        <div class="spinner-border spinner-border-lg text-primary mb-3"/>

        <h2>{{ $t('loading') }}</h2>
      </div>

      <requirements v-else-if="step === 'requirements'" @refresh="refreshRequirements"/>

      <download v-else-if="step === 'download'" @error="handleError"/>

      <database v-else-if="step === 'database'" @error="handleError"/>

      <config v-else-if="step === 'config'" @error="handleError"/>

      <installed v-else-if="step === 'installed'"/>
    </transition>

    <hr>

    <footer class="text-center">
      <button @click="setLocale('en')" class="btn btn-link p-0 mb-1 mx-1" title="English">
        <svg height="1.75rem" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 36 36">
          <path fill="#00247d" d="M0 9.059V13h5.628zM4.664 31H13v-5.837zM23 25.164V31h8.335zM0 23v3.941L5.63 23zM31.337 5H23v5.837zM36 26.942V23h-5.631zM36 13V9.059L30.371 13zM13 5H4.664L13 10.837z"/>
          <path fill="#cf1b2b" d="M25.14 23l9.712 6.801c.471-.479.808-1.082.99-1.749L28.627 23H25.14zM13 23h-2.141l-9.711 6.8c.521.53 1.189.909 1.938 1.085L13 23.943V23zm10-10h2.141l9.711-6.8c-.521-.53-1.188-.909-1.937-1.085L23 12.057V13zm-12.141 0L1.148 6.2C.677 6.68.34 7.282.157 7.949L7.372 13h3.487z"/>
          <path fill="#eee" d="M36 21H21v10h2v-5.836L31.335 31H32c1.117 0 2.126-.461 2.852-1.199L25.14 23h3.487l7.215 5.052c.093-.337.158-.686.158-1.052v-.058L30.369 23H36v-2zM0 21v2h5.63L0 26.941V27c0 1.091.439 2.078 1.148 2.8l9.711-6.8H13v.943l-9.914 6.941c.294.07.598.116.914.116h.664L13 25.163V31h2V21H0zM36 9c0-1.091-.439-2.078-1.148-2.8L25.141 13H23v-.943l9.915-6.942C32.62 5.046 32.316 5 32 5h-.663L23 10.837V5h-2v10h15v-2h-5.629L36 9.059V9zM13 5v5.837L4.664 5H4c-1.118 0-2.126.461-2.852 1.2l9.711 6.8H7.372L.157 7.949C.065 8.286 0 8.634 0 9v.059L5.628 13H0v2h15V5h-2z"/>
          <path fill="#cf1b2b" d="M21 15V5h-6v10H0v6h15v10h6V21h15v-6z"/>
        </svg>
      </button>

      <button @click="setLocale('fr')" class="btn btn-link p-0 mb-1 mx-1" title="Français">
        <svg height="1.75rem" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 36 36">
          <path fill="#ed2939" d="M36 27c0 2.209-1.791 4-4 4h-8V5h8c2.209 0 4 1.791 4 4v18z"/>
          <path fill="#002495" d="M4 5C1.791 5 0 6.791 0 9v18c0 2.209 1.791 4 4 4h8V5H4z"/>
          <path fill="#eee" d="M12 5h12v26H12z"/>
        </svg>
      </button>

      <p class="mb-0" v-html="$t('copyright', { year : new Date().getFullYear() })"></p>
    </footer>
  </div>
</template>

<script lang="ts">
import { AxiosError } from 'axios';
import { Component, Vue } from 'vue-property-decorator';
import { mapGetters, mapState } from 'vuex';
import Api from '@/services/Api';
import Requirements from '@/components/steps/Requirements.vue';
import Download from '@/components/steps/Download.vue';
import Database from '@/components/steps/Database.vue';
import Config from '@/components/steps/Config.vue';
import Installed from '@/components/steps/Installed.vue';

@Component({
  components: {
    Requirements, Database, Download, Config, Installed,
  },

  computed: {
    ...mapState(['loading', 'step']),
    ...mapGetters(['requirements', 'compatible', 'downloaded', 'errors', 'phpIniPath']),
  },
})
export default class AzuriomInstaller extends Vue {
  private preLoading = true;

  mounted() {
    this.refreshRequirements();
  }

  refreshRequirements() {
    this.$store.commit('startLoading');

    Api.fetch().then((response) => {
      this.$store.commit('updateData', response.data);
    }).catch((error) => {
      if (this.preLoading) {
        this.$store.commit('nextStep', 'error');
      }

      this.handleError(error);
    }).finally(() => {
      this.preLoading = false;

      this.$store.commit('finishLoading');
    });
  }

  handleError(error: AxiosError) {
    if (error.response && error.response.data && error.response.data.message) {
      this.addError(error.response.data.message);
      return;
    }

    this.addError(this.$t('error', { error }) as string);
  }

  addError(error: string) {
    this.$store.commit('addError', error);
  }

  setLocale(locale: string) {
    this.$i18n.locale = locale;
  }
}
</script>

<style lang="scss">
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.1s ease-in-out;
  }

  .fade-enter,
  .fade-leave-active {
    opacity: 0
  }

  .spinner-border-lg {
    height: 3rem;
    width: 3rem;
  }
</style>
