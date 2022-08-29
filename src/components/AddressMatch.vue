<template>
<div></div>
</template>

<script>
export default {
  name: "AddressMatch",
  mounted() {
    const addressMatchService = new window.ol.supermap.AddressMatchService('https://iserver.supermap.io/iserver/services/addressmatch-Address/restjsr/v1/address')
    const param = new window.SuperMap.GeoCodingParameter({
      address: '超图软件'
    })
    console.log('------------------------------------地址匹配------------------------------------')
    addressMatchService.code(param, e => {
      console.log(e)
    })
  },
  data() {
    return {}
  },
}
</script>

<style scoped>

</style>