<script>
export let ms
  
  const getMinutes = () => {
    return Math.floor(ms / 1000  / 60).toString().padStart(2, "0")
  }

  const getSeconds = () => {
    return Math.floor(ms / 1000 % 60).toString().padStart(2, "0")
  }

  const getMs = () => {
    return Math.floor(ms / 100 % 10)
  }
</script>



<div class="timer">
  <span class="timer-block">{getMinutes(ms)}</span>
  <span>:</span>
  <span class="timer-block">{getSeconds(ms)}</span>
  <span class="timer-ms">.{getMs(ms)}<span/>
</div>


<style>
.timer {
  font-size: 26px;
  font-weight: 100;
  display: flex;
  align-items: baseline
}
.timer-block {
  width: 26px;
}
.timer-ms {
  margin-left: 2px;
  font-size: 14px;
  width: 7px;
}
</style>