    <style>
        /* change player background color */
        #myVideo {
            background-color: #9ab87a;
        }

        .content{
    width:640px;
    height:480px;
    position:absolute;
    left:50%;
    top:50%;
    margin:-240px 0 0 -320px;
}
    </style>


    <script>
    import { onMount } from 'svelte';
    import { navigate } from "svelte-routing";
    import Thankyou from "./Thankyou.svelte";


    import { onDestroy } from 'svelte';

    var uploadVideo = false
    var player = null;


	onMount( function () {
        var options = {
            controls: true,
            width: 640,
            height: 480,
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

        player = videojs('myVideo', options, function () {
            // print version information at startup
            var msg = 'Using video.js ' + videojs.VERSION +
                ' with videojs-record ' + videojs.getPluginVersion('record') +
                ' and recordrtc ' + RecordRTC.version;
            videojs.log(msg);
        });

    
        // error handling
        player.on('deviceError', function () {
            console.log('device error:', player.deviceErrorCode);
        });

        player.on('error', function (element, error) {
            console.error(error);
        });

        // user clicked the record button and started recording
        player.on('startRecord', function () {
            console.log('started recording!');
        });

        // user completed recording and stream is available
        player.on('finishRecord', function () {
            uploadVideo = true;
            console.log('finished recording: ', player.recordedData);

        });

    });
    function handleClick() {
                    // the blob object contains the recorded data that
            // can be downloaded by the user, stored on server etc.
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

            // var files = document.getElementById("photoupload").files;
            // if (!files.length) {
            //     return alert("Please choose a file to upload first.");
            // }
            var uploadFile = player.recordedData;
            var timestamp = new Date().getTime().toString();
            var uploadFile = new File([uploadFile], timestamp);
            // var uploadFile = JSON.stringify(uploadFile)
            // var filesList = [uploadFile];
            // var file = files[0];
            // var fileName = file.name;
            var albumPhotosKey = "videostuff" + "/";
            var photoKey = albumPhotosKey + timestamp + ".mp4";

            // Use S3 ManagedUpload class as it supports multipart uploads
            // debugger;
            var upload = new AWS.S3.ManagedUpload({
                params: {
                    Bucket: albumBucketName,
                    Key: photoKey,
                    Body: uploadFile
                }
            });
            
            var promise = upload.on('httpUploadProgress', function(evt) {
          console.log("Uploaded :: " + parseInt((evt.loaded * 100) / evt.total)+'%');
        })
        .promise();
            promise.then(
                function (data) {
                    player.dispose();
                     navigate("/thankyou", { replace: true });
                },
                function (err) {
                    return alert("There was an error uploading your photo: ", err.message);
                }
            );


	}
    
    </script>

<div class="content">
    <video id="myVideo" playsinline class="video-js vjs-default-skin"></video>
    {#if uploadVideo}
        <button on:click|once={handleClick}>
	        Done Recording
        </button>
    {/if}
</div>

