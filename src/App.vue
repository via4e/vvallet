<template>
  <div>
    <img alt="Vue logo" src="./assets/logo.png" width="90" height="90" />
    <ShowAddress
      :addr="pair.address"
      :pubKey="pair.public"
      :balance="pair.balance"
      @clearPair="clearPair"
    />
    <hr />
    <template v-if="pair?.private && pair.private.length">
      {{ pairMessage }}
      <table>
        <tr>
          <td>mnemonic:</td>
          <td class="normal mnemonic">{{ pair.mnemonic }}</td>
        </tr>
        <tr>
          <td>private:</td>
          <td class="normal">{{ pair.private }}</td>
        </tr>
        <tr>
          <td>public:</td>
          <td class="normal">{{ pair.public }}</td>
        </tr>
        <tr>
          <td>address:</td>
          <td class="normal">{{ pair.address }}</td>
        </tr>
      </table>
    </template>
    <hr />
    <button @click="createPair()">Create</button>
    <button @click="restoreDialog()">Restore</button>

    <Restore
      v-if="showRestore"
      @restoreKey="restorePublicKey"
      @cancelRestore="restoreCancel"
    />
  </div>
</template>

<script>
import ShowAddress from "./components/ShowAddress.vue";
import Restore from "./components/Restore.vue";
import btc from "bitcore-lib";
import Mnemonic from "bitcore-mnemonic";

const DERIVATION = 'm/44\'/0\'/0\'/0/0'
window.btc = btc;
window.Mnemonic = Mnemonic;

export default {
  name: "App",
  components: {
    ShowAddress,
    Restore,
  },
  data() {
    return {
      pair: {},
      inputPrivate: "",
      showRestore: false,
      pairMessage: "Pair created..."
    };
  },
  methods: {
    createPair() {
      //Generate new mnemonic
      let code = new Mnemonic(Mnemonic.Words.ENGLISH);
      let mnemonic = code.toString();

      let xpriv = code.toHDPrivateKey();
      let derived = xpriv.derive(DERIVATION);
      let privateKey = derived.privateKey;
      let pk = new btc.PrivateKey( privateKey.toString() );

      let publicKey = new btc.PublicKey(pk);
      let address = new btc.Address(publicKey);

      console.log('from code', pk, address)

      this.pair = {
        private: pk.toWIF(),
        public: publicKey.toString(),
        address: address.toString(),
        mnemonic: mnemonic,
        balance: "0",
      };

      console.log("pair:", this.pair);
    },
    restoreDialog() {
      console.log(this.privateKey, this.publicKey, this.address);
      this.showRestore = true;
    },
    restorePublicKey(str) {
      console.log("restore from mnemonic:", str);

      let code = new Mnemonic(str);
      let mnemonic = code.toString();

      let xpriv = code.toHDPrivateKey();
      let derived = xpriv.derive(DERIVATION);
      let privateKey = derived.privateKey;
      let pk = new btc.PrivateKey( privateKey.toString() );

      let publicKey = new btc.PublicKey(pk);
      let address = new btc.Address(publicKey);

      this.pair = {
        private: pk.toWIF(),
        public: publicKey.toString(),
        address: address.toString(),
        mnemonic: mnemonic,
        balance: "0",
      };

      this.showRestore = false;
    },
    restoreCancel() {
      console.log("restore");
      this.showRestore = false;
    },
    clearPair() {
      this.pair = {}
    },
  },
};
</script>
