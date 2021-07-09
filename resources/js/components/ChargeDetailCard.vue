<template>
    <loading-card :loading="initialLoading" class="mb-6 py-3 px-6">
        <div class="flex border-b border-40 -mx-6 px-6">
            <div class="w-1/4 py-4"><h4 class="font-normal text-80">Payment</h4></div>
            <div class="w-3/4 py-4 break-words"><p class="text-90">
                <a :href="'https://dashboard.stripe.com/'+(charge.livemode ? '' : 'test/')+'payments/'+ charge.payment_intent" target="_blank">{{ charge.payment_intent }}</a>
            </p></div>
        </div>
        <div class="flex border-b border-40 -mx-6 px-6">
            <div class="w-1/4 py-4"><h4 class="font-normal text-80">Charge</h4></div>
            <div class="w-3/4 py-4 break-words"><p class="text-90">
                <a :href="'https://dashboard.stripe.com/'+(charge.livemode ? '' : 'test/')+'payments/'+ charge.id+'/review'" target="_blank">{{ charge.id }}</a>
            </p></div>
        </div>
        <div class="flex border-b border-40 -mx-6 px-6">
            <div class="w-1/4 py-4"><h4 class="font-normal text-80">Customer</h4></div>
            <div class="w-3/4 py-4 break-words"><p class="text-90">
                <a :href="'https://dashboard.stripe.com/'+(charge.livemode ? '' : 'test/') +'customers/'+ charge.customer" target="_blank">{{ charge.customer }}</a>
            </p></div>
        </div>
        <div class="flex border-b border-40 -mx-6 px-6">
            <div class="w-1/4 py-4"><h4 class="font-normal text-80">Receipt</h4></div>
            <div class="w-3/4 py-4 break-words"><p class="text-90">
                <a :href="charge.receipt_url" target="_blank">Receipt</a>
            </p></div>
        </div>
        <detail-text-field :field="{name: 'Amount', value: amount }"></detail-text-field>
        <detail-text-field :field="{name: 'Fee', value: fee }"></detail-text-field>
        <detail-text-field :field="{name: 'Net', value: net }"></detail-text-field>
        <detail-text-field :field="{name: 'Status', value: charge.status }"></detail-text-field>
        <detail-text-field :field="{name: 'Created', value: date }"></detail-text-field>
        <detail-text-field :field="{name: 'Metadata', value: charge.metadata }"></detail-text-field>
        <detail-boolean-field :field="{name: 'Livemode', value: charge.livemode}"></detail-boolean-field>
        <detail-boolean-field :field="{name: 'Captured', value: charge.captured}"></detail-boolean-field>
        <detail-boolean-field :field="{name: 'Paid', value: charge.paid}"></detail-boolean-field>
        <detail-boolean-field :field="{name: 'Refunded', value: charge.refunded}"></detail-boolean-field>
        <detail-text-field :field="{name: 'Dispute', value: charge.dispute }"></detail-text-field>
        <detail-text-field :field="{name: 'Fraud Details', value: charge.fraud_details }"></detail-text-field>
        <detail-text-field :field="{name: 'Transfer Group', value: charge.transfer_group }"></detail-text-field>
    </loading-card>
</template>

<script>
export default {
    props: ['chargeId'],

    data() {
        return {
            initialLoading: true,
            charge: {
                amount: 0,
                currency: ''
            },
        }
    },

    computed: {
        amount() {
            return this.formatMoney(this.charge.amount, this.charge.currency)
        },

        fee() {
            if (! this.charge.balance_transaction) {
                return 0
            }

            return this.formatMoney(this.charge.balance_transaction.fee, this.charge.balance_transaction.currency)
        },

        net() {
            if (! this.charge.balance_transaction) {
                return 0
            }

            return this.formatMoney((this.charge.amount - this.charge.balance_transaction.fee), this.charge.currency)
        },

        date() {
            return moment.unix(this.charge.created).format('YYYY/MM/DD h:mm:ss a')
        },
    },

    methods: {
        moment: moment,

        getCharge() {
            Nova.request().get('/nova-vendor/nova-stripe/stripe/charges/' + this.chargeId)
                .then((response) => {
                    this.charge = response.data.charge
                    this.initialLoading = false
                })
        },

        formatMoney(amount, currency) {
            return `${ (amount / 100).toFixed(2) } ${ currency.toUpperCase() }`
        },
    },

    created() {
        this.getCharge()
    }
}
</script>
