<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>

        <div
            v-if="total == 0"
            class="wethio-empty">
            <i class="fa fa-exchange wethio-empty__icon"/>
            <p class="wethio-empty__description">No transaction found</p>
        </div>

        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('transaction', 'transactions', total) }} </p>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="wethio-table--token-tx">
            <template
                slot="transactionHash"
                slot-scope="props">
                <nuxt-link
                    :to="{name: 'txs-slug', params: {slug: props.item.transactionHash}}"
                    class="text-truncate">{{ props.item.transactionHash }}</nuxt-link>
            </template>

            <template
                slot="block"
                slot-scope="props">
                <nuxt-link
                    v-if="props.item.block"
                    :to="{name: 'blocks-slug', params: {slug: props.item.blockNumber}}">
                    {{ props.item.blockNumber }}</nuxt-link>
                <span
                    v-else
                    class="text-muted">Pending...</span>
            </template>

            <template
                slot="timestamp"
                slot-scope="props">
                <div v-if="props.item.timestamp">
                    <span
                        v-b-tooltip.hover
                        :title="$moment(props.item.timestamp).format('lll')">
                        {{ $moment(props.item.timestamp).fromNow() }}</span>
                </div>
            </template>

            <template
                slot="from"
                slot-scope="props">
                <i
                    v-if="props.item.from_model && props.item.from_model.isContract"
                    class="tm tm-icon-contract mr-1 mr-lg-2"/>
                <span
                    v-if="holder === props.item.from"
                    class="text-truncate">{{ props.item.from }}</span>
                <nuxt-link
                    v-else
                    :to="{name: 'tokens-slug-trc721-holder',
                          params: {slug: props.item.address, holder: props.item.from}}"
                    class="text-truncate">{{ props.item.from }}</nuxt-link>
            </template>

            <template
                slot="arrow"
                slot-scope="props">
                <i
                    :class="props.item.from === holder ? 'text-danger' : 'text-success'"
                    class="tm-arrow-right"/>
            </template>

            <template
                slot="to"
                slot-scope="props">
                <div>
                    <i
                        v-if="props.item.to_model && props.item.to_model.isContract"
                        class="tm tm-icon-contract mr-1 mr-lg-2"/>
                    <span
                        v-if="holder === props.item.to"
                        class="text-truncate">{{ props.item.to }}</span>
                    <nuxt-link
                        v-else
                        :to="{name: 'tokens-slug-trc721-holder',
                              params: {slug: props.item.address, holder: props.item.to}}"
                        class="text-truncate">{{ props.item.to }}</nuxt-link>
                </div>
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
    props: {
        token: {
            type: String,
            default: ''
        },
        holder: {
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
            transactionHash: { label: 'TxHash' },
            timestamp: { label: 'Age' },
            from: { label: 'From' },
            arrow: { class: 'text-center' },
            to: { label: 'To' },
            tokenId: { label: 'Token ID' }
        },
        loading: true,
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 20,
        pages: 1,
        address: null
    }),
    async mounted () {
        const self = this
        // Init from router.
        const query = self.$route.query

        if (query.address) {
            self.address = query.address
        }

        this.getDataFromApi()
    },
    methods: {
        async getDataFromApi () {
            const self = this

            // Show loading.
            self.loading = true

            const params = {
                page: self.currentPage,
                limit: self.perPage
            }

            if (self.token) {
                params.token = self.token
            }
            if (self.holder) {
                params.holder = self.holder
            }

            const query = this.serializeQuery(params)
            const { data } = await this.$axios.get('/api/token-txs/trc721' + '?' + query)
            self.items = data.items
            self.total = data.total
            self.pages = data.pages

            if (self.page) {
                self.page.tokenTxsCount = self.total
            }

            // Hide loading.
            self.loading = false

            // Format data.
            self.items = self.formatData(self.items)

            return data
        },
        formatData (items = []) {
            const _items = []
            items.forEach((item) => {
                const _item = item

                // Format for timestamp.
                if (!item.block) {
                    _item.timestamp = item.createdAt
                } else {
                    _item.timestamp = item.block.timestamp
                }

                _items.push(_item)
            })

            return _items
        },
        onChangePaginate (page) {
            this.currentPage = page
            this.getDataFromApi()
        }
    }
}
</script>
