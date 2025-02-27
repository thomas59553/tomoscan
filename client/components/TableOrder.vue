<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('order', 'orders', total) }}</p>
        <form
            v-if="showFilter"
            class="form-inline mb-30 filter-box"
            method="get">
            <div class="form-group">
                <input
                    v-model="user"
                    name="user"
                    type="text"
                    class="form-control"
                    placeholder="User address">
            </div>
            <div class="form-group mx-sm-3">
                <input
                    v-model="baseToken"
                    name="baseToken"
                    type="text"
                    class="form-control"
                    placeholder="Base token">
            </div>
            <div class="form-group mx-sm-3">
                <input
                    v-model="quoteToken"
                    name="quoteToken"
                    type="text"
                    class="form-control"
                    placeholder="Quote token">
            </div>
            <div class="form-group mr-sm-3">
                <select
                    v-model="side"
                    name="side"
                    class="form-control mx-sm-1">
                    <option
                        value=""
                        selected
                        hidden
                        disabled>Select side</option>
                    <option value="">No filter</option>
                    <option value="BUY">BUY</option>
                    <option value="SELL">SELL</option>
                </select>
            </div>
            <div class="form-group mr-sm-3">
                <select
                    id="inputStatus"
                    v-model="status"
                    name="status"
                    class="form-control mx-sm-1">
                    <option
                        :selected="status === '' ? 'selected' : ''"
                        value=""
                        hidden
                        disabled>Select status</option>
                    <option value="">No filter</option>
                    <option
                        :selected="status === 'OPEN' ? 'selected' : ''"
                        value="OPEN">OPEN</option>
                    <option
                        :selected="status === 'FILLED' ? 'selected' : ''"
                        value="FILLED">FILLED</option>
                    <option
                        :selected="status === 'PARTIAL_FILLED' ? 'selected' : ''"
                        value="PARTIAL_FILLED">PARTIAL_FILLED</option>
                    <option
                        :selected="status === 'CANCELLED' ? 'selected' : ''"
                        value="CANCELLED">CANCELLED</option>
                    <option
                        :selected="status === 'REJECTED' ? 'selected' : ''"
                        value="REJECTED">REJECTED</option>
                </select>
            </div>
            <button
                type="submit"
                class="btn btn-primary mr-sm-3">Filter</button>
            <button
                type="button"
                class="btn btn-secondary"
                @click="reset">Reset</button>
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
            class="wethio-table--orders">
            <template
                slot="hash"
                slot-scope="props">
                <i
                    v-if="props.item.status === 'FILLED'"
                    class="fa fa-check text-success ml-15"
                    aria-hidden="true"/>
                <i
                    v-if="props.item.status === 'CANCELLED'"
                    class="fa fa-ban text-danger ml-15"
                    aria-hidden="true"/>
                <nuxt-link
                    :to="{name: 'orders-slug', params: {slug: props.item.hash.toLowerCase()}}">
                    {{ hiddenString(props.item.hash.toLowerCase(), 8) }}</nuxt-link>
            </template>
            <template
                slot="userAddress"
                slot-scope="props">
                <nuxt-link
                    :to="{name: 'address-slug', params: {slug: props.item.userAddress.toLowerCase()}}">
                    {{ hiddenString(props.item.userAddress.toLowerCase(), 8) }}</nuxt-link>
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
                slot="quantity"
                slot-scope="props">
                {{ formatNumber(props.item.quantity) + ' ' + props.item.baseSymbol }}
            </template>
            <template
                slot="price"
                slot-scope="props">
                <span v-if="props.item.type === 'MO'">MARKET</span>
                <span v-else>
                    {{ formatNumber(props.item.price) + ' ' + props.item.quoteSymbol }}
                </span>
            </template>
            <template
                slot="filledAmount"
                slot-scope="props">
                {{ formatNumber(props.item.filledAmount) + ' ' + props.item.baseSymbol }}
            </template>
            <template
                slot="type"
                slot-scope="props">
                {{ props.item.type === 'LO' ? 'Limit' : 'Market' }}
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
        showFilter: {
            type: Boolean,
            default: false
        },
        txHash: {
            type: String,
            default: ''
        },
        tradeHash: {
            type: String,
            default: ''
        }
    },
    data: () => ({
        fields: {
            hash: { label: 'Hash' },
            userAddress: { label: 'User' },
            pairName: { label: 'Pair Name' },
            side: { label: 'Side' },
            quantity: { label: 'Quantity' },
            price: { label: 'Price' },
            filledAmount: { label: 'Filled Amount' },
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
        side: '',
        status: '',
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
        if (this.$route.query.side) {
            this.side = this.$route.query.side
        }
        if (this.$route.query.status) {
            this.status = this.$route.query.status
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
            // on tx detail tab
            if (this.txHash !== '') {
                params.txHash = this.txHash
            } else if (this.tradeHash !== '') {
                params.tradeHash = this.tradeHash
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
                if (this.side !== '') {
                    params.side = this.side
                }
                if (this.status !== '') {
                    params.status = this.status
                }
            }

            const query = this.serializeQuery(params)
            const { data } = await this.$axios.get('/api/orders?' + query)
            self.total = data.total
            self.pages = data.pages

            if (data.items.length === 0) {
                self.loading = false
            }
            if (self.page) {
                self.page.txsCount = self.total
            }

            data.items.forEach(async (item, index, array) => {
                if (index === array.length - 1) {
                    self.items = self.formatData(array)

                    // Hide loading.
                    self.loading = false
                }
            })

            return data
        },
        async filter () {
            await this.getDataFromApi()
        },
        async reset () {
            this.user = ''
            this.baseToken = ''
            this.quoteToken = ''
            this.side = ''
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
