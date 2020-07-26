<script>
  import { onMount } from "svelte";
  import { navigate } from "svelte-routing";
  import Thankyou from "./Thankyou.svelte";

  import { onDestroy } from "svelte";

  var videoRecorder = null;

  var checkPhase = true;
  var recordingPhase = false;
  var uploadPhase = false;
  var deviceReady = false;

  var isRecording = false;
  var isPlayingback = false;
  var uploading = false;
  var hasUploaded = false;

  var lastRecording = null;

  onMount(function() {
    console.log("onload testing");

    window.addEventListener("resize", onResizeWindow);

    onDestroy(() => console.log("onDestroy"));

    // apply some workarounds for opera browser
    applyVideoWorkaround();

    // recording
    var videoRecorderOptions = {
      controls: false,
      fluid: false,
      fill: true,
      plugins: {
        record: {
          audio: true,
          video: true,
          maxLength: 120,
          debug: true
        }
      }
    };

    videoRecorder = videojs("videoRecorder", videoRecorderOptions, function() {
      // print version information at startup
      var msg =
        "2. Using video.js " +
        videojs.VERSION +
        " with videojs-record " +
        videojs.getPluginVersion("record") +
        " and recordrtc " +
        RecordRTC.version;
      videojs.log(msg);
      videoRecorder.record().getDevice();
      console.log(this.videoWidth());
      console.log(this.videoHeight());
    });

    videoRecorder.controlBar.progressControl.disable();

    // error handling
    videoRecorder.on("deviceError", function() {
      console.log("device error:", videoRecorder.deviceErrorCode);
    });

    videoRecorder.on("error", function(element, error) {
      console.error(error);
    });

    // user clicked the record button and started recording
    videoRecorder.on("startRecord", function() {
      console.log("started recording!");
    });

    videoRecorder.on("deviceReady", function(){
      let videoWidth = this.videoWidth();
      let videoHeight = this.videoHeight();

      let screenWidth = screen.width;
      let screenHeight = screen.height;

      let videoRatio = videoWidth / videoHeight;
      let screenRatio = screenWidth / screenHeight;

      if(videoRatio < screenRatio){
        let factor = screenHeight / videoHeight;
        
        let fillVideoWidth = factor * videoWidth;
        let fillVideoHeight = factor * videoHeight;

        factor = screenWidth / fillVideoWidth;
        let finalVideoWidth = screenWidth;
        let finalVideoHeight = fillVideoHeight * factor;

        let negTopOffset = (screenHeight - finalVideoHeight) / 2;

        let targetDiv = document.getElementById('videoDiv');
        let tempStr = "width: " + finalVideoWidth + "px; height: " + finalVideoHeight + "px; top: " + negTopOffset + "px;";
        console.log(tempStr);
        targetDiv.style.cssText = tempStr;

      }else{
        let factor = screenWidth / videoWidth;
        
        let fillVideoWidth = factor * videoWidth;
        let fillVideoHeight = factor * videoHeight;

        factor = screenHeight / fillVideoHeight;
        let finalVideoWidth = fillVideoWidth * factor;
        let finalVideoHeight = screenHeight;

        let negLeftOffset = (screenWidth - finalVideoWidth) / 2;

        let targetDiv = document.getElementById('videoDiv');
        let tempStr = "width: " + finalVideoWidth + "px; height: " + finalVideoHeight + "px; left: " + negLeftOffset + "px;";
        console.log(tempStr);
        targetDiv.style.cssText = tempStr;
      }
      if(checkPhase == false){
        recordingPhase = true;
        // class="videoDiv"
        let videoDiv = document.getElementById('videoDiv');
        videoDiv.classList.add('videoDiv');
      }
      deviceReady = true;

    });

    // user completed recording and stream is available
    videoRecorder.on("finishRecord", function() {
      console.log("finished recording: ", videoRecorder.recordedData);
      lastRecording = videoRecorder.recordedData;

      var recorder = document.getElementById('videoRecorder');
      var player = document.getElementById('videoPlayer');
      console.log(player);
      recorder.style.display = "none";
      player.style.display = "block";

      player.src = window.URL.createObjectURL(lastRecording);
    });

  });

  function startRecording() {
    if (!videoRecorder.record().isRecording()) {
      videoRecorder.record().start();
      isRecording = true;
    }
  }

  function stopRecording() {
    if (videoRecorder.record().isRecording()) {
      videoRecorder.record().stop();
      isRecording = false;
      recordingPhase = false;
      uploadPhase = true;
    }
  }

  function playbackRecording(){
    var player = videojs('videoPlayer');
    player.on('ended', function(){
      isPlayingback = false;
    });
    player.play();
    isPlayingback = true;
  }

  function resetRecording(){
    recordingPhase = true;

    uploadPhase = false;
    isRecording = false;
    isPlayingback = false;
    uploading = false;
    hasUploaded = false;

    lastRecording = null;

    var player = videojs('videoPlayer');
    player.dispose();

    let reMake = document.createElement('video');
    reMake.id = 'videoPlayer';
    reMake.classList.add('video-js');

    reMake.style.cssText = 'width: 100%; height: 100%; position: relative; top: 0%; left: 0%;';
    
    reMake.style.display = 'none';
    document.getElementById('videoDiv').appendChild(reMake);

    var recorder = document.getElementById('videoRecorder');
    recorder.style.display = 'block';
  }

  function readyForRecording(){
    checkPhase = false;
    if(deviceReady){
      recordingPhase = true;
    }
  }

  function handleClick() {
    // the blob object contains the recorded data that
    // can be downloaded by the user, stored on server etc.
    uploading = true;
    var albumBucketName = "feedbackprototype";
    var bucketRegion = "us-east-2";
    var IdentityPoolId = "us-east-2:2cb90056-0f69-4f12-a62d-f42ed708822a";

    AWS.config.update({
      region: bucketRegion,
      credentials: new AWS.CognitoIdentityCredentials({
        IdentityPoolId: IdentityPoolId
      })
    });

    var s3 = new AWS.S3({
      apiVersion: "2006-03-01",
      params: { Bucket: albumBucketName }
    });

    var uploadFile = videoRecorder.recordedData;
    var timestamp = new Date().getTime().toString();
    var uploadFile = new File([uploadFile], timestamp);
    var albumPhotosKey = "videostuff" + "/";
    var photoKey = albumPhotosKey + timestamp + ".mp4";

    // Use S3 ManagedUpload class as it supports multipart uploads
    var upload = new AWS.S3.ManagedUpload({
      params: {
        Bucket: albumBucketName,
        Key: photoKey,
        Body: uploadFile
      }
    });

    var promise = upload.on("httpUploadProgress", function(evt) {
      //  document.querySelector("#progressbar>div").style.width = parseInt((evt.loaded * 100) / evt.total) + "%";
      if(evt.loaded < 0){
        updateProgress(0);
      }else if(evt.loaded <= evt.total){
        let percentage = evt.loaded / evt.total;
        updateProgress(percentage);
      }else{
        updateProgress(1);
      }
    }).promise();

    promise.then(
      function(data) {
        videoRecorder.dispose();
        navigate("/thankyou", { replace: true });
      },
      function(err) {
        return alert("There was an error uploading your photo: ", err.message);
      }
    );

    hasUploaded = true;
  }

  function onResizeWindow(){
    let videoWidth = videoRecorder.videoWidth();
    let videoHeight = videoRecorder.videoHeight();

    let screenWidth = screen.width;
    let screenHeight = screen.height;

    let videoRatio = videoWidth / videoHeight;
    let screenRatio = screenWidth / screenHeight;

    if(videoRatio < screenRatio){
      let factor = screenHeight / videoHeight;
      
      let fillVideoWidth = factor * videoWidth;
      let fillVideoHeight = factor * videoHeight;

      factor = screenWidth / fillVideoWidth;
      let finalVideoWidth = screenWidth;
      let finalVideoHeight = fillVideoHeight * factor;

      let negTopOffset = (screenHeight - finalVideoHeight) / 2;

      let targetDiv = document.getElementById('videoDiv');
      let tempStr = "width: " + finalVideoWidth + "px; height: " + finalVideoHeight + "px; top: " + negTopOffset + "px;";
      console.log(tempStr);
      targetDiv.style.cssText = tempStr;

    }else{
      let factor = screenWidth / videoWidth;
      
      let fillVideoWidth = factor * videoWidth;
      let fillVideoHeight = factor * videoHeight;

      factor = screenHeight / fillVideoHeight;
      let finalVideoWidth = fillVideoWidth * factor;
      let finalVideoHeight = screenHeight;

      let negLeftOffset = (screenWidth - finalVideoWidth) / 2;

      let targetDiv = document.getElementById('videoDiv');
      let tempStr = "width: " + finalVideoWidth + "px; height: " + finalVideoHeight + "px; left: " + negLeftOffset + "px;";
      console.log(tempStr);
      targetDiv.style.cssText = tempStr;
    }
  }

  function updateProgress(percentage){
    let progressBar = document.getElementById('progCircle');
    let text = document.getElementById('progText'); 
    
    let percentageText;
    if(percentage > 1)
      percentageText = "uploading (100%)"
    else if(percentage < 0)
      percentageText = "uploading (0%)"
    else
      percentageText = "uploading (" + parseInt(percentage * 100) + "%)"
    text.innerText = percentageText


    var angle;
    let imdValue = percentage / 0.6 // range 0-1
    angle = imdValue * 225          // range 0-225
    angle = angle                   // range 0-225
    angle = angle / 360             // range 0-1
    percentage = percentage + angle
    if(percentage >= 1)
      percentage = percentage - 1

    var barCTX = progressBar.getContext("2d");
    var quarterTurn = Math.PI / 2;
    var endingAngle = ((2*percentage) * Math.PI) - quarterTurn;
    var startingAngle = ((2*angle) * Math.PI) - quarterTurn;

    progressBar.width = progressBar.width;
    barCTX.lineCap = 'square';

    barCTX.beginPath();
    barCTX.lineWidth = 8;
    barCTX.strokeStyle = '#808B96';
    barCTX.arc(75,75,65,startingAngle, endingAngle);
    barCTX.stroke();
  }


  // function runTest() {
  //   debugPhase = true;
  //   // const refreshRate = 1000 / 60;
  //   // const maxXPosition = 400;
  //   // let rect = document.getElementById('rect1');
  //   // let speedX = 1;
  //   // let positionX = 0;
  //   var anim = 0;
  //   function step() {
  //     updateProgress(anim);
  //     anim = anim + 0.005;
  //     if(anim >= 1)
  //       anim = 0;
  //       window.requestAnimationFrame(step);
  //   }

  //   window.requestAnimationFrame(step);
  // }

