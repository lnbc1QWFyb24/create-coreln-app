<script>
  import { onMount, onDestroy } from 'svelte'

  let canvas, ctx, image, centerX, centerY

  onMount(() => {
    setCanvasSize()
    ctx = canvas.getContext('2d')
    drawTriangle()

    window.addEventListener('resize', handleResize)
  })

  onDestroy(() => {
    window.removeEventListener('resize', handleResize)
  })

  function setCanvasSize() {
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight
  }

  function handleResize() {
    setCanvasSize()
    drawTriangle()
  }

  function drawRainbow(numColors, startColor, y1, y2) {
    // Draw the rainbow colors coming out of the right side
    let rainbowGradient = ctx.createLinearGradient(
      centerX,
      centerY - 40,
      canvas.width - 25,
      canvas.height - 150
    )
    let colorStop = 0
    function getColorStop() {
      return (colorStop += 1 / numColors)
    }
    var rainbow = {
      red: '#fb2d2d',
      orange: '#fb9600',
      yellow: '#f2f600',
      green: '#0bff32',
      blue: '#00d9f6',
      indigo: '#1c17ff',
      violet: '#b600f6'
    }
    const rainbowArray = Object.keys(rainbow)
    let startIndex = rainbowArray.indexOf(startColor)
    // console.log('startIndex: ' + startIndex)
    // console.log(rainbowArray)

    for (let i = 0; i < numColors; i++) {
      let index = startIndex++ % rainbowArray.length
      rainbowGradient.addColorStop(getColorStop(), rainbow[rainbowArray[index]])
    }

    // Draw the rainbow colors coming out of the right side
    ctx.strokeStyle = rainbowGradient
    ctx.beginPath()
    ctx.moveTo(centerX, centerY - y1)
    //                           bottom right coordinates on right side of beam
    ctx.lineTo(canvas.width, canvas.height - y2)
    ctx.stroke()
  }

  function drawTriangle() {
    // Clear the canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height)

    // Set the background color to black
    ctx.fillStyle = '#000'
    ctx.fillRect(0, 0, canvas.width, canvas.height)

    // Draw the triangle with white stroke
    ctx.strokeStyle = '#fff'
    ctx.lineWidth = 7
    centerX = canvas.width / 2
    centerY = canvas.height / 2

    // Draw the light beam coming from the left side
    ctx.fillStyle = 'white'
    ctx.beginPath()
    // starting x coord on the L side,  starting y coordinate on left side
    ctx.moveTo(0, 430)
    ctx.lineTo(canvas.width / 2, canvas.height / 2 - 40)
    ctx.stroke()

    var rainCoord = [
      [{ y1: 40, y2: 180 }],
      [
        { y1: 40, y2: 300 },
        { y1: 30, y2: 130 }
      ]
    ]

    // one rainbow is 40, 180
    //drawRainbow(7, "red", rainCoord[0][0].y1, rainCoord[0][0].y2);

    // two rainbows
    // drawRainbow(3, "red", 44, 300);
    // drawRainbow(3, "blue", 40, 180);

    // three rainbows
    // drawRainbow(2, "red", 50, 400);
    // drawRainbow(2, "yellow", 44, 300);
    // drawRainbow(2, "blue", 40, 180);

    // four rainbows
    drawRainbow(2, 'red', 60, 400)
    drawRainbow(2, 'yellow', 53, 325)
    drawRainbow(2, 'blue', 46.5, 250)
    drawRainbow(2, 'violet', 40, 180)

    // Draw the triangle with a black center
    ctx.beginPath()
    ctx.moveTo(centerX - 90, centerY + 50)
    ctx.lineTo(centerX, centerY - 100)
    ctx.lineTo(centerX + 90, centerY + 50)
    ctx.closePath()
    ctx.fillStyle = '#000'
    ctx.fill()
    ctx.strokeStyle = '#fff' // set stroke color to white
    ctx.stroke()
  }

  function hideScrollbars() {
    const html = document.querySelector('html')
    html.style.overflow = 'hidden'
  }
  $: hideScrollbars()
</script>

<canvas bind:this={canvas} style="" />
