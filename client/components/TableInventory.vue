<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>

        <div
            v-if="total == 0"
            class="wethio-empty">
            <i class="fa fa-cube wethio-empty__icon"/>
            <p class="wethio-empty__description">No token ID found</p>
        </div>

        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('token ID', 'Tokens ID', total) }}</p>
        <table-base
            v-if="total > 0"
            :fields="fields"
            :items="items"
            class="wethio-table--reward">
            <template
                slot="tokenId"
                slot-scope="props">
                {{ props.item.tokenId }}
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
        },
        parent: {
            type: String,
            default: ''
        }
    },
    data: () => ({
        fields: {
            tokenId: { label: 'Token ID' }
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

            const query = this.serializeQuery(params)
            const { data } = await this.$axios.get(`/api/tokens/${self.token}/nftHolder/${self.holder}?${query}`)
            self.items = data.items
            self.total = data.total
            self.pages = data.pages
            self.perPage = data.perPage

            // Hide loading.
            self.loading = false

            // return data
        },
        onChangePaginate (page) {
            this.currentPage = page
            this.getDataFromApi()
        }
    }
}
</script>
