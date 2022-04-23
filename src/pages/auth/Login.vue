<template>
    <div class ="row">
        <div class="col-12">
            <div class="surface-card p-4 shadow-2 border-round w-full">
                <div class="text-center mb-5">
                    <div class="text-900 text-3xl font-medium mb-3">Login to Vishleshan!</div>
                </div>
                <div>
                    <label for="email1" class="block text-900 font-medium mb-2">Email</label>
                    <InputText id="email1" type="text" class="w-full mb-3" v-model="v$.user.email.$model" />

                    <label for="password1" class="block text-900 font-medium mb-2">Password</label>
                    <InputText id="password1" type="password" class="w-full mb-3" v-model="v$.user.password.$model" />

                    <div class="flex align-items-center justify-content-between mb-6">
                        <router-link class="font-medium no-underline ml-2 text-blue-500 text-right cursor-pointer" :to="{ name: 'reset-password.generate-otp' }">Forgot password?</router-link>
                    </div>

                    <Button label="Sign In" icon="pi pi-user" class="w-full" v-on:click="handleLogin"></Button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { required, email, minLength } from '@vuelidate/validators'
import { useVuelidate } from '@vuelidate/core'
export default {
    name:"Login",
    data() {
        return {
            user: {
                email: '',
                password: '',
            },
        };
    },
    computed: {
        loggedIn() {
            return this.$store.state.auth.status.loggedIn;
        },
        checkUser() {
            return this.$store.state.auth.user;
        },
    },
    setup: () => ({ v$: useVuelidate() }),
    validations () {
        return {
          user: {
              email: { required, email },
              password: { required, min:minLength(6) },
          }
        }
    },
    created() {
        this.checkUserLogin();
    },
    methods: {
        checkUserLogin() {
            if (this.checkUser) {
                this.$router.push("/profile");
            }
        },
        handleLogin() {
            this.$store.dispatch("auth/login", this.user).then(
                () => {
                    this.$router.push("/profile");
                },
                (error) => {
                    this.loading = false;
                    this.message =
                    (error.response &&
                        error.response.data &&
                        error.response.data.message) ||
                        error.message ||
                        error.toString();
                    }
                );
            },
        },
    }
    </script>
