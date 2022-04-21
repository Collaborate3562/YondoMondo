<template>
  <div id="staking">
    <div class="container d-flex flex-column align-items-center w-full">
      <div class="d-flex flex-column container-box-1 w-full" v-if="wageCount <= 1">
        <h3 class="text-white text-capitalize title">
          How much do you want to wager?
        </h3>
        <div>
          <a class="d-block me-3 me-xxl-4 wage-btn-area-1" style="cursor:pointer" @click="handleWager(1)">
            <img
              src="@assets/imgs/illustrations/wager_btn_1_0.png"
              v-if="isIncreaseWageActive"
              alt="twitter icon"
            />
            <img
              src="@assets/imgs/illustrations/wager_btn_1_1.png"
              v-else
              alt="twitter icon"
            />
          </a>
        </div>
        <div class="increase-btn-area-1">
          <a class="d-block me-3 me-xxl-4" style="cursor:pointer" @click="handleIncreaseWage">
            <img
              src="@assets/imgs/illustrations/increase_wage.png"
              v-if="isIncreaseWageActive"
              alt="twitter icon"
            />
            <img
              src="@assets/imgs/illustrations/increase_wage.png"
              v-else
              alt="twitter icon"
            />
          </a>
        </div>
      </div>
      <div v-else class="d-flex flex-column align-items-center">
        <span class="text-white text-capitalize caption"
          >Thank you for entering, The game begins on [date] at [time]. See you
          then.</span
        >
        <a
          style="cursor:pointer"
          class="d-block me-3  increase-btn-area-2 d-flex justify-content-center"
          @click="handleIncreaseWage"
        >
          <img
            src="@assets/imgs/illustrations/increase_wage.png"
            v-if="isIncreaseWageActive"
            alt="twitter icon"
          />
          <img
            src="@assets/imgs/illustrations/increase_wage.png"
            v-else
            alt="twitter icon"
          />
        </a>
        <h3 class="text-white text-capitalize title">
          How much do you want to wager?
        </h3>
        <div class="d-flex wage-btn-area-2">
          <div v-for="i in wageBtnArr[wageCount - 1]" :key="i">
            <a class="d-block me-3 me-xxl-4" style="margin-bottom: 35.98px; cursor:pointer" @click="handleWager(i)">
              <img
                :src="require(`@/assets/imgs/illustrations/wager_btn_${i}_0.png`)"
                v-if="isWageClicked"
                alt="twitter icon"
                v-bind:class="wageBtnSizeClass"
              />
              <img
                v-else
                alt="twitter icon"
                :src="require(`@/assets/imgs/illustrations/wager_btn_${i}_1.png`)"
                v-bind:class="wageBtnSizeClass"
              />
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ethers } from 'ethers';
import {mapState} from 'vuex';
import { NFTContract, TokenContract, BattleContract } from '@src/web3/web3.js';
export default {
  name: 'StakingComponent',
  data: function () {
    return {
      count: 1,
      isIncreaseWageActive: false,
      isWageClicked: false
    };
  },
  computed: {
    ...mapState(['currentAccount','connect']),
    wageBtnArr() {
      return [1, 2, 3, 5, 8, 10];
    },
    wageCount() {
      return this.count;
    },
    wageBtnSizeClass() {
      return this.count > 4 ? 'wage-btn-size-2' : 'wage-btn-size-1';
    },
    srcPath() {
      return '@assets/imgs/illustrations/wager_btn_1_0';
    }
  },
  methods: {
    increaseCount() {
      if (this.count <= 6) return this.count++;
      else return (this.count = 1);
    },
    handleIncreaseWage() {
      this.isIncreaseWageActive != this.isIncreaseWageActive;
      this.increaseCount();
    },
    async handleWager(val){
      if(!this.connect) {
        this.$toast.open({
          message: 'Please connect wallet.',
          type: 'error',
          duration: 2000
        });
      } else {
        const provider = new ethers.providers.Web3Provider(window.ethereum);

        const { chainId } = await provider.getNetwork();
        if(chainId !== 3) {
          this.$toast.open({
            message: 'Please select Ropsten TestNetwork.',
            type: 'error',
            duration: 2000
          });
          return;
        }

        const NFTBalance = await NFTContract.methods.balanceOf(this.currentAccount).call();
        if(Number(NFTBalance) === 0) {
          this.$toast.open({
            message: 'You have no NFT. For Staking you must have NFT character.',
            type: 'error',
            duration: 3000
          });
          return;
        }

        const tokenBalance = await TokenContract.methods.balanceOf(this.currentAccount).call();
        if(Number(ethers.utils.formatEther(String(tokenBalance))) < val * 1000) {
          this.$toast.open({
            message: 'You have insufficient $YONDO.',
            type: 'error',
            duration: 3000
          });
          return;
        }

        const createdTime = await BattleContract.methods.getStartTime().call();
        const now = Math.floor(Date.now() / 1000); // for testing
        console.log(now);
        // const now = await BattleContract.methods.getNow().call();

        const PER_WEEK = 60 * 60 * 24 * 7;
        const PER_DAY = 60 * 60 * 24;

        const round = await BattleContract.methods.getRound(1).call();
        console.log('round', round);

        console.log('createdTime', createdTime);
        console.log(Number(createdTime) + PER_WEEK * Math.floor(Number(round)) + (Number(PER_DAY) * 3));
        console.log('estimateTime', Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now);

        if(now < createdTime + (PER_WEEK * Number(round)) + (PER_DAY * 3)) {
          this.$toast.open({
            message: `Not ready time. Please wait ${Math.floor((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)/PER_DAY)} days ${Math.floor(((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)%PER_DAY)/3600)} hours ${Math.floor(((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)%3600)/60)} minutes ${Math.floor((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)%60)} seconds.`,
            type: 'error',
            duration: 3000
          });
          return;
        }

        const divide = ((now - (Number(createdTime) + PER_WEEK * Math.floor(Number(round))))%(PER_WEEK))/PER_DAY;
        if(Number(divide) >= 3 && 4 > Number(divide)) {
          try {
            const result = await BattleContract.methods.Stake(val*1000, val).send({ from: this.currentAccount, value: ethers.utils.parseEther("0.002") });
            console.log('result', result);
          } catch(err) {
            console.log(err);
          }
        } else {
          this.$toast.open({
            message: `Not ready time. Please wait ${Math.floor((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)/PER_DAY)} days ${Math.floor(((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)%PER_DAY)/3600)} hours ${Math.floor(((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)%3600)/60)} minutes ${Math.floor((Number(createdTime) + (PER_WEEK * Math.floor(Number(round))) + (PER_DAY * 3) - now)%60)} seconds.`,
            type: 'error',
            duration: 3000
          });
          return;
        }
        console.log('day', divide);


      }
    }
  }
};
</script>
