<template>
    <div
        v-if="loading"
        :class="(loading ? 'wethio-loading wethio-loading--full' : '')"/>
    <section v-else>
        <div class="card wethio-card">
            <div class="wethio-card__header">
                <h3 class="wethio-card__headline">
                    Hash:
                    <read-more
                        :text="hash"
                        class="d-sm-none"/>
                    <read-more
                        :text="hash"
                        :max-chars="20"
                        class="d-none d-sm-inline-block d-lg-none"/>
                    <read-more
                        :text="hash"
                        :max-chars="30"
                        class="d-none d-lg-inline-block d-xl-none"/>
                    <span class="d-none d-xl-inline-block">{{ hash }}</span>
                </h3>
            </div>
            <div class="wethio-card__body">
                <table class="wethio-card__table">
                    <tbody>
                        <tr>
                            <td>Transaction Hash</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'txs-slug', params: {slug: lendingTrade.txHash}}">
                                    <read-more
                                        :text="lendingTrade.txHash"
                                        class="d-sm-none"/>
                                    <read-more
                                        :text="lendingTrade.txHash"
                                        :max-chars="20"
                                        class="d-none d-sm-block d-md-none"/>
                                    <read-more
                                        :text="lendingTrade.txHash"
                                        :max-chars="40"
                                        class="d-none d-md-block d-lg-none"/>
                                    <span class="d-none d-lg-block">{{ lendingTrade.txHash }}</span>

                                </nuxt-link>
                            </td>
                        </tr>
                        <tr>
                            <td>Lending Trade Status</td>
                            <td>
                                <span
                                    v-if="lendingTrade.status === 'OPEN'"
                                    class="text-success">OPEN</span>
                                <span
                                    v-else-if="lendingTrade.status === 'CLOSED'"
                                    class="text-danger">CLOSED</span>
                                <span
                                    v-else-if="lendingTrade.status === 'LIQUIDATED'"
                                    class="text-danger">LIQUIDATED</span>
                                <span
                                    v-else
                                    class="text-purple">{{ lendingTrade.status }}</span>
                            </td>
                        </tr>
                        <tr v-if="lendingTrade.status === 'LIQUIDATED' && lendingTrade.liquidated">
                            <td>Liquidation Info</td>
                            <td>
                                <p v-if="lendingTrade.liquidated.recallAmount">
                                    Recall Amount: {{ lendingTrade.liquidated.recallAmount }}
                                    <nuxt-link
                                        v-if="lendingTrade.collateralToken !== wethioNativeToken"
                                        :to="{
                                            name: 'tokens-slug',
                                            params: {slug: lendingTrade.collateralToken.toLowerCase()}}">
                                        {{ lendingTrade.collateralSymbol }}</nuxt-link>
                                </p>
                                <p v-if="lendingTrade.liquidated.liquidationAmount">
                                    Liquidation Amount: {{ lendingTrade.liquidated.liquidationAmount }}
                                    <nuxt-link
                                        v-if="lendingTrade.collateralToken !== wethioNativeToken"
                                        :to="{
                                            name: 'tokens-slug',
                                            params: {slug: lendingTrade.collateralToken.toLowerCase()}}">
                                        {{ lendingTrade.collateralSymbol }}</nuxt-link>
                                </p>
                                <p v-if="lendingTrade.liquidated.collateralPrice">
                                    Collateral Price: {{ lendingTrade.liquidated.collateralPrice }}
                                    <nuxt-link
                                        v-if="lendingTrade.lendingToken !== wethioNativeToken"
                                        :to="{
                                            name: 'tokens-slug',
                                            params: {slug: lendingTrade.lendingToken.toLowerCase()}}">
                                        {{ lendingTrade.lendingSymbol }}</nuxt-link>
                                    <span v-else>ZYN</span>
                                </p>
                                <p v-if="lendingTrade.liquidated.reason">
                                    Reason: {{ lendingTrade.liquidated.reason }}</p>
                            </td>
                        </tr>
                        <tr>
                            <td>Borrowing Relayer</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug',
                                          params: {slug:lendingTrade.borrowingRelayer.toLowerCase()}}"
                                    class="mr-1">{{ lendingTrade.borrowingRelayer.toLowerCase() }}
                                </nuxt-link>
                            </td>
                        </tr>
                        <tr>
                            <td>Investing Relayer</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug',
                                          params: {slug:lendingTrade.investingRelayer.toLowerCase()}}"
                                    class="mr-1">{{ lendingTrade.investingRelayer.toLowerCase() }}
                                </nuxt-link>
                            </td>
                        </tr>
                        <tr>
                            <td>Borrower</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug',
                                          params: {slug:lendingTrade.borrower.toLowerCase()}}"
                                    class="mr-1">{{ lendingTrade.borrower.toLowerCase() }}
                                </nuxt-link>
                            </td>
                        </tr>
                        <tr>
                            <td>Investor</td>
                            <td>
                                <nuxt-link
                                    :to="{name: 'address-slug',
                                          params: {slug:lendingTrade.investor.toLowerCase()}}"
                                    class="mr-1">{{ lendingTrade.investor.toLowerCase() }}
                                </nuxt-link>
                            </td>
                        </tr>
                        <tr>
                            <td>Amount</td>
                            <td>
                                {{ formatNumber(lendingTrade.amount) }}
                                <nuxt-link
                                    v-if="lendingTrade.lendingToken !== wethioNativeToken"
                                    :to="{name: 'tokens-slug',
                                          params: {slug: lendingTrade.lendingToken.toLowerCase()}}">
                                    {{ lendingTrade.lendingSymbol }}</nuxt-link>
                                <span v-else>ZYN</span>
                                (Borrower Fee:
                                {{ formatNumber(lendingTrade.borrowingFee) }}
                                <nuxt-link
                                    v-if="lendingTrade.lendingToken !== wethioNativeToken"
                                    :to="{name: 'tokens-slug',
                                          params: {slug: lendingTrade.lendingToken.toLowerCase()}}">
                                    {{ lendingTrade.lendingSymbol }}</nuxt-link>
                                <span v-else>ZYN</span>)
                            </td>
                        </tr>
                        <tr>
                            <td>Interest</td>
                            <td>{{ lendingTrade.interest }} %</td>
                        </tr>
                        <tr>
                            <td>Collateral Locked</td>
                            <td>
                                {{ formatNumber(lendingTrade.collateralLockedAmount) }}
                                <nuxt-link
                                    v-if="lendingTrade.collateralToken !== wethioNativeToken"
                                    :to="{name: 'tokens-slug',
                                          params: {slug: lendingTrade.collateralToken.toLowerCase()}}">
                                    {{ lendingTrade.collateralSymbol }}</nuxt-link>
                                <span v-else>ZYN</span>
                                (Price: {{ lendingTrade.collateralPrice }}
                                <nuxt-link
                                    v-if="lendingTrade.lendingToken !== wethioNativeToken"
                                    :to="{name: 'tokens-slug',
                                          params: {slug: lendingTrade.lendingToken.toLowerCase()}}">
                                    {{ lendingTrade.lendingSymbol }}</nuxt-link>
                                <span v-else>ZYN</span>))
                            </td>
                        </tr>
                        <tr>
                            <td>Auto</td>
                            <td>{{ lendingTrade.autoTopUp }}</td>
                        </tr>
                        <tr>
                            <td>Maturity Date
                            </td>
                            <td>{{ new Date(lendingTrade.liquidationTime * 1000) }}</td>
                        </tr>
                        <tr>
                            <td>Term
                            </td>
                            <td>{{ lendingTrade.term / 86400 }}
                                <span v-if="lendingTrade.term / 86400 <= 1">day</span>
                                <span v-else>days</span>
                                ({{ lendingTrade.term }} seconds)</td>
                        </tr>
                        <tr>
                            <td>Age</td>
                            <td>
                                <span
                                    v-b-tooltip.hover
                                    :title="$moment(lendingTrade.createdAt).format('lll')">
                                    {{ $moment(lendingTrade.createdAt).fromNow() }}</span>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <b-tabs
            ref="allTabs"
            class="wethio-tabs">
            <b-tab
                id="lendingOrders"
                lazy
                title="Lending Orders"
                href="#lendingOrders">
                <table-lending-order :trade-hash="hash"/>
            </b-tab>
            <b-tab
                id="lendingRepay"
                lazy
                title="Lending Repay"
                href="#lendingRepay">
                <table-lending-repay :trade-hash="hash"/>
            </b-tab>
            <b-tab
                id="lendingTopup"
                lazy
                title="Lending Topup"
                href="#lendingTopup">
                <table-lending-topup :trade-hash="hash"/>
            </b-tab>
            <b-tab
                id="lendingRecall"
                lazy
                title="Lending Recalls"
                href="#lendingRecall">
                <table-lending-recall :trade-hash="hash"/>
            </b-tab>
        </b-tabs>
    </section>
