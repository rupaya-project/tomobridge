<template>
    <b-container class="step-two text-center">
        <div style="margin-bottom: 30px">
            <h3 class="step-three__title">{{ $t('verifyTx') }}</h3>
            <p>
                {{ $t('txHash2') }}:<br>
                <a
                    :href="config.tomoscanUrl + '/txs/' + txHash"
                    class="step-three__tx-hash-link text-truncate"
                    target="_blank">
                    {{ txHash }}
                </a>
            </p>
        </div>
        <div class="step-three__progress">
            <div class="progress-bar">
                <div class="progress-bar__inner">
                    <div
                        :style="`width: ${calculatePercentage(confirmation, requiredConfirm)}%;`"
                        class="progress-bar__bar">
                        <span class="progress-bar__number text-primary">
                            {{ calculatePercentage(confirmation, requiredConfirm) }}%</span>
                    </div>
                </div>
                <span class="progress-bar__total">{{ requiredConfirm }} {{ $t('blocks') }}</span>
            </div>
            <!-- <div class="step-three__fee text-primary">
                Fee: 1 TOMO
            </div> -->
        </div>
        <!-- <span>({{ $t('doneCheckingTx') }}
            <b-link
                :to="'/txs'">
                Transaction History
            </b-link>
            )
        </span> -->
        <b-button
            class="step-two__button btn--big"
            @click="back">{{ $t('backBtn') }}</b-button>
    </b-container>
</template>

<script>
import axios from 'axios'
export default {
    name: 'App',
    components: {
    },
    props: {
        parent: {
            type: Object,
            default: () => {}
        }
    },
    data () {
        return {
            interval: '',
            requiredConfirm: 30,
            confirmation: 0,
            config: {},
            txHash: ''
        }
    },
    async updated () { },
    destroyed () {
        if (this.interval) {
            clearInterval(this.interval)
        }
    },
    created: async function () {
        const parent = this.parent
        this.config = parent.config
        this.txHash = parent.transactionHash
        const receipt = await this.web3.eth.getTransactionReceipt(this.txHash)
        const signedBlock = receipt.blockNumber

        this.interval = setInterval(async () => {
            const currentBlock = await this.web3.eth.getBlockNumber()
            this.confirmation = currentBlock - signedBlock
            if (this.confirmation >= this.requiredConfirm) {
                setTimeout(() => {
                    clearInterval(this.interval)
                    parent.step++
                }, 2000)
            }
        }, 5000)
    },
    methods: {
        back () {
            const par = this.parent
            par.step--
        },
        async scanTX () {
            const parent = this.parent
            const address = this.$store.state.address || ''
            const wrapToken = parent.toWrapToken
            const txData = await axios.get(
                `/api/wrap/getTransaction/withdraw/${wrapToken.name}/${address}`
            )
            if (txData && txData.data) {
                return txData.data
            }
        },
        calculatePercentage (current, total) {
            if (current >= total) {
                return 100
            } else {
                return Math.floor((current * 100) / total)
            }
        }
    }
}
</script>
