<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <div
            v-if="total == 0"
            class="wethio-empty">
            <i class="fa fa-user-secret wethio-empty__icon"/>
            <p class="wethio-empty__description">No account found</p>
        </div>

        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('account', 'accounts', total) }}</p>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="wethio-table--accounts">

            <template
                slot="rank"
                slot-scope="props">{{ props.item.rank }}</template>

            <template
                slot="hash"
                slot-scope="props">
                <div>
                    <i
                        v-if="props.item.isContract"
                        class="tm tm-icon-contract mr-1 mr-lg-2"/>
                    <nuxt-link
                        :to="{name: 'address-slug', params: {slug: props.item.hash}}"
                        class="text-truncate">{{ props.item.hash }}</nuxt-link>
                </div>
            </template>

            <template
                slot="balance"
                slot-scope="props">
                <span class="d-lg-none">{{ formatUnit(toTomo(props.item.balance, 5)) }}</span>
                <span class="d-none d-lg-block">{{ formatUnit(toTomo(props.item.balance)) }}</span>
            </template>
        </table-base>

        <b-pagination
            v-if="total > 0 && total > perPage"
            v-model="currentPage"
            :total-rows="pages * perPage"
            :per-page="perPage"
            :number-of-pages="pages"
            :limit="7"
            align="center"
            class="wethio-pagination"
            @change="onChangePaginate"/>
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
    data: () => ({
        fields: {
            rank: { label: 'Rank' },
            hash: { label: 'Address' },
            balance: { label: 'Balance' }
        },
        loading: true,
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 20,
        pages: 1
    }),
    async mounted () {
        try {
            const self = this

            // Init breadcrumbs data.
            this.$store.commit('breadcrumb/setItems', { name: 'accounts', to: { name: 'accounts' } })

            await self.getDataFromApi()
        } catch (error) {
            console.log(error)
        }
    },
    methods: {
        async getDataFromApi () {
            const self = this

            // Show loading.
            self.loading = true

            const params = {
                page: self.currentPage || 1,
                limit: self.perPage
            }

            const query = this.serializeQuery(params)
            const { data } = await this.$axios.get('/api/accounts' + '?' + query)
            self.items = data.items
            self.total = data.total
            self.pages = data.pages

            // Hide loading.
            self.loading = false

            return data
        },

        onChangePaginate (page) {
            this.currentPage = page
            this.getDataFromApi()
        }
    },
    head: () => ({
        title: 'Accounts'
    })
}
</script>

<style scoped>
</style>
