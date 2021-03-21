<template>
<li class="col-span-1 flex flex-col text-center bg-white rounded-l border divide-y divide-gray-200">
    <div class="flex-1 flex flex-col p-4">
        <img
            class="w-32 h-32 flex-shrink-0 mx-auto bg-white"
            :src="data.image"
            alt="">
            <h3 class="mt-6 text-gray-900 text-sm font-medium">#{{data.id}} - {{data.name}}</h3>
            <dl class="mt-1 flex-grow flex flex-col justify-between">
                <dd class="text-gray-500 text-sm">{{data.description}} </dd>
                <dd v-if="connected === false" class="mt-3">
                    <span class="px-2 py-1 text-gray-400 text-xs font-medium bg-gray-100 rounded-full">Initializing...</span>
                </dd>
                <dd v-if="connected === true" class="mt-3">
                    <span v-if="state.loading === true" class="px-2 py-1 text-gray-800 text-xs font-medium bg-gray-100 rounded-full">Loading...</span>
                    <span v-if="state.owned === false && state.loading === false && state.pendingTx === false" class="px-2 py-1 text-green-800 text-xs font-medium bg-green-100 rounded-full">Available for {{state.ethPrice}} ETH</span>
                    <span v-if="state.owned === true && state.loading === false && state.pendingTx === false" class="px-2 py-1 text-red-800  text-xs font-medium bg-red-100 rounded-full  truncate overflow-hidden w-40 align-middle inline-block">Owned by {{state.owner}}</span>
                    <span v-if="state.pendingTx === true" class="px-2 py-1 text-gray-800 text-xs font-medium bg-gray-100 rounded-full">Processing...</span>
                </dd>
            </dl>
        </div>
        <div>
            <div v-if="connected === true" class="-mt-px flex divide-x divide-gray-200">
                <div class=" w-0 flex-1 flex">
                    <a v-if="state.owned === false && state.loading === false" v-on:click="buy()"
                        class="cursor-pointer relative w-0 flex-1 inline-flex items-center justify-center py-4 text-sm text-gray-700 font-medium border border-transparent rounded-br-lg hover:text-gray-500">
                        <span class="" >Buy</span>
                    </a>
                    <div v-if="state.owned === true"
                        class=" relative w-0 flex-1 inline-flex items-center justify-center py-4 text-sm text-gray-400 font-medium border border-transparent rounded-br-lg">
                        <span class="truncate px-8" >Sold</span>
                    </div>
                </div>
            </div>
        </div>
    </li>
</template>
<script>
export default {
  name: 'Token',
  props: {
    data: Object,
    sold: Boolean,
    connected: Boolean,
    state: Object
  },
  created() {
  this.$emit('created');
},
  methods: {
    buy(){
      this.$emit('buy');
    },
  },
  watch: {
    connected: function(newVal, oldVal){
      this.$emit('check');
    }
  }
}
</script>
