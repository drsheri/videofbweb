<script>
  import { onMount } from "svelte";
  import { navigate } from "svelte-routing";
  import Thankyou from "./Thankyou.svelte";

  import { onDestroy } from "svelte";

  var videoRecorder = null;

  var checkPhase = true;
  var recordingPhase = false;
  var uploadPhase = false;

  var isRecording = false;
  var isPlayingback = false;
  var uploading = false;
  var hasUploaded = false;

  var lastRecording = null;

  onMount(function() {
    onDestroy(() => console.log("onDestroy"));

    // apply some workarounds for opera browser
    applyVideoWorkaround();

    // recording
    var videoRecorderOptions = {
      controls: false,
      fill: true,
      responsive: true,
      fluid: false,
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

    // user completed recording and stream is available
    videoRecorder.on("finishRecord", function() {
      console.log("finished recording: ", videoRecorder.recordedData);
      lastRecording = videoRecorder.recordedData;

      var recorder = document.getElementById('videoRecorder');
      var player = document.getElementById('videoPlayer');

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
    player.play();
    player.on('ended', function(){
      isPlayingback = false;
      // player.dispose();
      // // mainDiv

      // let reMake = document.createElement('video');
      // reMake.id = 'videoPlayer';
      // reMake.class = 'video-js';
      // reMake.style = 'display: none;';
      // document.getElementById('mainDiv').appendChild(reMake);
    });
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

    var recorder = document.getElementById('videoRecorder');
    var player = document.getElementById('videoPlayer');

    recorder.style.display = "block";
    player.style.display = "none";

  }

  function uploadVideo(){

  }

  function readyForRecording(){
    checkPhase = false;
    recordingPhase = true;
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
       document.querySelector("#progressbar>div").style.width = parseInt((evt.loaded * 100) / evt.total) + "%";
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
</script>

<style>

  /* change player background color */
  .content .video-js {
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
    background-color: rgba(70, 70, 70, 0.8);
    text-align: center;
    vertical-align: middle;
  }

  .videooverlay>p{
    font-family: Arial, Helvetica, sans-serif;
    font-weight: 100;
    font-size: 25px;
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
    transition: all 0.4s ease 0s;
    margin-left: -75px;
  }

  .buttonSubmit{
    color: #fff !important;
    text-transform: uppercase;
    text-decoration: none;
    font-size: 20px;
    background: #434343;
    border-radius: 5px;
    transition: all 0.4s ease 0s;
    margin-left: 75px;
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

  .content {
    height: 100vh;
    width: 100%;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
  }

  #progressbar {
    background-color: black;
    border-radius: 13px;
    width: 60%;
    margin: 0 auto;  /* (height of inner div) / 2 + padding */
    padding: 3px;
    position: absolute;
  }

  #progressbar>div {
    background-color: orange;
    width: 1%;
    /* Adjust with JavaScript */
    height: 20px;
    border-radius: 10px;
  }
</style>

<main>
    <div id="mainDiv" class="content">
      
      <video-js id="videoRecorder" playsinline class="video-js vjs-theme-defualt vjs-big-play-centered" />

      <video id="videoPlayer" class="video-js" style="display:none;">
        
      </video>

      {#if uploading}
          <div id="progressbar"><div></div></div>
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
          <button class="buttonMidBottom buttonRetry" on:click|once={resetRecording}>&#8635;&#8287;&#8287;RETRY</button>
          <button class="buttonMidBottom buttonSubmit" on:click|once={handleClick}>&#xf0ee;&#8287;&#8287;SUBMIT</button>
        {/if}

        

      {/if}

    </div> <!-- content div -->
</main>