</script>

<style>
  main {
    background: #fff;
    width: 100vw;
    height: 100vh;

    text-align: center;
    margin: 0 auto;
  }

  .videoDiv{
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
  }

  .videoPlayer{
    width: 100%;
    height: 100%;
  }

  /* change player background color */
  .content{
    height: 100%;
    width: 100%;
    position: relative;
    top: 0%;
    left: 0%;
    overflow: hidden;
    background: #fff;
  }

  .video-js {
    height: 100%;
    width: 100%;
    position: relative;
    top: 0%;
    left: 0%;
  }


  .videooverlay {
    position: absolute;
    width: 100%;
    height: auto;
    background: rgb(50, 50, 50, 0.5);
    text-align: center;
    vertical-align: middle;
  }

  .videooverlay>p{
    color: rgb(0, 0, 0, 0.65);
    padding: 30px;
    margin-bottom: 10px;

    font-family: Arial, Helvetica, sans-serif;
    font-weight: 500;
    font-size: 28px;
  }

  .buttonNext{
    color: #fff !important;
    text-transform: uppercase;
    text-decoration: none;
    font-size: 20px;
    background: #434343;
    /* width: 120px; */
    border-radius: 5px;
    transition: all 0.4s ease 0s;
  }

  .buttonRecord{
    color: #fff !important;
    text-transform: uppercase;
    text-decoration: none;
    font-size: 20px;
    background: #fa1a70;
    /* width: 130px; */
    border-radius: 5px;
    transition: all 0.4s ease 0s;
  }

  .buttonRetry{
    color: #fff !important;
    text-transform: uppercase;
    text-decoration: none;
    font-size: 20px;
    background: #434343;
    border-radius: 5px;
    border-color: #434343;
    transition: all 0.4s ease 0s;
    margin: 20px;
  }

  .buttonSubmit{
    color: #fff !important;
    text-transform: uppercase;
    text-decoration: none;
    font-size: 20px;
    background: #434343;
    border-radius: 5px;
    border-color: #434343;
    transition: all 0.4s ease 0s;
    margin: 20px;
  }

  .buttonPlay{
    position: absolute;
    background: url("/play.png");
    background-size: cover;
    display: inline-block;
    border: none;
    width: 50px;
    height: 50px;
  }

  .buttonMidBottom{
    position: absolute;
    bottom: 10%;
    padding: 10px;
    border-radius: 5px;
    display: inline-block;
    border: none;
    width: auto;
  }

  .uploadControlls{
    position: absolute;
    bottom: 10%;
    padding: 10px;
    border-radius: 5px;
    display: inline-block;
    border: none;
    width: auto;
  }

  .content {
    height: 100vh;
    width: 100%;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
  }

  #uploadProgressDiv{
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  #uploadProgressDiv > canvas{
    width: 80px;
    height: 80px;
  }

  #uploadProgressDiv > h5{
    text-align: center;
    font-size: 26px;
    font-weight: 100;
    font-family: 'Quicksand';
    color: rgb(0, 0, 0, 0.65);
  }

