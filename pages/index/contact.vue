<template>
<section class="main__section">
  <div class="container">
    <h2>{{ $t('contact') }}</h2>
  </div>
</section>
</template>

<script lang="ts">
import {
  Component,
  Watch,
  Vue
} from 'nuxt-property-decorator';

@Component
export default class Contact extends Vue {
  currentLocale = this.$i18n.locale;

  async asyncData({
    $strapi,
    app
  }: {
    $strapi: any,
    app: any
  }) {

    return {
      currentLocale: app.i18n.locale
    };
  }

}
</script>

<style scoped lang="scss">

</style>
