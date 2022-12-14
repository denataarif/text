<template>
  <div>
    <div class="flex justify-center bg-white w-screen h-screen">
    <div
      class="flex flex-col items-center justify-center w-full hp:px-4 md:px-8"
    >
      <div class="md:!w-[60%]">
        <div class="hp:mt-5">
          <h1 class="text-4xl font-bold">Masuk</h1>
        </div>
        <p class="mt-4">
          Kami membantu orangtua yang ingin mengetahui <br />
          perkembangan anak
        </p>
        <div class="flex flex-col w-full mt-4">
          <label for="username">Email</label>
          <div
            :class="[emailCheck ? 'border border-red-600' : 'border']"
            class="flex mb-2 items-center rounded"
          >
            <img class="px-2" width="40px" src="~images/email.svg" alt="" />
            <input
              v-model="email"
              placeholder="Email"
              class="py-3 w-full focus:outline-none rounded"
              type="text"
            />
          </div>
          <p v-if="invalidLogin === true" class="text-red-600 text-sm mb-1">Invalid email or password</p>
          <label for="username">Password</label>
          <div
            :class="[passwordCheck ? 'border border-red-600' : 'border']"
            class="flex items-center rounded"
          >
            <img class="px-2" width="40px" src="~icon/ic-lock.svg" alt="" />
            <input
              v-model="password"
              placeholder="Password"
              class="py-3 w-full focus:outline-none rounded"
              :type="typePsd"
            />
            <img
              v-if="typePassword"
              src="~icon/eye-open.svg"
              width="40px"
              class="px-2 hover:cursor-pointer"
              alt=""
              @click="changeTypePassword()"
            />
            <img
              v-else
              src="~icon/ic-eyeclosed.svg"
              alt=""
              width="40px"
              class="px-2 hover:cursor-pointer"
              @click="changeTypePassword()"
            />
          </div>
          <p v-if="invalidLogin === true" class="text-red-600 text-sm mt-1">Invalid email or password</p>
          <div class="flex justify-between">
            <div></div>
            <a href="/forgot-password" class="text-right mt-2"
              >Lupa kata sandi?</a
            >
          </div>
        </div>
        <div class="flex flex-col justify-center my-8">
          <button
            :disabled="disabledButton"
            :class="[
              emailCheck === false && password.length > 0
                ? 'bg-Primary/50 text-white'
                : 'bg-[#888888]'
            ]"
            class="py-2  rounded hp:px-3"
            @click="login()"
          >
            Masuk
          </button>
          <p class="text-center mt-4">
            Belum punya akun?
            <a href="/register" class="font-bold text-[#273C75]"
              >Daftar disini</a
            >
          </p>
        </div>
      </div>
    </div>
    <RightSide />
  </div>
  <ErrorModal
    v-show="confirmEmail"
    :showError="confirmEmail"
    :message="messageError"
    :title="'Failed to login'"
    :isLink="true"
    @close="confirmEmail = false"
    @resend="resendEmail()"
  />
  </div>
</template>

<script>
import RightSide from '~/components/Organisms/RightSide/RightSide.vue'
import ErrorModal from '~/components/Modals/Error/ErrorModals.vue'
export default {
  name: 'LoginPage',
  components: {
    RightSide,
    ErrorModal
  },
  data() {
    return {
      email: '',
      password: '',
      typePassword: false,
      typePsd: 'password',
      emailCheck: false,
      debounce: null,
      passwordCheck: false,
      buttonTrue: null,
      confirmEmail: false,
      invalidLogin: false,
      messageError: ''
    }
  },

  computed: {
    disabledButton() {
      return this.emailCheck === false && this.password === ''
    }
  },

  watch: {
    email() {
      // eslint-disable-next-line no-undef
      clearTimeout(this.debounce)

      this.debounce = setTimeout(() => {
        this.emailFormatCheck()
        this.invalidLogin = false
      }, 500)
    },

    password() {
      clearTimeout(this.debounce)
      this.debounce = setTimeout(() => {
        this.invalidLogin = false
      }, 100)
    }
  },

  methods: {
    async login() {
      await this.$auth
        .loginWith('local', {
          data: {
            user: {
              email: this.email,
              password: this.password
            }
          }
        })
        .then((res) => {
          this.$router.push('/home')
        })
        .catch((err) => {
          if(err.response.status === 403){
            this.confirmEmail = true;
            this.invalidLogin = false;
            this.messageError = err.response.data.message
          } else {
            this.invalidLogin = true;
        }
        // console.log(err)
        })
    },

    async resendEmail(){
      await this.$store.dispatch('confirmEmail/resendConfirmationEmail', {
        email: this.email
      }).catch((err) => {
        console.log(err)
      })
    },

    changeTypePassword() {
      this.typePassword = !this.typePassword
      if (this.typePassword === true) {
        this.typePsd = 'text'
      } else {
        this.typePsd = 'password'
      }
    },

    emailFormatCheck() {
      // eslint-disable-next-line no-useless-escape
      const regex = /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,24})+$/

      if (regex.test(this.email)) {
        this.emailCheck = false
      } else {
        this.emailCheck = true
      }
    },

    passwordLengthCheck() {
      if (this.password.length > 0 && this.emailCheck === true) {
        this.passwordCheck = true
        this.emailCheck = true
      } else {
        this.passwordCheck = false
      }
    }
  }
}
</script>

<style></style>
