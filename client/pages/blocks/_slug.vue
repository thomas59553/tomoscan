<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <div class="card wethio-card wethio-card--block">
            <div class="wethio-card__header">
                <h3
                    v-if="block"
                    class="wethio-card__headline">Block
                    <span class="d-none d-lg-inline-block headline__block-number">#{{ block.number }}</span>
                </h3>
                <div
                    v-if="block"
                    class="block-breadcrumb">
                    <div class="block-breadcrumb__prev">
                        <i class="tm tm-chevrons-left"/>
                        <nuxt-link :to="{name: 'blocks-slug', params: {slug: block.number - 1}}">Prev</nuxt-link>
                    </div>
                    <span class="block-breadcrumb__divider">|</span>
                    <div class="block-breadcrumb__next">
                        <nuxt-link :to="{name: 'blocks-slug', params: {slug: block.number + 1}}">Next</nuxt-link>
                        <i class="tm tm-chevrons-right"/>
                    </div>
                </div>
            </div>
            <div class="wethio-card__body">
                <table
                    v-if="block"
                    class="wethio-card__table">
                    <tbody>
                        <tr>
                            <td>Height</td>
                            <td>{{ block.number }}</td>
                        </tr>
                        <tr>
                            <td>Epoch</td>
                            <td>
                                <nuxt-link :to="{name: 'epochs-slug', params: {slug: Math.ceil(block.number / 900)}}">
                                    {{ Math.ceil(block.number / 900) }}</nuxt-link>
                            </td>
                        </tr>
                        <tr v-if="timestamp_moment">
                            <td>TimeStamp</td>
                            <td>{{ timestamp_moment }}</td>
                        </tr>
                        <tr>
                            <td>Transactions</td>
                            <td>{{ (block.e_tx || block.e_tx >= 0) ? block.e_tx : block.transactions.length }}
                                transactions</td>
                        </tr>
                        <tr>
                            <td>Hash</td>
                            <td>
                                <read-more
                                    :text="block.hash"
                                    class="d-sm-none"/>
                                <read-more
                                    :text="block.hash"
                                    :max-chars="20"
                                    class="d-none d-sm-block d-md-none"/>
                                <read-more
                                    :text="block.hash"
                                    :max-chars="40"
                                    class="d-none d-md-block d-lg-none"/>
                                <span class="d-none d-lg-block">{{ block.hash }}</span>
                            </td>
                        </tr>
                        <tr>
                            <td>Parent Hash</td>
                            <td>
                                <nuxt-link :to="{name: 'blocks-slug', params: {slug: block.number - 1}}">
                                    <read-more
                                        :text="block.parentHash"
                                        class="d-sm-none"/>
                                    <read-more
                                        :text="block.parentHash"
                                        :max-chars="20"
                                        class="d-none d-sm-block d-md-none"/>
                                    <read-more
                                        :text="block.parentHash"
                                        :max-chars="40"
                                        class="d-none d-md-block d-lg-none"/>
                                    <span class="d-none d-lg-block">{{ block.parentHash }}</span>
                                </nuxt-link>
                            </td>
                        </tr>
                        <tr>
                            <td>Created By</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug', params: {slug: block.signer}}"
                                    class="d-sm-none">
                                    <read-more
                                        v-if="block.signer"
                                        :text="block.signer"/>
                                    <read-more
                                        v-else
                                        :text="block.miner"/>
                                </nuxt-link>
                                <nuxt-link
                                    :to="{name: 'address-slug', params: {slug: block.signer}}"
                                    class="d-none d-sm-block d-md-none">
                                    <read-more
                                        v-if="block.signer"
                                        :text="block.signer"
                                        :max-chars="20"/>
                                    <read-more
                                        v-else
                                        :text="block.miner"
                                        :max-chars="20"/>
                                </nuxt-link>
                                <nuxt-link
                                    :to="{name: 'address-slug', params: {slug: block.signer}}"
                                    class="d-none d-md-block">
                                    <span
                                        v-if="block.signer"
                                        :text="block.signer"
                                        :maxChars="20">{{ block.signer }}</span>
                                    <span
                                        v-else
                                        :maxChars="20">{{ block.miner }}</span>
                                </nuxt-link>
                            </td>
                        </tr>
                        <tr>
                            <td>Verified By</td>
                            <td>
                                <span
                                    v-if="block.m2 !== 'N/A'">
                                    <nuxt-link
                                        :to="{name: 'address-slug', params: {slug: block.m2}}"
                                        class="d-sm-none">
                                        <read-more
                                            :text="block.m2"/>
                                    </nuxt-link>
                                    <nuxt-link
                                        :to="{name: 'address-slug', params: {slug: block.m2}}"
                                        class="d-none d-sm-block d-md-none">
                                        <read-more
                                            :text="block.m2"
                                            :max-chars="20"/>
                                    </nuxt-link>
                                    <nuxt-link
                                        :to="{name: 'address-slug', params: {slug: block.m2}}"
                                        class="d-none d-md-block">
                                        <span
                                            :maxChars="20">{{ block.m2 }}</span>
                                    </nuxt-link>
                                </span>
                                <span
                                    v-else>{{ block.m2 }}</span>
                            </td>
                        </tr>
                        <tr>
                            <td>Finality</td>
                            <td>{{ block.finality }} %</td>
                        </tr>
                        <tr>
                            <td>Gas Used</td>
                            <td>{{ formatNumber(block.gasUsed) }}</td>
                        </tr>
                        <tr>
                            <td>Gas Limit</td>
                            <td>{{ formatNumber(block.gasLimit) }}</td>
                        </tr>
                        <!--tr>
                            <td>Nonce</td>
                            <td>
                                <read-more
                                    :text="block.nonce"
                                    class="d-sm-none"/>
                                <read-more
                                    :text="block.nonce"
                                    :max-chars="20"
                                    class="d-none d-sm-block"/>
                            </td>
                        </tr-->
                        <tr>
                            <td>Extra Data</td>
                            <td>
                                <read-more
                                    :text="block.extraData"
                                    class="d-sm-none"/>
                                <read-more
                                    :text="block.extraData"
                                    :max-chars="20"
                                    class="d-none d-sm-block d-md-none"/>
                                <read-more
                                    :text="block.extraData"
                                    :max-chars="40"
                                    class="d-none d-md-block"/>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <!--:title="'Transactions (' + formatNumber(totalTxsCount) + ')'"-->
        <b-tabs
            ref="allTabs"
            v-model="tabIndex"
            class="wethio-tabs"
            @input="onSwitchTab">
            <b-tab
                lazy
                title="Transactions"
                href="#transactions">
                <table-tx-by-block
                    v-if="hashTab === '#transactions'"
                    :block="block.number"
                    :block-timestamp="block.timestamp"
                    :parent="'transactions'"
                    :page="this"/>
            </b-tab>
            <!--:title="'BlockSigner (' + formatNumber(blockSignerCount) + ')'"-->
            <b-tab
                lazy
                title="BlockSigner"
                href="#blockSigner">
                <block-signer
                    v-if="hashTab === '#blockSigner'"
                    :block="number"
                    :parent="'blockSigner'"
                    :page="this"/>
            </b-tab>
            <b-tab
                v-if="block.slashedNode"
                lazy
                title="SlashedNode"
                href="#slashedNode">
                <div
                    v-if="block.slashedNode.length === 0"
                    class="wethio-empty">
                    <i class="fa fa-cube wethio-empty__icon"/>
                    <p class="wethio-empty__description">No slashed node</p>
                </div>
                <p
                    v-if="block.slashedNode.length > 0"
                    class="wethio-total-items">{{ _nFormatNumber('node', 'nodes', block.slashedNode.length) }}</p>
                <table
                    v-if="block.slashedNode.length > 0"
                    class="wethio-table">
                    <thead>
                        <tr><th>Coinbase</th></tr>
                    </thead>
                    <tbody>
                        <tr
                            v-for="(item, index) in block.slashedNode"
                            :key="index">
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug', params: {slug: item}}">{{ item }}</nuxt-link>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </b-tab>
            <b-tab
                v-if="block.validators"
                lazy
                title="List Validators"
                href="#validators">
                <div
                    v-if="block.validators.length === 0"
                    class="wethio-empty">
                    <i class="fa fa-cube wethio-empty__icon"/>
                    <p class="wethio-empty__description">No slashed node</p>
                </div>
                <p
                    v-if="block.validators.length > 0"
                    class="wethio-total-items">
                    {{ _nFormatNumber('validator', 'validators', block.validators.length) }}</p>
                <table
                    v-if="block.validators.length > 0"
                    class="wethio-table">
                    <thead>
                        <tr><th>Coinbase</th></tr>
                    </thead>
                    <tbody>
                        <tr
                            v-for="(item, index) in block.validators"
                            :key="index">
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug', params: {slug: item}}">{{ item }}</nuxt-link>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </b-tab>
            <b-tab
                v-if="block.randoms"
                lazy
                title="List Randomize"
                href="#randoms">
                <div
                    v-if="block.randoms.length === 0"
                    class="wethio-empty">
                    <i class="fa fa-cube wethio-empty__icon"/>
                    <p class="wethio-empty__description">No slashed node</p>
                </div>
                <p
                    v-if="block.randoms.length > 0"
                    class="wethio-total-items">
                    {{ _nFormatNumber('number', 'numbers', block.randoms.length) }}</p>
                <table
                    v-if="block.validators.length > 0"
                    class="wethio-table">
                    <thead>
                        <tr><th>Random number</th></tr>
                    </thead>
                    <tbody>
                        <tr
                            v-for="(item, index) in block.randoms"
                            :key="index">
                            <td>{{ item }}</td>
                        </tr>
                    </tbody>
                </table>
            </b-tab>
        </b-tabs>
    </section>
