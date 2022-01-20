<template>
    <section>
        <table-token-tx-zrc21/>
    </section>
</template>

<script>
import TableTokenTxTrc21 from '~/components/TableTokenTxTrc21'

export default {
    components: {
        TableTokenTxTrc21
    },
    async mounted () {
        // Init breadcrumbs data.
        this.$store.commit('breadcrumb/setItems', { name: 'tokentxs-zrc21', to: { name: 'tokentxs-zrc21' } })
    },
    head () {
        return {
            title: 'Token ZRC21 Transfers'
        }
    }
}
</script>

<style scoped>

</style>
