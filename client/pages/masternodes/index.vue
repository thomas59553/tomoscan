<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <div
            v-if="total == 0"
            class="wethio-empty">
            <i class="fa fa-user-secret wethio-empty__icon"/>
            <p class="wethio-empty__description">No masternode found</p>
        </div>

        <p
            v-if="total > 0"
            class="wethio-total-items">{{ _nFormatNumber('MasterNode', 'MasterNodes', total) }}</p>

        <b-table
            v-if="total > 0"
            :items="items"
            :fields="fields"
            :per-page="perPage"
            :current-page="currentPage"
            :show-empty="true"
            empty-text="There are no masternodes to show"
            class="wethio-table wethio-table--masternode"
            stacked="md">

            <template
                slot="address"
                slot-scope="props">
                <div>
                    <nuxt-link
                        :to="{name: 'address-slug', params: {slug: props.item.address}}"
                        class="text-truncate">{{ props.item.address }}</nuxt-link>
                </div>
            </template>

            <template
                slot="latestSignedBlock"
                slot-scope="props">
                <nuxt-link :to="{name: 'blocks-slug', params: {slug: props.item.latestSignedBlock}}">
                    {{ props.item.latestSignedBlock }}</nuxt-link>
            </template>

            <template
                slot="capacity"
                slot-scope="props">
                {{ formatNumber(props.item.capacity) }}
            </template>

            <template
                slot="owner"
                slot-scope="props">
                <div>
                    <nuxt-link
                        :to="{name: 'address-slug', params: {slug: props.item.owner}}"
                        class="text-truncate">{{ props.item.owner }}</nuxt-link>
                </div>
            </template>
        </b-table>

        <b-pagination-nav
            v-if="total > 0 && total > perPage"
            :per-page="perPage"
            :number-of-pages="pages"
            :link-gen="linkGen"
            :limit="7"
            :value="currentPage"
            align="center"
            class="wethio-pagination"/>
    </section>
</template>

<script>
import mixin from '~/plugins/mixin'

export default {
    components: {
    },
    mixins: [mixin],
    data: () => ({
        fields: [
            {
                key: 'number',
                label: 'Num'
            },
            {
                key: 'address',
                label: 'Address'
            },
            {
                key: 'name',
                label: 'Name'
            },
            {
                key: 'capacity',
                label: 'Capacity'
            },
            {
                key: 'owner',
                label: 'Owner'
            },
            {
                key: 'latestSignedBlock',
                label: 'Latest signed block'
            }
        ],
        loading: true,
        total: 0,
        items: [],
        currentPage: 1,
        perPage: 50,
        pages: 1
    }),
    watch: {
        $route (to, from) {
            const page = this.$route.query.page
            this.onChangePaginate(page)
        }
    },
    async mounted () {
        try {
            const self = this

            // Init breadcrumbs data.
            this.$store.commit('breadcrumb/setItems', { name: 'masternodes', to: { name: 'masternodes' } })

            const query = this.$route.query
            if (query.page && !isNaN(query.page)) {
                self.currentPage = parseInt(query.page)
            }

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

            const { data } = await this.$axios.get('/api/masternodes')
            let num = 1
            const items = (data || {}).items || []

            for (let i = 0; i < items.length; i++) {
                if (items[i].isMasternode) {
                    self.items.push({
                        number: num,
                        address: items[i].candidate,
                        capacity: items[i].capacityNumber,
                        name: items[i].name || 'Anonymous',
                        owner: items[i].owner,
                        latestSignedBlock: items[i].latestSignedBlock
                    })
                    num++
                }
            }
            self.total = data.activeCandidates
            self.pages = Math.ceil(self.total / self.perPage)

            // Hide loading.
            self.loading = false

            return data
        },
        linkGen (pageNum) {
            return {
                query: {
                    page: pageNum
                },
                hash: this.parent
            }
        },
        onChangePaginate (page) {
            const self = this
            self.currentPage = page
        }
    },
    head: () => ({
        title: 'Masternodes'
    })
}
</script>
