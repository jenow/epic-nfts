<template>
  <div class="container">
    <loading
      v-model:active="isLoading"
      :can-cancel="false"
      :is-full-page="true"
    />
    <p v-if="currentAccount !== null" class="small-sub-text">
      Account: {{ currentAccount }}
    </p>
    <div class="header-container">
      <p class="header gradient-text">Mint my NFT</p>
      <p class="sub-text">Yes, mint me!</p>
      <p>
        <a
          class="sub-text"
          href="https://testnets.opensea.io/assets/squarenft-nxe1repf3h"
          target="_blank"
          >See collection</a
        >
      </p>
      <button
        v-if="currentAccount === null"
        @click="connectWallet"
        class="cta-button connect-wallet-button"
      >
        Connect to Wallet
      </button>
      <div v-else>
        <button class="cta-button connect-wallet-button" @click="askContract">
          Mint NFT
        </button>
        <p class="sub-text">Only {{ 50 - mintedTime }} available, hurry up!</p>
      </div>
      <p v-if="mintStatus !== null" class="sub-text">
        {{ mintStatus }}
      </p>
      <a :href="link" target="_blank" class="sub-text" v-if="link != null"
        >Minted! Click here to see.</a
      >
      <p v-if="error !== null" class="error">{{ error }}</p>
    </div>
  </div>
</template>

<script>
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/vue-loading.css";

import { ethers } from "ethers";
import myEpicNft from "../utils/MyEpicNFT.json";

const CONTRACT_ADDRESS = "0xda72F64E1816D704E294259cE0B69e4DE337ecC7";

export default {
  name: "EpicNFT",
  components: {
    Loading,
  },
  data() {
    return {
      isLoading: false,
      error: null,
      ethereum: null,
      currentAccount: null,
      mintStatus: null,
      link: null,
      mintedTime: 0,
    };
  },
  methods: {
    async connectWallet() {
      if (!this.ethereum) {
        alert("Get MetaMask!");
        return;
      }
      console.log(this.ethereum);
      const accounts = await this.ethereum.request({
        method: "eth_requestAccounts",
      });

      this.currentAccount = accounts[0];
    },
    async askContract() {
      try {
        if (this.ethereum) {
          const provider = new ethers.providers.Web3Provider(this.ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(
            CONTRACT_ADDRESS,
            myEpicNft.abi,
            signer
          );
          this.mintedTime = parseInt(await connectedContract.getMintedTime());

          this.isLoading = true;
          let nftTxn = await connectedContract.makeAnEpicNFT();

          connectedContract.on("NewEpicNFTMinted", async (from, tokenId) => {
            console.log(from, tokenId.toNumber());
            this.link = `https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`;
            this.mintedTime = parseInt(await connectedContract.getMintedTime());
          });
          console.log("Event setup");

          this.mintStatus = "Mining... please wait";
          await nftTxn.wait();
          this.isLoading = false;

          this.mintStatus = "Mined, see transaction: ";
          // this.link = `https://rinkeby.etherscan.io/tx/${nftTxn.hash}`;
        } else {
          this.mintStatus = "Ethereum object doesn't exist!";
        }
      } catch (e) {
        this.isLoading = false;
        this.error = 'There is not NFT available anymore :(';
      }
    },
  },
  async mounted() {
    const { ethereum } = window;
    this.ethereum = ethereum;
    if (this.etherum === null) {
      alert("Get Metamask!");
      return;
    }
    const accounts = await this.ethereum.request({ method: "eth_accounts" });

    if (accounts.length !== 0) {
      const account = accounts[0];
      console.log("Found an authorized account:", account);
      this.currentAccount = account;
      let chainId = await ethereum.request({ method: "eth_chainId" });
      console.log("Connected to chain " + chainId);

      // String, hex code of the chainId of the Rinkebey test network
      const rinkebyChainId = "0x4";
      if (chainId !== rinkebyChainId) {
        alert("You are not connected to the Rinkeby Test Network!");
      } else {
        const provider = new ethers.providers.Web3Provider(this.ethereum);
        const signer = provider.getSigner();
        const connectedContract = new ethers.Contract(
          CONTRACT_ADDRESS,
          myEpicNft.abi,
          signer
        );
        connectedContract.on("NewEpicNFTMinted", async (from, tokenId) => {
          console.log(from, tokenId.toNumber());
          this.link = `https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`;
          this.mintedTime = parseInt(await connectedContract.getMintedTime());
        });
        console.log("Event setup");
        this.mintedTime = parseInt(await connectedContract.getMintedTime());
      }
    } else {
      console.log("No authorized account found");
    }
  },
};
</script>

<style scoped>
.gradient-text {
  background: -webkit-linear-gradient(left, #ffa31a 0%, #1b1b1b 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.container {
  height: 100%;
  background-color: black;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.header-container {
  padding-top: 30px;
}

.header {
  margin: 0;
  font-size: 50px;
  font-weight: bold;
}

.sub-text {
  font-size: 25px;
  color: white;
}

.small-sub-text {
  font-size: 12px;
  color: white;
}

.error {
  color: red;
}
.cta-button {
  height: 45px;
  border: 0;
  width: auto;
  padding-left: 40px;
  padding-right: 40px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  color: white;
}

.connect-wallet-button {
  background: -webkit-linear-gradient(left, #ffa31a 0%, #1b1b1b 100%);
  background-size: 200% 200%;
  animation: gradient-animation 4s ease infinite;
}

.mint-button {
  background: -webkit-linear-gradient(left, #ffa31a 0%, #1b1b1b 100%);
  background-size: 200% 200%;
  animation: gradient-animation 4s ease infinite;
  margin-right: 15px;
}

.opensea-button {
  background-color: rgb(32, 129, 226);
}

.mint-count {
  color: white;
  font-size: 18px;
  font-weight: bold;
}

/* KeyFrames */

@-webkit-keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

@-moz-keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

@keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
