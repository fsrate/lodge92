<template>
<section class="main__section">
  <div class="container">
    <h2>Register</h2>
    <form class="form" @submit.prevent="sendForm" ref="form" v-on:change="processForm" novalidate>
      <div class="form__group" :class="{'form__group--error': $v.name.$dirty && $v.name.$error}">
        <span class="form__alert">{{ validationMessage('name') }}</span>
        <input class="form__input" type="text" name="name" v-model="$v.name.$model" @blur="$v.name.$touch()">
        <label class="form__label">Name</label>
      </div>
      <div class="form__group" :class="{'form__group--error': $v.email.$dirty && $v.email.$error}">
        <span class="form__alert">{{ validationMessage('email') }}</span>
        <input class="form__input" type="email" name="email" v-model.trim="$v.email.$model" @blur="$v.email.$touch()">
        <label class="form__label">E-mail</label>
      </div>
      <div class="form__group" :class="{'form__group--error': $v.password.$dirty && $v.password.$error}">
        <span class="form__alert">{{ validationMessage('password') }}</span>
        <input class="form__input" type="password" name="password" v-model.trim="$v.password.$model" @blur="$v.password.$touch()">
        <label class="form__label">Password</label>
      </div>
      <div class="form__group form__group--alert">
        <span class="form__alert" v-if="alert" v-html="alert"></span>
      </div>
      <div class="form__group form__group--submit">
        <button class="form__btn" type="submit" ref="submitBtn">Register</button>
      </div>
    </form>
  </div>
</section>
</template>

<script lang="ts">
import {
  Component,
  Vue
} from 'nuxt-property-decorator';
import {
  validationMixin
} from 'vuelidate';
import {
  alpha,
  email,
  minLength,
  maxLength,
  required,
  helpers
} from 'vuelidate/lib/validators';

const alphaDiacritic = helpers.regex('alphaDiacritic', /^[a-zA-ZÀ-ž\s]*$/);

@Component({
  mixins: [validationMixin],
  validations: {
    name: {
      required,
      alphaDiacritic,
      minLength: minLength(3),
      maxLength: maxLength(50)
    },
    email: {
      required,
      email
    },
    password: {
      required,
      minLength: minLength(3),
      maxLength: minLength(3)
    }
  }
})
export default class HireMe extends Vue {
  private name = '';
  private email = '';
  private password = '';
  private alert = '';
  private validationMessages: {
    [key: string]: any
  } = {
    name: {
      required: "Please enter your name",
      alphaDiacritic: "Your name may contain letters only",
      minLength: "Your name may contain at least 3 letters",
      maxLength: "Your name may contain no more then 50 letters"
    },
    email: {
      required: "Please enter your e-mail address",
      email: "Please enter a valid e-mail address",
      maxLength: "Your e-mail address may contain no more then 50 letters"
    },
    password: {
      required: "Please enter your password",
      minLength: "Your password message may contain no less then 3 characters",
      maxLength: "Your password message may contain no more then 30 characters"
    }
  }

  created() {
    // console.dir((this as any).$strapi)
    // console.dir(this.$axios)
  }

  processForm() {
    for (let key in this.validationMessages) {
      this.$v[key].$touch();
      this.validationMessage(key);
    }
  }

  validationMessage(fieldName: string) {
    for (let key in this.validationMessages[fieldName]) {
      if (this.$v[fieldName].$dirty && this.$v[fieldName][key] as unknown == false)
        return this.validationMessages[fieldName][key];
    }
  }

  async sendForm() {
    if (this.$v.$invalid == true)
      return;

    // (this.$refs.submitBtn as HTMLButtonElement).disabled = true;

    const user = {
      email: this.$v.email.$model,
      username: this.$v.name.$model,
      password: this.$v.password.$model
    }

    try {
      (this as any).$axios.setToken(false);
      await (this as any).$axios.post("auth/local/register", user);
    } catch (e) {
      console.dir(e.response.data.message[0].messages[0].message);
    }

  }

  resetForm() {
    for (let key in this.validationMessages) {
      this.$v[key].$model = '';
    }
    this.$v.$reset();
    (this.$refs.submitBtn as HTMLButtonElement).disabled = false;
  }

}
</script>

<style lang="scss" scoped>
@import "../../assets/scss/components/_form";
</style>
