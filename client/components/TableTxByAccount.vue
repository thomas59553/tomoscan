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
            class="wethio-total-items">{{ _nFormatNumber('transaction', 'transactions', total) }}</p>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="wethio-table--transactions">
            <template
                slot="hash"
                slot-scope="props">
                <nuxt-link
                    :to="{name: 'txs-slug', params: {slug: props.item.hash}}"
                    class="text-truncate">
                    <i
                        v-if="type !== 'pending' && !props.item.status"
                        class="fa fa-exclamation mr-1 text-danger tx-failed"/>
                    {{ props.item.hash }}</nuxt-link>
            </template>

            <template
                slot="block"
                slot-scope="props">
                <nuxt-link
                    v-if="props.item.blockNumber"
                    :to="{name: 'blocks-slug', params: {slug: props.item.blockNumber}}">
                    {{ props.item.blockNumber }}</nuxt-link>
                <span
                    v-else
                    class="text-muted">Pending...</span>
            </template>

            <template
                slot="timestamp"
                slot-scope="props">
                <span
                    v-b-tooltip.hover
                    :title="$moment(props.item.timestamp).format('lll')">
                    {{ $moment(props.item.timestamp).fromNow() }}</span>
            </template>

            <template
                slot="gas"
                slot-scope="props">{{ formatNumber(props.item.gas) }}</template>

            <template
                slot="from"
                slot-scope="props">
                <i
                    v-if="props.item.from_model && props.item.from_model.isContract"
                    class="tm tm-icon-contract mr-1 mr-lg-2"/>
                <span
                    v-if="address == props.item.from"
                    class="text-truncate">{{ (props.item.from_model && props.item.from_model.accountName) ?
                        props.item.from_model.accountName : props.item.from }}</span>
                <nuxt-link
                    v-else
                    :to="{name: 'address-slug', params: {slug: props.item.from}}"
                    class="text-truncate">{{ (props.item.from_model && props.item.from_model.accountName) ?
                        props.item.from_model.accountName : props.item.from }}</nuxt-link>
            </template>

            <template
                slot="arrow"
                slot-scope="props">
                <i
                    :class="props.item.from == address ? 'text-danger' : 'text-success'"
                    class="tm-arrow-right"/>
            </template>

            <template
                slot="to"
                slot-scope="props">
                <div v-if="props.item.to">
                    <i
                        v-if="props.item.to_model && props.item.to_model.isContract"
                        class="tm tm-icon-contract mr-1 mr-lg-2"/>
                    <span
                        v-if="address == props.item.to"
                        class="text-truncate">{{ (props.item.to_model && props.item.to_model.accountName) ?
                            props.item.to_model.accountName : props.item.to }}</span>
                    <nuxt-link
                        v-else
                        :to="{name: 'address-slug', params:{slug: props.item.to}}"
                        class="text-truncate">{{ (props.item.to_model && props.item.to_model.accountName) ?
                            props.item.to_model.accountName : props.item.to }}</nuxt-link>
                </div>
                <div
                    v-else
                    class="contract-creation">
                    <span>Contract Creation</span>
                </div>
            </template>

            <template
                slot="value"
                slot-scope="props">{{ formatUnit(toTomo(props.item.value)) }}</template>

            <template
                slot="txFee"
                slot-scope="props">
                <span
                    v-b-tooltip.hover
                    :title="formatUnit(toTomo(props.item.gasPrice * props.item.gasUsed, 18))">
                    {{ formatUnit(toTomo(props.item.gasPrice * props.item.gasUsed, 8)) }}</span></template>
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
        address: {
            type: String,
            default: ''
        },
        type: {
            type: String,
            default: ''
        },
        page: {
            type: Object,
            default: () => {
                return {}
            }
        },
        parent: {
            type: String,
            default: ''
        }
    },
    data: () => ({
        fields: {
            hash: { label: 'TxHash' },
            block: { label: 'Block' },
            timestamp: { label: 'Age' },
            from: { label: 'From' },
            arrow: { label: '' },
            to: { label: 'To' },
            value: { label: 'Value', cssClass: 'pr-lg-4' },
            txFee: { label: 'TxFee' }
        },
        loading: true,
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 20,
        pages: 1,
        blockNumber: null
    }),
    async mounted () {
        this.getDataFromApi()
    },
    methods: {
        async getDataFromApi () {
            // Show loading.
            this.loading = true
            const params = {
                page: this.currentPage,
                limit: this.perPage
            }

            params.tx_type = this.type
            const query = this.serializeQuery(params)
            const { data } = await this.$axios.get('/api/txs/listByAccount/' + this.address + '?' + query)
            this.total = data.total || this.tx_total || (data.items || []).length
            this.pages = data.pages || (this.total % this.perPage)

            if (data.items.length === 0) {
                this.loading = false
            }
            if (this.page) {
                if (this.type === 'in') {
                    this.page.totalInTx = this.total
                } else if (this.type === 'out') {
                    this.page.totalOutTx = this.total
                } else {
                    this.page.totalInTx = this.total
                    this.page.totalOutTx = 0
                }
            }

            data.items.forEach(async (item, index, array) => {
                if (index === array.length - 1) {
                    this.items = array

                    // Format data.
                    if (this.blockNumber) {
                        this.items = this.formatData(this.items, this.block_timestamp)
                    } else {
                        this.items = this.formatData(this.items, null)
                    }

                    // Hide loading.
                    this.loading = false
                }
            })

            return data
        },
        formatData (items = [], blockTimestamp) {
            const _items = []
            items.forEach((item) => {
                const _item = item

                // Format for timestamp.
                if (blockTimestamp) {
                    _item.timestamp = blockTimestamp
                } else if (!item.block) {
                    if (item.timestamp) {
                        _item.timestamp = item.timestamp
                    } else {
                        _item.timestamp = item.createdAt
                    }
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
        },
        isPending () {
            return this.type === 'pending'
        }
    },
    head () {
        if (this.block) {
            return {
                title: 'Block ' + this.$route.params.slug + ' Info'
            }
        } else {
            return {
                title: this.isPending() ? 'Transactions Pending' : 'Transactions'
            }
        }
    }
}
</script>
