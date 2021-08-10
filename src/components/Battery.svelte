<script>
  import { Device } from '@capacitor/device'
  import { onMount } from 'svelte'
  let bat
  let isCharging = false
  const getBatteryInfo = async () => {
    let b = await Device.getBatteryInfo()
    bat = (b.batteryLevel * 100) + '%'
    console.log(b)
    if (b.isCharging) isCharging = true
  }
  onMount(() => {
    getBatteryInfo()
  })
</script>

<h1>{bat ? bat : ''}</h1>
<h3>{isCharging ? '⚡' : ''}</h3>

<style lang="scss">
h1 {
  font-size: 4rem;
  font-weight: 100;
  text-align: center;
}
</style>
