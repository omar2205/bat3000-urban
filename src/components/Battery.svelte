<script>
  import { Device } from '@capacitor/device'
  import { App } from '@capacitor/app'
  import { Http as http } from '@capacitor-community/http'
  import { BackgroundMode } from '@ionic-native/background-mode'
  import { onMount } from 'svelte'

  // const BackgroundMode = cordova.plugins.backgroundMode

  const BCAKEND_URL = 'http://192.168.1.99:9000/c3xu87/phone/battery'
  const CRITICAL_LVL = 30
  let sent_alert = false
  
  let bat
  let isCharging
  let state_isActive = true

  const getBatteryInfo = async () => {
    let b = await Device.getBatteryInfo()
    bat = parseInt((b.batteryLevel * 100)) + '%'
    // console.log(b, isCharging)
    isCharging = b.isCharging
  }

  onMount(() => {
    setInterval(() => {
      getBatteryInfo()
    }, 2000)

    App.addListener('appStateChange', async ({isActive}) => {
      if (isActive) {
        state_isActive = isActive
        return
      }
      // background work
      BackgroundMode.enable(true)
      // check battery every 2s
      // setInterval(() => {
      //   if(bat >= )
      // }, 2000)
      $: bat, checkBat()
    })
  })

  const checkBat = async () => {
    // if we already sent the request
    if(sent_alert === true) return
    if(parseInt(bat) >= CRITICAL_LVL) {
      const res = await http.post({
        url: BCAKEND_URL,
        headers: {'Content-Type': 'application/json'},
        data: {percentage: parseInt(bat), plugged: (isCharging ? 'PLUGGED' : 'UNPLUGGED')}
      })
      // debug, send response
      http.post({
        url: 'http://192.168.1.99:9000/h',
        data: JSON.stringify(res, null, 2)
      })
      // we reached our desired battery level
      // now we turn off background mode
      sent_alert = true
      BackgroundMode.enable(false)
    }
  }
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
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
  position: absolute;
  top: 0;
  left: 0;
  h2 {
    font-size: 2rem;
  }
}
</style>
