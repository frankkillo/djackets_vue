<template>
    <div class="page-check-out">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Check out</h1>
            </div>
            <div class="column is-12 box">
                <table class="table is-narrow is-fullwidth" v-if="cartTotalLength">
                    <thead>
                        <tr>
                            <th>Product</th>
                            <th>Price</th>
                            <th>Quantity</th>
                            <th>Total</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr
                            v-for="item in cart.items"
                            v-bind:key="item.product.id"
                        >
                            <td>{{ item.product.name }}</td>
                            <td>£{{ item.product.price}}</td>
                            <td>{{ item.quantity }}</td>
                            <td>£{{ getItemTotal(item).toFixed(2) }}</td>
                        </tr>
                    </tbody>
                    <tfoot>
                        <tr>
                            <td colspan="2">Total</td>
                            <td>{{ cartTotalLength }}</td>
                            <td>£{{ getTotalPrice.toFixed(2) }}</td>
                        </tr>
                    </tfoot>
                </table>
            </div>
            <div class="column is-12 box">
                <h2 class="subtitle">Shipping details</h2>
                <p class="has-text-grey mb-4">* All fields are required</p>
                <div class="columns is-multiline">
                    <div class="column is-6">
                        <div class="field">
                            <label>First name*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="first_name">
                            </div>
                        </div>
                        <div class="field">
                            <label>Last name*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="last_name">
                            </div>
                        </div>
                        <div class="field">
                            <label>Email*</label>
                            <div class="control">
                                <input type="email" class="input" v-model="email">
                            </div>
                        </div>
                        <div class="field">
                            <label>Phone*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="phone">
                            </div>
                        </div>
                    </div>
                    <div class="column is-6">
                        <div class="field">
                            <label>Address*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="address">
                            </div>
                        </div>
                        <div class="field">
                            <label>Zipcode*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="zipcode">
                            </div>
                        </div>
                        <div class="field">
                            <label>Place*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="place">
                            </div>
                        </div>
                    </div>
                </div>
                <div class="notification is-danger mt-4" v-if="errors.length">
                    <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
                </div>
                <hr>
                <div id="card-element" class="mb-5"></div>
                <table v-if="cartTotalLength">
                    <hr>
                    <button class="button is-dark" @click="submitForm">Pay with stripe</button>
                </table>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

export default {
    name: 'CheckOut',
    data () {
        return {
            cart: {
                items: []
            },
            stripe: {},
            card: {},
            first_name: '',
            last_name: '',
            email: '',
            phone: '',
            address: '',
            zipcode: '',
            place: '',
            errors: []
        }
    },
    mounted () {
        this.cart = this.$store.state.cart
        document.title = 'Check out | Djackets'

        if (this.cartTotalLength > 0) {
            this.stripe = Stripe('pk_test_51L1ZcNIjt4INJgaRYgcRYPpa3ZTe4rXzRRvBSaYymOuYWKbWAItZ9jeT6CrUGFEy59nYoAFx07MGekCo5SkGvOnD005MAet4IQ')
            const elements = this.stripe.elements()
            this.card = elements.create('card', {hidePostalCode: true})

            this.card.mount('#card-element')
        }
    },
    methods: {
        getItemTotal(item) {
            return item.quantity * item.product.price
        },
        submitForm () {
            this.errors = []

            if (this.first_name === '') {
                this.errors.push('The first name is missing')
            }
            if (this.last_name === '') {
                this.errors.push('The last name is missing')
            }
            if (this.email === '') {
                this.errors.push('The email is missing')
            }
            if (this.phone === '') {
                this.errors.push('The phone is missing')
            }
            if (this.address === '') {
                this.errors.push('The address is missing')
            }
            if (this.zipcode === '') {
                this.errors.push('The zipcode is missing')
            }
            if (this.place === '') {
                this.errors.push('The place is missing')
            }
            if (!this.errors.length) {
                this.$store.commit('setIsLoading', true)

                this.stripe.createToken(this.card).then(result => {
                    if (result.error) {
                        this.$store.commit('setIsLoading', false)
                        
                        this.errors.push(result.error.message)

                        console.log(result.error.message)
                    } else {
                        this.stripeTokenHandler(result.token)
                    }
                })                
            }
        },
        async stripeTokenHandler (token) {
            const items = []

            for (let i = 0; i < this.cart.items.length; i++) {
                const item = this.cart.items[i]
                const obj = {
                    product: item.product.id,
                    quantity: item.quantity,
                    price: item.quantity * item.product.price
                }

                items.push(obj)
            }

            const data = {
                first_name: this.first_name,
                last_name: this.last_name,
                email: this.email,
                phone: this.phone,
                address: this.address,
                zipcode: this.zipcode,
                place: this.place,
                items: items,
                stripe_token: token.id
            }

            await axios
                .post('api/v1/checkout/', data)
                .then(response => {
                    this.$store.commit('clearCart')
                    this.$router.push('/cart/success')
                })
                .catch(error => {
                    this.errors.push('Something went wrong. Please try again')
                    console.log(error)
                })

            this.$store.commit('setIsLoading', false)
        }
    },
    computed: {
        cartTotalLength() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.quantity
            }, 0)
        },
        getTotalPrice() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.product.price * curVal.quantity
            }, 0)
        }
    }
}
</script>