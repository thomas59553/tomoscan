<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <div
            v-if="total == 0"
            class="wethio-empty">
            <i class="fa fa-user-secret wethio-empty__icon"/>
            <p class="wethio-empty__description">No signer found</p>
        </div>

        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('signer', 'signers', total) }} found</p>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="signers"
            class="wethio-table--signers">
            <template
                slot="signer"
                slot-scope="props">
                <nuxt-link
                    :to="{name: 'address-slug', params: {slug: props.item.signer}}">{{ props.item.signer }}</nuxt-link>
            </template>

        </table-base>

    </section>
</template>
<script>
import mixin from '~/plugins/mixin'
import TableBase from '~/components/TableBase'

export default {
    components: {
        TableBase
    },
    mixins: [mixin],
    props: {
        block: {
            type: String,
            default: ''
        },
        page: {
            type: Object,
            default: () => {
                return {}
            }
        }
    },
    data: () => ({
        fields: {
            number: { label: 'Number' },
            signer: { label: 'Signer' }
        },
        loading: true,
        total: 0,
        signers: []
    }),
    async mounted () {
        this.getDataFromApi()
    },
    methods: {
        async getDataFromApi () {
            const self = this

            // Show loading.
            self.loading = true

            const { data } = await this.$axios.get('/api/blocks/signers/' + self.block)
            const signers = data.signers
            // let dataSigners = []
            let num = 1
            signers.forEach((item) => {
                self.signers.push({
                    signer: item,
                    number: num++
                })
            })
            self.total = data.signers.length
            self.page.blockSignerCount = self.total

            // Hide loading.
            self.loading = false

            return data
        }
    }
}
</script>