</template>
<script>
import mixin from '~/plugins/mixin'
import ReadMore from '~/components/ReadMore'
import TableLendingOrder from '~/components/TableLendingOrder'
import TableLendingRepay from '~/components/TableLendingRepay'
import TableLendingTopup from '~/components/TableLendingTopup'
import TableLendingRecall from '~/components/TableLendingRecall'

export default {
    components: {
        TableLendingOrder,
        TableLendingRepay,
        TableLendingTopup,
        TableLendingRecall,
        ReadMore
    },
    mixins: [mixin],
    data () {
        return {
            hash: null,
            lendingTrade: {},
            loading: true,
            wethioNativeToken: process.env.WETHIO_NATIVE_TOKEN
        }
    },
    created () {
        this.hash = this.$route.params.slug
    },
    async mounted () {
        try {
            this.loading = true

            // Init breadcrumbs data.
            this.$store.commit('breadcrumb/setItems', {
                name: 'lending-trades-slug',
                to: { name: 'lending-trades-slug', params: { slug: this.hash } }
            })

            const { data } = await this.$axios.get('/api/lending/trades/' + this.hash)

            this.lendingTrade = data

            this.loading = false
        } catch (error) {
            console.log(error)
        }
    },
    head () {
        return {
            title: 'Lending Trade ' + this.$route.params.slug + ' detail'
        }
    }
}
</script>
