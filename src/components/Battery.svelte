<script>
  import { App } from '@capacitor/app'
  import { Device } from '@capacitor/device'
  import { Toast } from '@capacitor/toast'
  import { Http as http } from '@capacitor-community/http'
  import { onMount } from 'svelte'

  let debug_mode = false

  let BackgroundMode
  if (typeof cordova !== 'undefined') {
    BackgroundMode = cordova.plugins.backgroundMode
  }

  const BACKEND_URL = 'http://192.168.1.99:9000'
  const BACKEND_URL_BAT = 'http://192.168.1.99:9000/c3xu87/phone/battery'
  const CRITICAL_LVL = 95
  let sent_alert = false

  let bat
  let isCharging
  let state_isActive = true

  // if var bat changes, invoke checkBat()
  $: bat, checkBat()

  const debug_send = async (msg) => {
    http.post({
      url: BACKEND_URL+'/h',
      headers: { 'Content-Type': 'text/html; charset=utf-8' },
      data: `---> ${msg}`
    })
  }

  const getBatteryInfo = async () => {
    let b = await Device.getBatteryInfo()
    bat = Math.round(parseFloat(b.batteryLevel) * 100) + '%'
    // console.log(b, isCharging)
    isCharging = b.isCharging
  }

  onMount(() => {
    getBatteryInfo()
    if(debug_mode) debug_send('mounted')
    if(debug_mode) debug_send(`current status - ${bat} ${isCharging ? 'Charging ⚡' : 'Not-Charging'}`)

    setInterval(getBatteryInfo, 2000)    

    App.addListener('appStateChange', ({ isActive }) => {
      state_isActive = isActive
      if(debug_mode) debug_send(`state-change ${isActive}`)
    })
  })

  const checkBat = async () => {
    if (sent_alert) return
    if (isCharging === false) return
    if (typeof BackgroundMode !== 'undefined') BackgroundMode.enable(true)
    if(parseInt(bat) >= CRITICAL_LVL) {
      const res = await http.post({
        url: BACKEND_URL,
        headers: { 'Content-Type': 'application/json' },
        data: { percentage: parseInt(bat), plugged: (isCharging ? 'PLUGGED' : 'UNPLUGGED') }
      })
      await Toast.show({
        text: 'Battery reached wanted charge level. sent request.',
        duration: 'long'
      })
      sent_alert = true
      if (typeof BackgroundMode !== 'undefined') BackgroundMode.enable(false)
    }
  }

</script>

<h1>{bat ? bat : ''}<h2>{isCharging === true ? '⚡' : ''}</h2>
</h1>

<style lang="scss">
  h1 {
    font-size: 25vw;
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