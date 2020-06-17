<script>
  import { onMount } from "svelte";
  import { navigate } from "svelte-routing";
  import Thankyou from "./Thankyou.svelte";

  import { onDestroy } from "svelte";

  var uploadVideo = false;
  var isRecording = false;
  var uploading = false;
  var player = null;

  onMount(function() {
    onDestroy(() => console.log("onDestroy"));

    var options = {
      controls: true,
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

    // apply some workarounds for opera browser
    applyVideoWorkaround();

    player = videojs("myVideo", options, function() {
      // print version information at startup
      var msg =
        "Using video.js " +
        videojs.VERSION +
        " with videojs-record " +
        videojs.getPluginVersion("record") +
        " and recordrtc " +
        RecordRTC.version;
      videojs.log(msg);
      player.record().getDevice();
    });

    // error handling
    player.on("deviceError", function() {
      console.log("device error:", player.deviceErrorCode);
    });

    player.on("error", function(element, error) {
      console.error(error);
    });

    // user clicked the record button and started recording
    player.on("startRecord", function() {
      console.log("started recording!");
      isRecording = true;
    });

    // user completed recording and stream is available
    player.on("finishRecord", function() {
      isRecording = false;
      uploadVideo = true;
      console.log("finished recording: ", player.recordedData);
    });
  });

  function startRecording() {
    if (player.record().isRecording()) {
      player.record().stop();
    } else {
      player.record().start();
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

    var uploadFile = player.recordedData;
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
        player.dispose();
        navigate("/thankyou", { replace: true });
      },
      function(err) {
        return alert("There was an error uploading your photo: ", err.message);
      }
    );
  }
</script>

<style>
  /* change player background color */
  .content .video-js {
    height: 95%;
    width: 95%;
    position: relative;
    top: 0%;
    left: 0%;
  }

  .videooverlay {
    position: absolute;
    /* top:0;
        left:0;
        right:0;
        bottom:0; */
    /* width: 100%; */
    width: inherit;
    height: 100px;
    background-color: rgba(255, 255, 255, 0.49);
    text-align: center;
    vertical-align: middle;
    line-height: 90px;
  }

  .content {
    height: 95vh;
    width: 95vw;
    padding: 0;
    /* margin: 0;
    display: -webkit-box;
    display: -moz-box;
    display: -ms-flexbox;
    display: -webkit-flex; */
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
  }

  #progressbar>div {
    background-color: orange;
    width: 1%;
    /* Adjust with JavaScript */
    height: 20px;
    border-radius: 10px;
  }
</style>

<div class="content">
  <video
    id="myVideo"
    playsinline
    class="video-js vjs-theme-defualt vjs-big-play-centered" />
  <div class="videooverlay">
    {#if uploading}
      <p>uploading your video</p>
    {:else}
      <p>Check yourself and start recording when you are ready</p>
    {/if}

    {#if uploading}
      <div id="progressbar">
        <div />
      </div>
    {:else if uploadVideo}
      {#if isRecording}
        <button on:click|once={startRecording}>Done Recording</button>
      {:else}
        <button on:click|once={startRecording}>Restart Recording</button>
      {/if}
      <button on:click|once={handleClick}>Upload</button>
    {:else if isRecording}
      <button on:click|once={startRecording}>Done Recording</button>
    {:else}
      <button on:click|once={startRecording}>Begin Recording</button>
    {/if}

  </div>
</div>
