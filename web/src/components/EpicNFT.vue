<template>
  <div class="container">
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
      <button
        v-else
        class="cta-button connect-wallet-button"
        @click="askContract"
      >
        Mint NFT
      </button>
      <p v-if="currentAccount !== null" class="sub-text">
        Account: {{ currentAccount }}
      </p>
      <p v-if="mintStatus !== null" class="sub-text">
        {{ mintStatus }}
      </p>
      <a :href="link" target="_blank" class="sub-text">{{ link }}</a>
    </div>
  </div>
</template>

<script>
import { ethers } from "ethers";
import myEpicNft from "../utils/MyEpicNFT.json";

export default {
  name: "EpicNFT",
  data() {
    return {
      ethereum: null,
      currentAccount: null,
      mintStatus: null,
      link: null,
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
      const CONTRACT_ADDRESS = "0x51307b5070D5cDE9d593cB8b6F5D297A9b91Ab5F";

      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(
            CONTRACT_ADDRESS,
            myEpicNft.abi,
            signer
          );

          connectedContract.on("NewEpicNFTMinted", (from, tokenId) => {
            console.log(from, tokenId.toNumber());
            this.link = `https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`;
          });

          let nftTxn = await connectedContract.makeAnEpicNFT();

          this.mintStatus = "Mining... please wait";
          await nftTxn.wait();

          this.mintStatus = "Mined, see transaction: ";
          // this.link = `https://rinkeby.etherscan.io/tx/${nftTxn.hash}`;
        } else {
          this.mintStatus = "Ethereum object doesn't exist!";
        }
      } catch (error) {
        console.log(error);
      }
    },
  },
  async mounted() {
    const { ethereum } = window;
    this.ethereum = ethereum;
    if (this.etherum === null) {
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
