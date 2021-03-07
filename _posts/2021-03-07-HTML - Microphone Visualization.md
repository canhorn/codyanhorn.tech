---
layout: post
author: Cody Merritt Anhorn
title: HTML - Microphone Visualization
categories: [blog]
tags: [HTML, OBS]
---

In this Article I will go over using HTML and JavaScript to create a simple Talking Avatar using the Mic and a two sprites.

## Background

Since I am a twitch streamer, I though it would be fun to create an avatar that talks when I talk. I did not want to go full VTuber like avatar, but I do like making things in the browser so I though it would be fun to play with the Microphone and make a Avatar change when talking. 

Using the <code>navigator.mediaDevices.getUserMedia({ audio: true })</code> JavaScript API you can get permission to use the audio devices of the browser. Using the AudioContext API you can also get access to an Analyser of the AudioContext that can then be used to calculate the decibels currently being picked up by the mic.

## Demo and Source

Here is a working demo, and below the demo is the source for the Demo.

<div class="demo-section">
    <div class="avatar-container">
        <img
            id="avatar-sprite"
            style="width: initial"
            src="/image/Posts/2021-03-07/Idle.png"
        />
    </div>
    <button id="button">Start Listening</button>

<script>
window.onload = function () {
    // Below are some parameters you can tweak to customize your Avatar
    // This threshold is for how many decibels the mic should pickup to trigger the talking sprite
    const LOUD_VOLUME_THRESHOLD = 30;
    // How often we should check the Threshold
    const INTERVAL_TIME = 100;
    // These are the Idle and Talk Sprite we will use
    const idleSprite = "/image/Posts/2021-03-07/Idle.png";
    const talkSprite = "/image/Posts/2021-03-07/Talk.png";

    // Grab our Avatar Sprite so we can easily update the src of the image to simulate talking!
    const avatarSprite = document.getElementById("avatar-sprite");

    // The logic we should run when the Mic is permission is allowed 
    const soundAllowed = function (stream) {
        // We create an AudioContext and feed it our stream we received from mediaDevice request.
        const audioContext = new AudioContext();
        const audioStream = audioContext.createMediaStreamSource(stream);
        const analyser = audioContext.createAnalyser();
        const fftSize = 1024;
        analyser.fftSize = fftSize;
        audioStream.connect(analyser);

        const bufferLength = analyser.frequencyBinCount;
        const frequencyArray = new Uint8Array(bufferLength);
        document.getElementById("button").innerHTML = "Listening...";

        // We create an interval that checks the decibels and updates the Avatar to our threshold. 
        setInterval(() => {
            analyser.getByteFrequencyData(frequencyArray);
            // Loop the frequencyArray adding all the values together to get our total decibels.
            let total = 0;
            for (let i = 0; i < 255; i++) {
                var x = frequencyArray[i];
                total += x * x;
            }
            const rms = Math.sqrt(total / bufferLength);
            let db = 20 * (Math.log(rms) / Math.log(10));
            db = Math.max(db, 0);

            if (db >= LOUD_VOLUME_THRESHOLD) {
                // We are being Loud, so make our Avatar Talk
                avatarSprite.src = talkSprite;
            } else {
                // We are quite, so Idle our Avatar
                avatarSprite.src = idleSprite;
            }
        }, INTERVAL_TIME);
    };

    const start = () => {
        // Here we use the navigator.mediaDevices API to get the UserMedia for Audio
        navigator.mediaDevices
            .getUserMedia({ audio: true })
            .then(soundAllowed)
            // If the user denies the Mic permission, we show an error message
            // Will also trigger in the case of any error.
            .catch((error) => {
                document.getElementById("button").innerHTML =
                    "You must allow your microphone.";

                // To help debugging we display the full 'error' in the console
                console.log(error);
            });
    };

    // Enable the button, in the case autostart is not found in the URL
    document.getElementById("button").onclick = function () {
        start();
    };

    // A little hook that will autostart listening if the URL contains 'autostart'
    if (window.location.toString().includes("autostart")) {
        start();
    }
};
</script>

</div>

### HTML Source

~~~ html
<div class="avatar-container">
    <img
        id="avatar-sprite"
        style="width: initial"
        src="/image/Posts/2021-03-07/Idle.png"
    />
</div>
<button id="button">Start Listening</button>
~~~

### JavaScript Source

~~~ js
window.onload = function () {
    // Below are some parameters you can tweak to customize your Avatar
    // This threshold is for how many decibels the mic should pickup to trigger the talking sprite
    const LOUD_VOLUME_THRESHOLD = 30;
    // How often we should check the Threshold
    const INTERVAL_TIME = 100;
    // These are the Idle and Talk Sprite we will use
    const idleSprite = "/image/Posts/2021-03-07/Idle.png";
    const talkSprite = "/image/Posts/2021-03-07/Talk.png";

    // Grab our Avatar Sprite so we can easily update the src of the image to simulate talking!
    const avatarSprite = document.getElementById("avatar-sprite");

    // The logic we should run when the Mic is permission is allowed 
    const soundAllowed = function (stream) {
        // We create an AudioContext and feed it our stream we received from mediaDevice request.
        const audioContext = new AudioContext();
        const audioStream = audioContext.createMediaStreamSource(stream);
        const analyser = audioContext.createAnalyser();
        const fftSize = 1024;
        analyser.fftSize = fftSize;
        audioStream.connect(analyser);

        const bufferLength = analyser.frequencyBinCount;
        const frequencyArray = new Uint8Array(bufferLength);
        document.getElementById("button").innerHTML = "Listening...";

        // We create an interval that checks the decibels and updates the Avatar to our threshold. 
        setInterval(() => {
            analyser.getByteFrequencyData(frequencyArray);
            // Loop the frequencyArray adding all the values together to get our total decibels.
            let total = 0;
            for (let i = 0; i < 255; i++) {
                var x = frequencyArray[i];
                total += x * x;
            }
            const rms = Math.sqrt(total / bufferLength);
            let db = 20 * (Math.log(rms) / Math.log(10));
            db = Math.max(db, 0);

            if (db >= LOUD_VOLUME_THRESHOLD) {
                // We are being Loud, so make our Avatar Talk
                avatarSprite.src = talkSprite;
            } else {
                // We are quite, so Idle our Avatar
                avatarSprite.src = idleSprite;
            }
        }, INTERVAL_TIME);
    };

    const start = () => {
        // Here we use the navigator.mediaDevices API to get the UserMedia for Audio
        navigator.mediaDevices
            .getUserMedia({ audio: true })
            .then(soundAllowed)
            // If the user denies the Mic permission, we show an error message
            // Will also trigger in the case of any error.
            .catch((error) => {
                document.getElementById("button").innerHTML =
                    "You must allow your microphone.";

                // To help debugging we display the full 'error' in the console
                console.log(error);
            });
    };

    // Enable the button, in the case autostart is not found in the URL
    document.getElementById("button").onclick = function () {
        start();
    };

    // A little hook that will autostart listening if the URL contains 'autostart'
    if (window.location.toString().includes("autostart")) {
        start();
    }
};
~~~