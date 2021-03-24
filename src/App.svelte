<script>
  export let name;
  import { onMount } from "svelte";

  let startPos;
  let api = "http://47.90.21.235:7001/api/weather/daily";
  const SKYCON = {
    CLEAR_DAY: "晴（白天）",
    CLEAR_NIGHT: "晴（夜间）",
    PARTLY_CLOUDY_DAY: "多云（白天）",
    PARTLY_CLOUDY_NIGHT: "多云（夜间）",
    CLOUDY: "阴",
    LIGHT_HAZE: "轻度雾霾",
    MODERATE_HAZE: "中度雾霾",
    HEAVY_HAZE: "重度雾霾",
    LIGHT_RAIN: "小雨",
    MODERATE_RAIN: "中雨",
    HEAVY_RAIN: "大雨",
    STORM_RAIN: "暴雨",
    FOG: "雾",
    LIGHT_SNOW: "小雪",
    MODERATE_SNOW: "中雪",
    HEAVY_SNOW: "大雪",
    STORM_SNOW: "暴雪",
    DUST: "浮尘",
    SAND: "沙尘",
    WIND: "大风",
  };
  const getAQIDesc = (value) => {
    if (value < 50) {
      return "优";
    } else if (value < 100) {
      return "良";
    } else if (value < 150) {
      return "轻度污染";
    } else if (value < 200) {
      return "中度污染";
    }
    return "重度污染";
  };
  const shortReport = (res) => {
    const {
      data: { daily },
    } = res;
    const { temperature, air_quality, skycon_08h_20h } = daily;
    const { aqi } = air_quality;
    const { date, min: temperatureMin, max: temperatureMax } = temperature[0];
    const {
      avg: { chn },
    } = aqi[0];
    const { value: skyconValue } = skycon_08h_20h[0];
    return `${date} | ${SKYCON[skyconValue]} | ${Math.round(
      temperatureMin
    )} ~ ${Math.round(temperatureMax)}℃ | AQI: ${Math.round(chn)} ${getAQIDesc(
      chn
    )}`;
  };
  let geoSuccess = async (position) => {
    startPos = position;
    let lat = startPos.coords.latitude;
    let lng = startPos.coords.longitude;
    document.getElementById("lat").innerHTML = lat;
    document.getElementById("lng").innerHTML = lng;
    // request weather
    const res = await fetch(`${api}?lat=${lat}&lng=${lng}`);
    const list = await res.json();
    console.log("list", list);
    const msg = shortReport(list);
    document.getElementById("msg").innerHTML = msg;
  };
  let geoError = function (error) {
    console.log("Error occurred. Error code: " + error.code);
    // error.code can be:
    //   0: unknown error
    //   1: permission denied
    //   2: position unavailable (error response from location provider)
    //   3: timed out
  };
  onMount(async () => {
    navigator.geolocation.getCurrentPosition(geoSuccess, geoError);
  });
</script>

<main>
  <h1>Hello {name}!</h1>
  <div id="msg" />
  <div id="lat" />
  <div id="lng" />
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
