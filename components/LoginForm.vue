<template>
<form class="form form--login" ref="form" @submit.prevent="sendLoginForm" novalidate>
  <div class="form__group form__group--txt">
    <input class="form__input" type="email" name="email" v-model="email">
    <label class="form__label">{{ $t('userEmail') }}</label>
  </div>
  <div class="form__group form__group--txt">
    <input class="form__input" type="password" name="password" v-model="password">
    <label class="form__label">{{ $t('userPassword') }}</label>
  </div>
  <div class="form__group form__group--alert" v-if="alert">
    <span class="form__alert">{{ alert }}</span>
  </div>
  <div class="form__group form__group--submit">
    <button class="form__btn" type="submit" ref="submitBtn">{{ $t('userLogIn') }}</button>
  </div>
</form>
</template>

<script lang="ts">
import {
  Component,
  Vue,
  Prop
} from 'nuxt-property-decorator';
import axios from "axios";

@Component
export default class LoginForm extends Vue {
  public currentLocale = this.$i18n.locale;
  private email = '';
  private password = '';

  @Prop() global: any;
  @Prop() alert: any;

  sendLoginForm() {
    const creds = {
      identifier: this.email,
      password: this.password
    }

    this.$emit('login', creds);
  }

}
</script>

<style lang="scss" scoped>
@import "../assets/scss/components/_form";
@import "../assets/scss/components/_modal";
</style>
