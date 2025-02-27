<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <div class="card wethio-card wethio-card--token">
            <div class="wethio-card__header">
                <h2 class="wethio-card__headline">
                    <img
                        v-if="checkAvatarExist(tokenBranch, hash)"
                        :src="`https://raw.githubusercontent.com/tomochain/tokens/${tokenBranch}/tokens/${hash}.png`"
                        width="35px">
                    {{ tokenName }}&nbsp;</h2>
                <i
                    v-if="isVerified"
                    class="fa fa-check-circle token-status"
                    aria-hidden="true"/>
                <h6 class="mb-0">{{ symbol }}</h6>
            </div>
            <div class="wethio-card__body">
                <div
                    v-if="isPhising"
                    class="alert alert-danger">
                    Warning! There are reports that this address was used in a Phishing scam.
                    Please exercise caution when interacting with this address.</div>
                <b-row>
                    <b-col md="6">
                        <table
                            v-if="token"
                            class="wethio-card__table">
                            <tbody>
                                <tr>
                                    <td>Total Supply</td>
                                    <td>{{ formatUnit(formatNumber(token.totalSupplyNumber), symbol) }}</td>
                                </tr>
                                <tr>
                                    <td>Holders</td>
                                    <td>{{ formatNumber(holdersCount) }}
                                        {{ holdersCount > 1 ? 'addresses' : 'address' }}</td>
                                </tr>
                                <tr>
                                    <td>Transfers</td>
                                    <td>{{ formatNumber(tokenTxsCount) }}</td>
                                </tr>
                                <tr>
                                    <td>Official Site</td>
                                    <td
                                        v-if="moreInfo">
                                        <a
                                            :href="moreInfo.website"
                                            target="_blank"
                                            class="text-truncate">{{ moreInfo.website }}</a>
                                    </td>
                                    <td
                                        v-else>
                                        Not Available,
                                        <a
                                            href="https://github.com/tomochain/tokens"
                                            target="_blank"
                                            class="text-truncate">Update</a> ?
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </b-col>
                    <b-col md="6">
                        <table
                            v-if="token"
                            class="wethio-card__table">
                            <tbody>
                                <tr>
                                    <td>Contract</td>
                                    <td>
                                        <nuxt-link
                                            :to="{name: 'address-slug', params: {slug: token.hash}}"
                                            class="text-truncate">{{ token.hash }}</nuxt-link>
                                    </td>
                                </tr>
                                <tr>
                                    <td>Token type</td>
                                    <td>{{ token.type.toUpperCase() }}</td>
                                </tr>
                                <tr v-if="token.type !== 'trc721'">
                                    <td>Decimal</td>
                                    <td>{{ token.decimals }}</td>
                                </tr>
                                <tr>
                                    <td>Links</td>
                                    <td class="token-info-link">
                                        <ul
                                            v-if="moreInfo && moreInfo.communities"
                                            class="list-inline s-icons">
                                            <li
                                                v-for="(community, key) in moreInfo.communities"
                                                :key="key"
                                                class="list-inline-item">
                                                <a
                                                    :href="community">
                                                    <i :class="'fa fa-' + key"/>
                                                </a>
                                            </li>
                                        </ul>
                                        <span v-else>
                                            Not Available,
                                            <a
                                                href="https://github.com/tomochain/tokens"
                                                target="_blank"
                                                class="text-truncate">Update</a> ?
                                        </span>
                                    </td>
                                </tr>
                                <tr>
                                    <td>Filtered By</td>
                                    <td>
                                        <div class="input-group input-group-sm filter-address">
                                            <input
                                                v-model="addressFilter"
                                                type="text"
                                                class="form-control form-control-sm"
                                                placeholder="Address"
                                                aria-label="Address"
                                                @keyup.enter="filterAddress(addressFilter, token.type)">
                                            <div class="input-group-append">
                                                <button
                                                    class="btn btn-primary btn-primary-sm"
                                                    type="button"
                                                    @click="filterAddress(addressFilter, token.type)">
                                                    <i class="fa fa-filter"/>
                                                </button>
                                            </div>
                                        </div>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </b-col>
                </b-row>
            </div>
        </div>

        <b-row>
            <b-col>
                <b-tabs
                    ref="allTabs"
                    v-model="tabIndex"
                    class="wethio-tabs">
                    <!--:title="'Token Transfers (' + formatNumber(tokenTxsCount) + ')'"-->
                    <b-tab
                        lazy
                        title="Token Transfers"
                        href="#tokenTransfers">
                        <table-token-tx
                            v-if="token.type === 'zrc20'"
                            :token="hash"
                            :parent="'#tokenTransfers'"
                            :page="this"/>
                        <table-token-tx-nft
                            v-if="token.type === 'trc721'"
                            :token="hash"
                            :parent="'#tokenTransfers'"
                            :page="this"/>
                        <table-token-tx-zrc21
                            v-if="token.type === 'zrc21'"
                            :token="hash"
                            :parent="'#tokenTransfers'"
                            :page="this"/>
                    </b-tab>
                    <!--:title="'Token Holders (' + formatNumber(holdersCount) + ')'"-->
                    <b-tab
                        lazy
                        title="Token Holders"
                        href="#tokenHolders">
                        <table-token-holder
                            v-if="token.type === 'zrc20'"
                            :address="hash"
                            :parent="'#tokenHolders'"
                            :page="this"/>
                        <table-token-nft-holder
                            v-if="token.type === 'trc721'"
                            :address="hash"
                            :parent="'#tokenHolders'"
                            :page="this"/>
                        <table-token-zrc21-holder
                            v-if="token.type === 'zrc21'"
                            :address="hash"
                            :parent="'#tokenHolders'"
                            :page="this"/>
                    </b-tab>
                    <b-tab
                        v-if="address && address.isContract && smartContract"
                        lazy
                        title="Code"
                        href="#code"
                        @click="refreshCodeMirror">
                        <read-source-code
                            ref="readSourceCode"
                            :token="hash"
                            :smartcontract="smartContract"
                            :address="address"/>
                    </b-tab>
                    <b-tab
                        v-if="smartContract"
                        lazy
                        title="Read Contract"
                        href="#readContract">
                        <read-contract
                            :contract="hash"/>
                    </b-tab>
                </b-tabs>
            </b-col>
        </b-row>
    </section>