</template>
<script>
import mixin from '~/plugins/mixin'
import TableTxByBlock from '~/components/TableTxByBlock'
import ReadMore from '~/components/ReadMore'
import BlockSigner from '~/components/BlockSigner'

export default {
    components: {
        TableTxByBlock,
        ReadMore,
        BlockSigner
    },
    mixins: [mixin],
    data () {
        return {
            number: null,
            block: null,
            epoch: null,
            timestamp_moment: null,
            loading: true,
            totalTxsCount: 0,
            blockSignerCount: 0,
            tabIndex: 0
        }
    },
    computed: {
        hashTab () {
            return this.$route.hash || '#transactions'
        }
    },
    created () {
        const number = this.$route.params.slug
        if (number) {
            this.number = number.toString()
        }
    },
    async mounted () {
        try {
            this.loading = true

            // Init breadcrumbs data.
            this.$store.commit('breadcrumb/setItems', {
                name: 'blocks-slug',
                to: { name: 'blocks-slug', params: { slug: this.number } }
            })

            const responses = await Promise.all([
                this.$axios.get('/api/blocks/' + this.$route.params.slug)
            ])

            this.block = responses[0].data
            const moment = this.$moment(responses[0].data.timestamp)
            this.timestamp_moment = `${moment.fromNow()} (${moment})`

            this.totalTxsCount = this.block.e_tx

            this.loading = false
        } catch (error) {
            console.log(error)
        }
    },
    methods: {
        onSwitchTab: function () {
            const allTabs = this.$refs.allTabs
            if (allTabs) {
                const value = this.tabIndex
                const location = window.location
                location.hash = allTabs.tabs[value].href
            }
        }
    },
    head () {
        return {
            title: 'Block ' + this.$route.params.slug + ' Info'
        }
    }
}
</script>
