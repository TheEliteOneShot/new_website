<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useHead } from '@vueuse/head'
import { toFormValidator } from '@vee-validate/zod'
import { useUserSession } from '/@src/stores/userSession'
import { useForm } from 'vee-validate'
import { z as zod } from 'zod'
import { useApi } from '/@src/composable/useApi'

import { useDarkmode } from '/@src/stores/darkmode'
import { useNotyf } from '/@src/composable/useNotyf'
import { createUser } from '/@src/utils/api/user'

const api = useApi();
const darkmode = useDarkmode()
const router = useRouter()
const userSession = useUserSession();
const notif = useNotyf()

const isLoading = ref(false)

// Define a validation schema
const validationSchema = toFormValidator(
  zod
    .object({
      username: zod
        .string({
          required_error: "A username is required",
        })
        .nonempty('A username is required'),
      firstname: zod
        .string({
          required_error: 'A first name is required',
        })
        .nonempty('A first name is required'),
      lastname: zod
        .string({
          required_error: 'An email is required',
        })
        .nonempty('A last name is required')
      ,
      email: zod
        .string({
          required_error: 'An email is required',
        })
        .email('A valid email is required'),

      password: zod
        .string({
          required_error: 'A password is required',
        })
        .min(8, 'The password has to be at least 8 characters'),
      passwordCheck: zod.string({
        required_error: 'Retype the same password',
      })
    })
    .refine((data) => data.password === data.passwordCheck, {
      message: 'The passwords do not match',
      path: ['passwordCheck', 'password'],
    })
)

const { handleSubmit, setFieldError } = useForm({
  validationSchema,
  initialValues: {
    username: '',
    firstname: '',
    lastname: '',
    email: '',
    password: '',
    passwordCheck: ''
  },
})

const onSignup = handleSubmit(async (values) => {
  if (!isLoading.value) {
    isLoading.value = true
    notif.dismissAll()
    await createUser(api, { username: values.username, firstname: values.firstname, lastname: values.lastname, email: values.email, password: values.password })
      .then((tokens) => {
        notif.success('Successfully created account')
        userSession.setToken(tokens?.access_token);
        userSession.setRefreshToken(tokens?.refresh_token);
        router.push( {name: 'app'})
      })
      .catch((error: any) => {
        if (error?.response?.data?.detail === "USERNAME_TAKEN") {
          setFieldError("username", "This username has already been taken.")
        }
      });
    isLoading.value = false
  }
})

useHead({
  title: 'Create Account',
})
</script>

<template>
  <div class="auth-wrapper-inner columns is-gapless">
    <!-- Form section -->
    <div class="column is-5">
      <div class="hero is-fullheight is-white">
        <div class="hero-heading">
          <label class="dark-mode ml-auto" tabindex="0"
            @keydown.space.prevent="(e) => (e.target as HTMLLabelElement).click()">
            <input type="checkbox" :checked="!darkmode.isDark" @change="darkmode.onChange" />
            <span></span>
          </label>
          <div class="auth-logo">
            <RouterLink :to="{ name: 'index' }">
              <AnimatedLogo class="top-logo" width="36px" height="36px" />
            </RouterLink>
          </div>
        </div>
        <div class="hero-body">
          <div class="container">
            <div class="columns">
              <div class="column is-12">
                <div class="auth-content">
                  <h2>Create Account</h2>
                  <p>or <RouterLink :to="{ name: 'auth-login' }">
                      Login
                    </RouterLink>
                  </p>

                </div>
                <div class="auth-form-wrapper">
                  <!-- Login Form -->
                  <form @submit="onSignup">
                    <div id="signin-form" class="login-form">
                      <!-- Input -->
                      <VField id="firstname" v-slot="{ field }">
                        <VControl icon="feather:user">
                          <VInput type="text" placeholder="First Name" autocomplete="firstname" />
                          <p v-if="field?.errorMessage" class="help is-danger">
                            {{ field.errorMessage }}
                          </p>
                        </VControl>
                      </VField>
                      <!-- Input -->
                      <VField id="lastname" v-slot="{ field }">
                        <VControl icon="feather:user">
                          <VInput type="text" placeholder="Last Name" autocomplete="lastname" />
                          <p v-if="field?.errorMessage" class="help is-danger">
                            {{ field.errorMessage }}
                          </p>
                        </VControl>
                      </VField>

                      <!-- Input -->
                      <VField id="username" v-slot="{ field }">
                        <VControl icon="feather:user">
                          <VInput type="text" placeholder="Username" autocomplete="username" />
                          <p v-if="field?.errorMessage" class="help is-danger">
                            {{ field.errorMessage }}
                          </p>
                        </VControl>
                      </VField>

                      <!-- Input -->
                      <VField id="email" v-slot="{ field }">
                        <VControl icon="feather:mail">
                          <VInput type="text" placeholder="example@email.com" autocomplete="email" />
                          <p v-if="field?.errorMessage" class="help is-danger">
                            {{ field.errorMessage }}
                          </p>
                        </VControl>
                      </VField>

                      <!-- Input -->
                      <VField id="password" v-slot="{ field }">
                        <VControl icon="feather:lock">
                          <VInput type="password" placeholder="Password" autocomplete="new-password" />
                          <p v-if="field?.errorMessage" class="help is-danger">
                            {{ field.errorMessage }}
                          </p>
                        </VControl>
                      </VField>

                      <!-- Input -->
                      <VField id="passwordCheck" v-slot="{ field }">
                        <VControl icon="feather:lock">
                          <VInput type="password" placeholder="Retype password" />
                          <p v-if="field?.errorMessage" class="help is-danger">
                            {{ field.errorMessage }}
                          </p>
                        </VControl>
                      </VField>

                      <!-- Submit -->

                      <div class="login">
                        <VButton :loading="isLoading" type="submit" color="primary" bold fullwidth raised>
                          Create
                        </VButton>
                      </div>
                    </div>
                  </form>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>
