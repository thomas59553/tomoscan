<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <div class="card wethio-card wethio-card--address">
            <div class="wethio-card__header">
                <h3
                    :class="`wethio-card__headline
                    ${(address && address.isContract ? ' wethio-card__headline--is-contract' : '')}`">
                    <span v-if="address && address.isContract">Contract: </span>
                    <read-more
                        :text="address.hash"
                        class="d-sm-none"/>
                    <read-more
                        :text="address.hash"
                        :max-chars="20"
                        class="d-none d-sm-inline-block d-lg-none"/>
                    <read-more
                        :text="address.hash"
                        :max-chars="30"
                        class="d-none d-lg-inline-block d-xl-none"/>
                    <span class="d-none d-xl-inline-block">{{ address.hash }}</span>
                </h3>
            </div>
            <div class="wethio-card__body">
                <table class="wethio-card__table">
                    <tbody>
                        <tr v-if="address.accountName">
                            <td>Account Name</td>
                            <td>
                                <span>{{ address.accountName }}</span>
                            </td>
                        </tr>
                        <tr>
                            <td>ZYN Balance</td>
                            <td>
                                <span>{{ formatUnit(toTomo(address.balance, 18)) }}</span>
                            </td>
                        </tr>
                        <tr>
                            <td>ZYN USD Value</td>
                            <td>
                                <span>{{ formatNumber(usdPrice * toTomoNumber(address.balance)) }} (price from
                                    <a
                                        target="_blank"
                                        href="https://www.coingecko.com/en/coins/tomochain">CoinGecko</a>)</span>
                            </td>
                        </tr>
                        <tr>
                            <td>Transactions</td>
                            <td>
                                <span>{{ formatNumber(totalInTx + totalOutTx) }}</span> txns
                            </td>
                        </tr>
                        <tr v-if="address && !address.isContract">
                            <td>Code</td>
                            <td>
                                <read-more
                                    :text="address.code"
                                    class="d-sm-none"/>
                                <read-more
                                    :text="address.code"
                                    :max-chars="20"
                                    class="d-none d-sm-inline-block d-lg-none"/>
                                <read-more
                                    :text="address.code"
                                    :max-chars="30"
                                    class="d-none d-lg-inline-block d-xl-none"/>
                                <span class="d-none d-xl-inline-block">{{ address.code }}</span>
                            </td>
                        </tr>
                        <tr v-if="address && address.token">
                            <td>Token Contract</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'tokens-slug', params: {slug: address.token.hash}}">
                                    {{ trimWord(address.token.name) }}({{ trimWord(address.token.symbol) }})
                                </nuxt-link>
                            </td>
                        </tr>
                        <tr v-if="address && address.contractCreation">
                            <td>Contract Creator</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug', params: {slug: address.contractCreation}}"
                                    class="text-truncate">{{ address.contractCreation }}</nuxt-link>
                                <span>&nbsp;at txns&nbsp;</span>
                                <span v-if="address.fromTxn">
                                    <nuxt-link
                                        :to="{name: 'txs-slug', params: {slug: address.fromTxn}}"
                                        class="text-truncate">{{ address.fromTxn }}</nuxt-link>
                                </span>
                            </td>
                        </tr>
                        <tr
                            v-if="address && address.isContract && ! address.contract"
                            class="is-contract-message">
                            <td>
                                <div>Are you the contract creator?
                                    <nuxt-link :to="{name: 'contracts-verify', query: {address: hash}}">
                                        Verify And Publish
                                    </nuxt-link>
                                    your contract source code today!
                                </div>
                            </td>
                        </tr>
                        <tr>
                            <td/>
                            <td>
                                <nuxt-link
                                    :to="{name: 'download', query: {address: address.hash}}">Download Data</nuxt-link>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <div class="text-center text-lg-right wethio-qrcode">
                    <div class="text-center">
                        <button
                            v-clipboard="address.hash"
                            type="button"
                            class="btn btn-sm mr-2 code-actions__copy"
                            @success="copyAddress">
                            <i class="fa fa-clipboard"/> Copy
                        </button>
                    </div>
                    <div class="text-center">
                        <vue-qrcode
                            :value="address.hash"
                            :options="{size: 250}"
                            class="img-fluid"/>
                    </div>
                </div>
            </div>
        </div>

        <b-tabs
            ref="allTabs"
            v-model="tabIndex"
            class="wethio-tabs">
            <b-tab
                id="transactions"
                title="Transactions"
                href="#transactions">
                <table-tx-by-account
                    :address="hash"
                    :type="'all'"
                    :parent="'#inTransactions'"
                    :page="this"/>
            </b-tab>
            <b-tab
                id="internalTransactions"
                lazy
                title="Internal Transactions"
                href="#internalTransactions">
                <table-internal-tx
                    :address="hash"
                    :parent="'#internalTransactions'"
                    :page="this"/>
            </b-tab>
            <b-tab
                v-if="address && address.hasTrc20"
                id="zrc20Holding"
                lazy
                title="ZRC20 Holding"
                href="#zrc20Holding">
                <table-tokens-by-account
                    :holder="hash"
                    :token-type="'zrc20'"
                    :parent="'zrc20Holding'"
                    :page="this"/>
            </b-tab>
            <b-tab
                id="zrc21Holding"
                lazy
                title="ZRC21 Holding"
                href="#zrc21Holding">
                <table-tokens-by-account
                    :holder="hash"
                    :token-type="'zrc21'"
                    :parent="'zrc21Holding'"
                    :page="this"/>
            </b-tab>
            <b-tab
                v-if="address && address.hasTrc721"
                id="trc721Inventory"
                lazy
                title="TRC721 Inventory"
                href="#trc721Inventory">
                <table-tokens-by-account
                    :holder="hash"
                    :token-type="'trc721'"
                    :parent="'trc721Inventory'"
                    :page="this"/>
            </b-tab>
            <b-tab
                v-if="address && address.isContract && smartContract"
                id="code"
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
                v-if="address && address.isContract && smartContract"
                id="readContract"
                lazy
                title="Read Contract"
                href="#readContract">
                <read-contract
                    :contract="hash"/>
            </b-tab>
            <b-tab
                v-if="!address.isContract"
                id="rewards"
                lazy
                title="Rewards"
                href="#rewards">
                <table-reward
                    :address="hash"
                    :parent="'rewards'"
                    :page="this"/>
            </b-tab>
            <b-tab
                id="tradeHistories"
                lazy
                title="Trades History"
                href="#tradeHistories">
                <table-trade-history
                    :user-address="hash"/>
            </b-tab>
            <b-tab
                id="lendingTrades"
                lazy
                title="Lending Trades"
                href="#lendingTrades">
                <table-lending-trade
                    :user-address="hash"/>
            </b-tab>
            <b-tab
                id="lendingTopup"
                lazy
                title="Lending Topups"
                href="#lendingTopup">
                <table-lending-topup
                    :user-address="hash"/>
            </b-tab>
            <b-tab
                id="lendingRepay"
                lazy
                title="Lending Repays"
                href="#lendingRepay">
                <table-lending-repay
                    :user-address="hash"/>
            </b-tab>
            <b-tab
                id="lendingRecall"
                lazy
                title="Lending Recalls"
                href="#lendingRecall">
                <table-lending-recall
                    :user-address="hash"/>
            </b-tab>
        </b-tabs>
    </section>
</template>
<script>
import mixin from '~/plugins/mixin'
import TableTxByAccount from '~/components/TableTxByAccount'
import TableInternalTx from '~/components/TableInternalTx'
import TableTokensByAccount from '~/components/TableTokensByAccount'
import TableTradeHistory from '~/components/TableTradeHistory'
import TableLendingTrade from '~/components/TableLendingTrade'
import TableLendingTopup from '~/components/TableLendingTopup'
import TableLendingRepay from '~/components/TableLendingRepay'
import TableLendingRecall from '~/components/TableLendingRecall'
import ReadMore from '~/components/ReadMore'
import VueQrcode from '@xkeshi/vue-qrcode'
import ReadContract from '~/components/ReadContract'
import TableReward from '~/components/TableReward'
import ReadSourceCode from '~/components/ReadSourceCode'

export default {
    components: {
        TableLendingRecall,
        ReadSourceCode,
        TableTxByAccount,
        TableInternalTx,
        TableTokensByAccount,
        ReadMore,
        VueQrcode,
        ReadContract,
        TableReward,
        TableTradeHistory,
        TableLendingTrade,
        TableLendingTopup,
        TableLendingRepay
    },
    mixins: [mixin],
    data: () => ({
        hash: null,
        address: null,
        smartContract: null,
        loading: true,
        totalInTx: 0,
        totalOutTx: 0,
        tabIndex: 0
    }),
    computed: {
        usdPrice () {
            return this.$store.state.app.usdPrice
        },
        codemirror () {
            return [
                this.$refs.tomoCmSourceCode.codemirror,
                this.$refs.tomoCmAbiCode.codemirror,
                this.$refs.tomoCmCode.codemirror
            ]
        },
        hashTab () {
            return this.$route.hash || '#inTransactions'
        }
    },
    created () {
        const hash = this.$route.params.slug
        if (hash) {
            this.hash = hash.toLowerCase()
        }
    },
    mounted () {
        try {
            // Init breadcrumbs data.
            this.$store.commit('breadcrumb/setItems', {
                name: 'address-slug',
                to: { name: 'address-slug', params: { slug: this.hash } }
            })

            this.getAccountFromApi()
            this.getUSDPrice()
        } catch (error) {
            console.log(error)
        }
    },
    methods: {
        async getAccountFromApi () {
            this.loading = true

            const responses = await Promise.all([
                this.$axios.get('/api/accounts/' + this.hash)
            ])

            this.address = responses[0].data
            this.smartContract = responses[0].data.contract

            this.loading = false
        },
        async getUSDPrice () {
            this.$store.dispatch('app/getUSDPrice')
        },
        copyAddress () {
            this.$toast.show('Copied')
        }
    },
    head () {
        return {
            title: 'Address ' + this.hash
        }
    }
}
</script>
