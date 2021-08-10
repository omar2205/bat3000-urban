<script>
  import { Device } from '@capacitor/device'
  import { onMount } from 'svelte'
  let bat
  let isCharging
  const getBatteryInfo = async () => {
    let b = await Device.getBatteryInfo()
    bat = parseInt((b.batteryLevel * 100)) + '%'
    // console.log(b, isCharging)
    isCharging = b.isCharging
  }
  onMount(() => {
    setInterval(() => {
      getBatteryInfo()
    }, 1000)
  })
</script>

<h1>{bat ? bat : ''}<h2>{isCharging === true ? '⚡' : ''}</h2></h1>

<style lang="scss">
h1 {
  font-size: 20vw;
  font-weight: 100;
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  height: calc(100vh - 65px);
  margin: 0;
  padding: 0;
  h2 {
    font-size: 2rem;
  }
}
</style>