</template>
<script>
import mixin from '~/plugins/mixin'
import TableTokenTx from '~/components/TableTokenTx'
import TableTokenTxNft from '~/components/TableTokenTxNft'
import TableTokenTxTrc21 from '~/components/TableTokenTxTrc21'
import TableTokenHolder from '~/components/TableTokenHolder'
import TableTokenNftHolder from '~/components/TableTokenNftHolder'
import TableTokenTrc21Holder from '~/components/TableTokenTrc21Holder'
import ReadContract from '~/components/ReadContract'
import ReadSourceCode from '~/components/ReadSourceCode'

export default {
    components: {
        ReadSourceCode,
        TableTokenTx,
        TableTokenTxNft,
        TableTokenTxTrc21,
        ReadContract,
        TableTokenHolder,
        TableTokenNftHolder,
        TableTokenTrc21Holder
    },
    mixins: [mixin],
    data () {
        return {
            hash: null,
            token: null,
            tokenName: null,
            symbol: null,
            loading: true,
            tokenTxsCount: 0,
            holdersCount: 0,
            moreInfo: null,
            isVerified: false,
            isPhising: false,
            addressFilter: null,
            address: null,
            smartContract: null,
            holderBalance: 0,
            tabIndex: 0,
            tokenBranch: process.env.TOKEN_BRANCH
        }
    },
    computed: {
        hashTab () {
            return this.$route.hash || '#tokenTransfers'
        }
    },
    created () {
        this.hash = this.$route.params.slug.toLowerCase()
    },
    async mounted () {
        try {
            const self = this

            self.loading = true

            // Init breadcrumbs data.
            this.$store.commit('breadcrumb/setItems', {
                name: 'tokens-slug',
                to: { name: 'tokens-slug', params: { slug: self.hash } }
            })

            const params = {}

            if (self.hash) {
                params.token = self.hash
            }

            params.list = 'token'
            const query = this.serializeQuery(params)

            const responses = await Promise.all([
                self.$axios.get('/api/tokens/' + self.hash),
                self.$axios.get('/api/counting' + '?' + query)
            ])

            self.token = responses[0].data
            self.tokenName = responses[0].data.name
            self.symbol = responses[0].data.symbol

            self.tokenTxsCount = responses[1].data.tokenTxs

            self.holdersCount = responses[1].data.tokenHolders

            self.loading = false
            self.isVerified = responses[0].data.isVerified
            self.isPhising = responses[0].data.isPhising
            self.moreInfo = responses[0].data.moreInfo
            self.getAccountFromApi()
        } catch (error) {
            console.log(error)
        }
    },
    methods: {
        async getAccountFromApi () {
            const self = this

            const { data } = await this.$axios.get('/api/accounts/' + self.hash)
            self.address = data
            self.smartContract = data.contract
        },
        onSwitchTab: function () {
            const allTabs = this.$refs.allTabs
            const location = window.location
            const value = this.tabIndex
            if (allTabs) {
                if (location.hash !== allTabs.tabs[value].href) {
                    this.$router.replace({
                        hash: allTabs.tabs[value].href
                    })
                } else {
                    location.hash = allTabs.tabs[value].href
                }
            }
        }
    },
    head () {
        return {
            title: 'Token ' + this.$route.params.slug + ' Info'
        }
    }
}
</script>