</style>

<main>
    <div id="mainDiv" class="content">
      
      {#if !uploading}
        <div id="videoDiv">
          <video-js id="videoRecorder" playsinline class="video-js vjs-theme-defualt vjs-big-play-centered" />
          <video id="videoPlayer" class="video-js" style="display:none;"/>
        </div>
      {/if}

      <div class="videooverlay">

        {#if checkPhase}
          <p>Check yourself and start recording when you are ready</p>
        {/if}

        {#if recordingPhase}
          {#if !isRecording}
            <p>Record a short video telling us what you like about your new Casper mattress. 🙌🏻 Hit record when ready 👇🏻</p>
          {/if}
        {/if}


      </div> <!-- overlay div -->

      {#if uploading}
        <div id="uploadProgressDiv">
          <canvas id="progCircle" width="150px" height="150px"/>
          <h5 id="progText">uploading (0%)</h5>
        </div>
      {/if}


      {#if checkPhase}
        <button class="buttonMidBottom buttonNext" on:click|once={readyForRecording}>&#10132;&#8287;&#8287;Next</button>
      {/if}

      {#if recordingPhase}
        {#if !isRecording}
          <button class="buttonMidBottom buttonRecord" on:click|once={startRecording}>&#9898;&#8287;&#8287;RECORD</button>
        {/if}

        {#if isRecording}
          <button class="buttonMidBottom buttonRecord" on:click|once={stopRecording}>&#11036;&#8287;&#8287;STOP</button>
        {/if}
      {/if}

      {#if uploadPhase}
        {#if !uploading}
            {#if !isPlayingback}
              <button class="buttonPlay" on:click|once={playbackRecording}></button>
            {/if}
          <div class="uploadControlls">
            <button class="buttonRetry" on:click|once={resetRecording}>&#8635;&#8287;&#8287;RETRY</button>
            <button class="buttonSubmit" on:click|once={handleClick}><i class="fa fa-cloud-upload"></i>&#8287;&#8287;SUBMIT</button>
          </div>
        {/if}
      {/if}

    </div> <!-- content div -->
</main>