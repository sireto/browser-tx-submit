<template>
  <div>
  <b-form-textarea
    id="textarea"
    v-model="text"
    placeholder="Enter something..."
    rows="25"
    max-rows="6"
  ></b-form-textarea>
  <b-button @click="submitTx">Sign me </b-button>
  </div>
</template>

<script>
import {Transaction, TransactionWitnessSet, Vkeywitnesses} from "@emurgo/cardano-serialization-lib-browser";

export default {
  name: 'IndexPage',
  data:()=>{
     return {
       text:'',

     }
  } ,
  created:async ()=>{
    let c = await window.cardano.ccvault.enable()  ;
    window.adaProvider=c
  },
  methods:{
    async submitTx(){

      let transaction;
      try {
        // const S = await import('@emurgo/cardano-serialization-lib-browser')

        transaction = Transaction.from_bytes(Uint8Array.from(Buffer.from(this.text, 'hex')))
        // @ts-ignore
      }catch(e){
        alert(e.message);
        console.log(e)
        return;
      }
        const witnesesRaw = await window.adaProvider.signTx(
          Buffer.from(transaction.to_bytes()).toString('hex'),
          true
        )
      try {
        const newWitnesses = TransactionWitnessSet.from_bytes(Buffer.from(witnesesRaw, "hex"))
        console.log("newWitnessSetList is :", newWitnesses.vkeys()?.len())


        const newWitnessSet = TransactionWitnessSet.new();
        if (transaction.witness_set().plutus_data())
          newWitnessSet.set_plutus_data(transaction.witness_set().plutus_data());
        if (transaction.witness_set().plutus_scripts())
          newWitnessSet.set_plutus_scripts(transaction.witness_set().plutus_scripts())
        if (transaction.witness_set().redeemers())
          newWitnessSet.set_redeemers(transaction.witness_set().redeemers())

        // add the new witness.
        if (transaction.witness_set().vkeys()) {
          const newVkeySet=Vkeywitnesses.new()

          for(let i=0;i<transaction.witness_set().vkeys().len();i++){
            newVkeySet.add(transaction.witness_set().vkeys().get(i))

          }
          for(let i=0;i<newWitnesses.vkeys().len();i++){
            newVkeySet.add(newWitnesses.vkeys().get(i))
          }
          newWitnessSet.set_vkeys(newVkeySet)

        } else {
          newWitnessSet.set_vkeys(newWitnesses.vkeys())
        }
        transaction = Transaction.new(transaction.body(), newWitnessSet, transaction.auxiliary_data());
        const signedTxString = Buffer.from(transaction.to_bytes()).toString('hex')
        // @ts-ignore
        await window.adaProvider.submitTx(signedTxString);
      }catch(e){
        alert(e.message);
        console.log(e);
      }
    }
  }
}
</script>
