<template>
     <div class="page-my-account">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">My account</h1>
            </div>
            <div class="column is-12">
                <button @click="logOut()" class="button is-danger">Log out</button>
            </div>
            <div class="column is-12">
                <h2 class="subtitle">My orders</h2>
                <OrderSummary
                    v-for="order in orders"
                    v-bind:key="order.id"
                    v-bind:order="order" />
            </div>
        </div>
     </div>
</template>

<script>
import axios from 'axios'
import OrderSummary from '@/components/OrderSummary'
import {toast} from 'bulma-toast'

export default {
    name: 'MyAccount',
    data () {
        return {
            orders: []
        }
    },
    mounted () {
        document.title = 'My Account | Djackets'
        this.getOrderList()
    },
    methods: {
        logOut () {
            axios.defaults.headers.common['Authorization'] = ""
            localStorage.removeItem('token')
            localStorage.removeItem('username')
            localStorage.removeItem('user_id')
            this.$store.commit('removeToken')

            this.$router.push('/')
        },
        async getOrderList () {
            this.$store.commit('setIsLoading', true)

            await axios
                .get('/api/v1/orders/')
                .then(response => {
                    this.orders = response.data
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
            
            this.$store.commit('setIsLoading', false)
        }
    },
    components: {
        OrderSummary
    }
}
</script>