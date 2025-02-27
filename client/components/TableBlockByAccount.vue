<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>

        <div
            v-if="total == 0"
            class="wethio-empty">
            <i class="fa fa-cube wethio-empty__icon"/>
            <p class="wethio-empty__description">No item found</p>
        </div>

        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('block', 'blocks', total) }}</p>

        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="wethio-table--tx-by-account">
            <template
                slot="block"
                slot-scope="props">
                <nuxt-link :to="{name: 'blocks-slug', params: {slug: props.item.number}}">
                    {{ props.item.number }}</nuxt-link>
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
                slot="e_tx"
                slot-scope="props">{{ props.item.e_tx }}</template>

            <template
                slot="gasUsed"
                slot-scope="props">{{ formatNumber(props.item.gasUsed) }}</template>
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
            block: { label: 'Block' },
            timestamp: { label: 'Age' },
            e_tx: { label: 'txn' },
            gasUsed: { label: 'Gas Used' }
        },
        loading: true,
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 20,
        pages: 1
    }),
    async mounted () {
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

            const hash = self.address

            const query = this.serializeQuery(params)
            const { data } = await this.$axios.get('/api/accounts/' + hash + '/mined?' + query)
            self.items = data.items
            self.total = data.total
            self.pages = data.pages
            self.perPage = data.perPage

            if (self.page) {
                self.page.blocksCount = self.total
            }

            // Hide loading.
            self.loading = false

            return data
        },
        onChangePaginate (page) {
            this.currentPage = page
            this.getDataFromApi()
        }
    }
}
</script>
