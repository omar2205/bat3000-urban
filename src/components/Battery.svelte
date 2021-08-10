<script>
  import { Device } from '@capacitor/device'
  import { onMount } from 'svelte'
  let bat
  let isCharging = false
  const getBatteryInfo = async () => {
    let b = await Device.getBatteryInfo()
    bat = parseInt((b.batteryLevel * 100)) + '%'
    console.log(b)
    if (b.isCharging) isCharging = true
  }
  onMount(() => {
    getBatteryInfo()
  })
</script>

<h1>{bat ? bat : ''}<h2>{isCharging ? '⚡' : ''}</h2></h1>

<style lang="scss">
h1 {
  font-size: 4rem;
  font-weight: 100;
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  h2 {
    font-size: 2rem;
  }
}
</style>
