<template>
    <div id="verify">
        <div id="top">
          <img id="logo" src="../assets/logo1.png">
          <div id="dashtext">
              Dashboard
          </div>
        </div>
        <div id="left">
            <li></li>
            <li></li>
        </div>
        <div style="margin-left: 30%;">
          <br>
          <br>
          <h1 style="color: cornflowerblue; margin-left: 15%">iContinu Facial Verification</h1>
          <br>
          <video id="video" autoplay></video>
          <canvas id="canvas" width="640" height="480" style="display: none;"></canvas>
          <br><br>
          <div style="margin-left: 20%;">
          <button id="submitbtn" @click="takePhoto">Submit</button>
          </div>
          <br>
      </div>
    </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'verify',
  data () {
    return {
      headImgSrc: ''
    }
  },
  mounted () {
    this.getCompetence()
  },
  methods: {
    getCompetence () {
      // Older browsers might not implement mediaDevices at all, so we set an empty object first
      if (navigator.mediaDevices === undefined) {
        navigator.mediaDevices = {}
      }

      // Some browsers partially implement mediaDevices. We can't just assign an object
      // with getUserMedia as it would overwrite existing properties.
      // Here, we will just add the getUserMedia property if it's missing.
      if (navigator.mediaDevices.getUserMedia === undefined) {
        navigator.mediaDevices.getUserMedia = function (constraints) {
          // First get ahold of the legacy getUserMedia, if present
          var getUserMedia = navigator.webkitGetUserMedia || navigator.mozGetUserMedia

          // Some browsers just don't implement it - return a rejected promise with an error
          // to keep a consistent interface
          if (!getUserMedia) {
            return Promise.reject(new Error('getUserMedia is not implemented in this browser'))
          }

          // Otherwise, wrap the call to the old navigator.getUserMedia with a Promise
          return new Promise(function (resolve, reject) {
            getUserMedia.call(navigator, constraints, resolve, reject)
          })
        }
      }

      navigator.mediaDevices.getUserMedia({ audio: false, video: true })
        .then(function (stream) {
          var video = document.querySelector('video')
          // Older browsers may not have srcObject
          if ('srcObject' in video) {
            video.srcObject = stream
          } else {
            // Avoid using this in new browsers, as it is going away.
            video.src = window.URL.createObjectURL(stream)
          }
          video.style.MozTransform = 'scaleX(-1)'
          video.style.webkitTransform = 'scaleX(-1)'
          video.style.OTransform = 'scaleX(-1)'
          video.style.msFilter = 'FlipH'
          video.style.filter = 'FlipH'
          video.style.transform = 'scaleX(-1)'
          video.onloadedmetadata = function (e) {
            video.play()
          }
        })
        .catch(function (err) {
          console.log(err.name + ': ' + err.message)
        })
    },
    takePhoto () {
      const demoFaceId = 'cd44a05d-3c64-41bf-8c44-1560c752193d'
      let canvas = document.getElementById('canvas')
      let context = canvas.getContext('2d')
      let video = document.getElementById('video')
      // draw image on canvas
      context.drawImage(video, 0, 0, 480, 320)
      video.pause()
      var image = canvas.toDataURL('image/png')
      console.log('click take photo')
      let file = this.dataURLtoFile(image, 'taken')
      this.axios.post(
        'https://australiaeast.api.cognitive.microsoft.com/face/v1.0/detect?returnFaceId=true',
        file,
        {
          headers: {
            'content-type': 'application/octet-stream',
            'Ocp-Apim-Subscription-Key': 'ab34ba5577ba40c5bf1df4851fb2ded2'
          }
        }
      ).then(response => {
        this.axios.post(
          'http://0.0.0.0:8000/iContinu/face/',
          {
            faceId_1: demoFaceId,
            faceId_2: response.data[0].faceId
          })
          .then(res => {
            console.log(res)
            console.log(res.data.confidence)
            alert('Confidence is: ' + res.data.confidence.confidence)
            video.play()
          })
      })
    },
    dataURLtoFile (dataURL, filename) {
      var arr = dataURL.split(',')
      var mime = arr[0].match(/:(.*?);/)[1]
      var bstr = atob(arr[1])
      var n = bstr.length
      var u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      return new File([u8arr], filename, {type: mime})
    }
  }
}
</script>

<style scoped>
#verify{
  min-width: 1000px;
  min-height: 1000px;
  padding: 0;
  margin: 0;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  position: absolute;
}

#content{
  background: rgba(255,255,255,0.5);
  width: 100%;
  height: 400px;
  font-size: 26px;
  padding-left: 15px ;
  padding-right: 15px;
  padding-top: 15px;
  margin: 0 auto;
}

#top{
  width: 100%;
  height: 200px;
  min-width: 800px;
  padding: 0;
  margin: 0;
  left: 0;
  top: 0;
  background: url("../assets/page2Bg1.png") no-repeat;
}

#logo{
  padding-top: 10px;
  padding-left: 10px;
  width: 200px;
  position: absolute;
}
#dashtext{
  margin-left: 10%;
  padding-top: 100px;
  font-weight: bold;
  color: orangered;
  font-size: 50px;
  text-align: left;
}
#left{
  width: 170px;
  height: 100%;
  background: antiquewhite;
  position: absolute;
  overflow: hidden;
  border-right: 1px grey solid;
}
#right{
  background: antiquewhite;
  padding-top: 33px;
  padding-left: 43px;
  margin-left: 171px;
  width: 100%;
  height: 100%;
  position: absolute;
  float: left;
}
#photo{
  width: 500px;
  height: 360px;
  border: #232323 1px solid;
}
#submitbtn{
  background-color: rgba(255,192,203,0.4);
  border-radius: 15px;
  height: 30px;
  width: 200px;
  outline: none;
  margin-top: 20px;
}
</style>
