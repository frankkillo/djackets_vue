<template>
    <tr>
        <td>{{ item.product.name }}</td>
        <td>£{{ item.product.price}}</td>
        <td>{{ item.quantity }}
            <a @click="decrementQuantity(item)"><strong> - </strong></a>   
            <a @click="incrementQuantity(item)"><strong> + </strong></a>
        </td>
        <td>{{ getItemTotal(item).toFixed(2) }}</td>
        <td><button class="delete" @click="removeFromCart(item)"></button></td>
    </tr>
</template>

<script>
export default {
    name: 'CartItem',
    props: {
        initializeItem: Object
    },
    data () {
        return {
            item: this.initializeItem
        }
    },
    methods: {
        getItemTotal (item) {
            return item.quantity * item.product.price
        },
        decrementQuantity (item) {
            if (item.quantity > 1) {
                item.quantity -= 1

                this.updateCart()
            }
        },
        incrementQuantity (item) {
            item.quantity += 1

            this.updateCart()
        },
        updateCart() {
            localStorage.setItem('cart', JSON.stringify(this.$store.state.cart))
        },
        removeFromCart (item) {
            this.$emit('removeFromCart', item)

            this.updateCart()
        }
    }
}
</script>