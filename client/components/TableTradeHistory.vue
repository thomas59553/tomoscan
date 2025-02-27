<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>

        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('trade history', 'trades history', total) }}</p>
        <form
            v-if="showFilter"
            class="form-inline mb-30 filter-box"
            method="get">
            <div class="form-group mr-2 mb-2">
                <input
                    v-model="user"
                    name="user"
                    type="text"
                    class="form-control"
                    placeholder="User address">
            </div>
            <div class="form-group mr-2 mb-2">
                <input
                    v-model="baseToken"
                    name="baseToken"
                    type="text"
                    class="form-control"
                    placeholder="Base Token">
            </div>
            <div class="form-group mr-2 mb-2">
                <input
                    v-model="quoteToken"
                    name="quoteToken"
                    type="text"
                    class="form-control"
                    placeholder="Quote Token">
            </div>
            <div class="form-group mr-2 mb-2">
                <input
                    v-model="relayerAddress"
                    name="relayerAddress"
                    type="text"
                    class="form-control"
                    placeholder="Relayer Address">
            </div>
            <div class="form-group mr-2 mb-2">
                <button
                    type="submit"
                    class="btn btn-primary">Filter</button>
            </div>
            <div class="form-group mr-2 mb-2">
                <button
                    type="button"
                    class="btn btn-secondary"
                    @click="reset">Reset</button>
            </div>
        </form>

        <div
            v-if="total === 0"
            class="wethio-empty">
            <i class="fa fa-exchange wethio-empty__icon"/>
            <p class="wethio-empty__description">No order found</p>
        </div>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="wethio-table--trades">
            <template
                slot="hash"
                slot-scope="props">
                <nuxt-link
                    :to="{name: 'trades-slug', params: {slug: props.item.hash.toLowerCase()}}">
                    <i
                        v-if="props.item.status === 'ERROR'"
                        class="fa fa-exclamation mr-1 text-danger tx-failed"/>
                    {{ hiddenString(props.item.hash.toLowerCase(), 8) }}</nuxt-link>
            </template>
            <template
                slot="taker"
                slot-scope="props">
                <nuxt-link
                    :to="{name: 'address-slug', params: {slug: props.item.taker.toLowerCase()}}">
                    {{ hiddenString(props.item.taker.toLowerCase(), 8) }}</nuxt-link>
            </template>
            <template
                slot="maker"
                slot-scope="props">
                <nuxt-link
                    :to="{name: 'address-slug', params: {slug: props.item.maker.toLowerCase()}}">
                    {{ hiddenString(props.item.maker.toLowerCase(), 8) }}</nuxt-link>
            </template>
            <template
                slot="pairName"
                slot-scope="props">
                <span>
                    <nuxt-link
                        v-if="props.item.baseToken !== wethioNativeToken"
                        :to="{name: 'tokens-slug', params: {slug: props.item.baseToken}}">
                        {{ props.item.baseSymbol }}</nuxt-link>
                    <span v-else>{{ props.item.baseSymbol }}</span>/<nuxt-link
                        v-if="props.item.quoteToken !== wethioNativeToken"
                        :to="{name: 'tokens-slug', params: {slug: props.item.quoteToken}}">
                        {{ props.item.quoteSymbol }}</nuxt-link>
                    <span v-else>{{ props.item.quoteSymbol }}</span>
                </span>
            </template>
            <template
                slot="pricepoint"
                slot-scope="props">
                {{ props.item.pricepoint === null ? 0 : formatNumber(props.item.pricepoint) }}
                {{ props.item.quoteSymbol }}
            </template>
            <template
                slot="amount"
                slot-scope="props">
                {{ formatNumber(props.item.amount) + ' ' + props.item.baseSymbol }}
            </template>
            <template
                slot="side"
                slot-scope="props">
                {{ props.item.takerOrderSide }}
            </template>
            <template
                slot="createdAt"
                slot-scope="props">
                <span
                    v-b-tooltip.hover
                    :title="$moment(props.item.createdAt).format('lll')">
                    {{ $moment(props.item.createdAt).fromNow() }}</span>
            </template>
        </table-base>

        <b-pagination
            v-if="total > 0 && total > perPage"
            v-model="currentPage"
            :total-rows="pages * perPage"
            :per-page="perPage"
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
        page: {
            type: Object,
            default: () => {
                return {}
            }
        },
        showFilter: {
            type: Boolean,
            default: false
        },
        userAddress: {
            type: String,
            default: ''
        },
        txHash: {
            type: String,
            default: ''
        },
        orderHash: {
            type: String,
            default: ''
        }
    },
    data: () => ({
        fields: {
            hash: { label: 'Trade hash' },
            taker: { label: 'Taker' },
            maker: { label: 'Maker' },
            pairName: { label: 'Pair Name' },
            pricepoint: { label: 'Price' },
            amount: { label: 'Amount' },
            side: { label: 'Taker Side' },
            createdAt: { label: 'Age' }
        },
        loading: true,
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 20,
        pages: 1,
        blockNumber: null,
        user: '',
        baseToken: '',
        quoteToken: '',
        relayerAddress: '',
        wethioNativeToken: process.env.WETHIO_NATIVE_TOKEN
    }),
    async created () {
        if (this.$route.query.user) {
            this.user = this.$route.query.user
        }
        if (this.$route.query.baseToken) {
            this.baseToken = this.$route.query.baseToken
        }
        if (this.$route.query.quoteToken) {
            this.quoteToken = this.$route.query.quoteToken
        }
        if (this.$route.query.relayerAddress) {
            this.relayerAddress = this.$route.query.relayerAddress
        }
        await this.getDataFromApi()
    },
    methods: {
        async getDataFromApi () {
            const self = this

            self.loading = true
            const params = {
                page: self.currentPage,
                limit: self.perPage
            }
            // show on address detail tab
            if (this.userAddress !== '') {
                params.user = this.userAddress

            // show on tx detail tab
            } else if (this.txHash !== '') {
                params.txHash = this.txHash
            } else if (this.orderHash !== '') {
                params.orderHash = this.orderHash
            } else {
                if (this.user !== '') {
                    params.user = this.user.trim()
                }
                if (this.baseToken !== '') {
                    params.baseToken = this.baseToken.trim()
                }
                if (this.quoteToken !== '') {
                    params.quoteToken = this.quoteToken.trim()
                }
                if (this.relayerAddress !== '') {
                    params.relayerAddress = this.relayerAddress
                }
            }
            const query = this.serializeQuery(params)
            const { data } = await this.$axios.get('/api/trades?' + query)
            self.total = data.total
            self.pages = data.pages

            if (data.items.length === 0) {
                self.loading = false
            }
            if (self.page) {
                self.page.txsCount = self.total
            }
            let totalAmount = 0
            let totalCost = 0
            data.items.forEach((item, index, array) => {
                totalAmount += item.amount
                totalCost += item.amount * item.pricepoint
                if (index === array.length - 1) {
                    self.items = self.formatData(array)
                    // Hide loading.
                    self.loading = false
                }
            })
            console.log('total amount', totalAmount, 'total cost', totalCost)
            const avgPrice = totalCost / totalAmount
            if (this.orderHash !== '') {
                this.page.avgPrice = avgPrice
            }
            return data
        },
        async filter () {
            await this.getDataFromApi()
        },
        async reset () {
            this.user = ''
            this.baseToken = ''
            this.quoteToken = ''
            this.type = ''
            await this.getDataFromApi()
        },
        formatData (items = []) {
            const _items = []
            items.forEach((item) => {
                const _item = item

                if (item.timestamp) {
                    _item.timestamp = item.timestamp
                } else {
                    _item.timestamp = item.createdAt
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
