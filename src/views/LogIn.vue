<template>
    <div class="page-log-in">
        <div class="columns">
            <div class="column is-4 is-offset-4">
                <h1 class="title">Log in</h1>
                <form @submit.prevent="submitForm">
                    <div class="field">
                        <label>Username</label>
                        <div class="control">
                            <input type="text" class="input" v-model="username">
                        </div>
                    </div>
                    <div class="field">
                        <label>Password</label>
                        <div class="control">
                            <input type="password" class="input" v-model="password">
                        </div>
                    </div>
                    <div class="notification is-danger" v-if="errors.length">
                        <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
                    </div>
                    <div class="field">
                        <div class="control">
                            <button class="button is-dark">Log in</button>
                        </div>
                    </div>
                    <hr>
                    Or <router-link to="/sign-up">click here</router-link> to sign up!
                </form>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import {toast} from 'bulma-toast'
export default {
    name: 'LogIn',
    data() {
        return {
            username: '',
            password: '',
            errors: []
        }
    },
    mounted () {
        document.title = 'Log in | Djackets'
    },
    methods: {
        async submitForm () {
            this.errors = []

            if (this.username === '') {
                this.errors.push('The username is missing')
            }
            if (this.password === '') {
                this.errors.push('The password is missing')
            }
            if (!this.errors.length) {
                axios.defaults.headers.common["Authorization"] = ""
                localStorage.removeItem("token")
                localStorage.removeItem("username")
                localStorage.removeItem("user_id")

                const formData = {
                    username: this.username,
                    password: this.password
                }
                await axios
                    .post('/api/v1/token/login', formData)
                    .then(response => {
                        const token = response.data.auth_token
                        this.$store.commit('setToken', token)
                        
                        axios.defaults.headers.common["Authorization"] = "Token " + token
                        localStorage.setItem("token", token)

                        const toPath = this.$route.query.to || '/cart'
                        this.$router.push(toPath)
                    })
                    .catch(error => {
                        if (error.response) {
                            for (const property in error.response.data) {
                                this.errors.push(`${property}: ${error.response.data[property]}`)
                            }
                        } else if (error.message) {
                            this.errors.push('Something went wrong. Please try again.')
                            console.log(JSON.stringify(error))
                        }
                    })
                axios
                    .get('api/v1/users/me')
                    .then(response => {
                        this.$store.commit('setUser',
                            {
                                username: response.data.username,
                                id: response.data.id
                            })

                        localStorage.setItem('username', response.data.username)
                        localStorage.setItem('user_id', response.data.id)
                    })
                    .catch(error => {
                        console.log(JSON.stringify(error))
                        toast({
                            message: 'Something went wrong. Please try again.',
                            type: 'is-danger',
                            dismissible: true,
                            pauseOnHover: true,
                            duration: 2000,
                            position: 'bottom-right',
                        })
                    })
            }
        }
    }

}
</script>