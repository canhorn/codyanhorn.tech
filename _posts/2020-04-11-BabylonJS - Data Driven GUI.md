---
layout: post
author: Cody Merritt Anhorn
title: BabylonJS - Data Driven GUI
categories: [blog, babylon.js]
tags: [BabylonJS, Game Development, GUI]
---

In this article we will take a JavaScript JSON array of GUI control data and events, then using loops and Object.assign we will create GUI controls. I use a more robust version of this in my GUI generation, but that is more to allow for complex mapping of GUI controls to child controls. With this it allows for 90% of GUI scenarios to be created in a dynamic and easy to change structure.

## Details

Here is an example of the data we will want to turn into GUI controls. Do note that this is JavaScript, it has a function callback that does some logic on a sphere.

~~~ javascript
const guiData = [{
    type: "Slider",
    data: {
        minimum: 0.1,
        maximum: 20,
        value: 5,
        height: "20px",
        width: "150px",
        color: "#003399",
        background: "grey",
        left: "120px",
        horizontalAlignment: BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_LEFT,
        verticalAlignment: BABYLON.GUI.Control.VERTICAL_ALIGNMENT_CENTER,
    },
    events: [{
        name: "onValueChangedObservable",
        callback: function (value) {
            var sphere = scene.getMeshByName("sphere");
            sphere.scaling = unitVec.scale(value);
        }
    }]
}]
~~~

Here is the logic to create dynamic data based on the above GUI control details.

~~~ javascript
const createGui = function (advancedTexture) {
    // Here we loop through the array of gui controls
    for (const controlDetails of guiData) {
        // Using the type, which correlates to a control on the GUI
        //  namespace, we are able to create the control.
        const control = new BABYLON.GUI[controlDetails.type]();

        // Using the Object.assign we are able to assign the override
        //  data from the control details.
        Object.assign(control, controlDetails.data);

        // This loop will add any existing events to the just created control.
        for(const eventDetails of controlDetails.events) {
            if(control[eventDetails.name]) {
                control[eventDetails.name].add(
                    eventDetails.callback
                );
            }
        }

        // Add the created control to the screen.
        advancedTexture.addControl(control);
    }
};
~~~


## Example

Below is a very simple application showing off a GUI created from a JSON representation.

Here is a <a href="https://www.babylonjs-playground.com/#56CZSZ" target="_blank">playground link</a> to see the code and experiment with it your self.

<script src="https://code.jquery.com/pep/0.4.2/pep.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/dat-gui/0.6.2/dat.gui.min.js"></script>
<script src="https://preview.babylonjs.com/ammo.js"></script>
<script src="https://preview.babylonjs.com/cannon.js"></script>
<script src="https://preview.babylonjs.com/Oimo.js"></script>
<script src="https://preview.babylonjs.com/libktx.js"></script>
<script src="https://preview.babylonjs.com/earcut.min.js"></script>
<script src="https://preview.babylonjs.com/babylon.js"></script>
<script src="https://preview.babylonjs.com/inspector/babylon.inspector.bundle.js"></script>
<script src="https://preview.babylonjs.com/materialsLibrary/babylonjs.materials.min.js"></script>
<script src="https://preview.babylonjs.com/proceduralTexturesLibrary/babylonjs.proceduralTextures.min.js"></script>
<script src="https://preview.babylonjs.com/postProcessesLibrary/babylonjs.postProcess.min.js"></script>
<script src="https://preview.babylonjs.com/loaders/babylonjs.loaders.js"></script>
<script src="https://preview.babylonjs.com/serializers/babylonjs.serializers.min.js"></script>
<script src="https://preview.babylonjs.com/gui/babylon.gui.min.js"></script>
<style>
    html, body {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
    }
    #renderCanvas {
        width: 100%;
        height: 100%;
        touch-action: none;
    }
</style>
<canvas id="renderCanvas"></canvas>
<script>
    var canvas = document.getElementById("renderCanvas");
    var engine = null;
    var scene = null;
    var sceneToRender = null;
    var createDefaultEngine = function() { return new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true }); };
    const unitVec = new BABYLON.Vector3(1, 1, 1);
    const guiData = [{
        type: "Slider",
        data: {
            minimum: 0.1,
            maximum: 20,
            value: 5,
            height: "20px",
            width: "150px",
            color: "#003399",
            background: "grey",
            left: "120px",
            horizontalAlignment: BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_LEFT,
            verticalAlignment: BABYLON.GUI.Control.VERTICAL_ALIGNMENT_CENTER,
        },
        events: [{
            name: "onValueChangedObservable",
            callback: function (value) {
                var sphere = scene.getMeshByName("sphere");
                sphere.scaling = unitVec.scale(value);
            }
        }]
    }];
    const createScene = function () {
        const scene = new BABYLON.Scene(engine);
        const light = new BABYLON.PointLight("Omni", new BABYLON.Vector3(0, 100, 100), scene);
        const camera = new BABYLON.ArcRotateCamera("Camera", 0, 0.8, 100, new BABYLON.Vector3.Zero(), scene);
        camera.attachControl(canvas, true);
        const sphere = BABYLON.MeshBuilder.CreateSphere("sphere", {}, scene);
        sphere.scaling = unitVec.scale(5);
        const advancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI("UI", true, scene);
        // Here we loop through the array of gui controls
        createGui(advancedTexture)
        return scene;
    };
    const createGui = function (advancedTexture) {
        // Here we loop through the array of gui controls
        for (const controlDetails of guiData) {
            // Using the type, which correlates to a control on the GUI
            //  namespace, we are able to create the control.
            const control = new BABYLON.GUI[controlDetails.type]();
            // Using the Object.assign we are able to assign the override
            //  data from the control details.
            Object.assign(control, controlDetails.data);
            // This loop will add any existing events to the just created control.
            for(const eventDetails of controlDetails.events) {
                if(control[eventDetails.name]) {
                    control[eventDetails.name].add(
                        eventDetails.callback
                    );
                }
            }
            // Add the created control to the screen.
            advancedTexture.addControl(control);
        }
    };
    engine = createDefaultEngine();
    if (!engine) throw 'engine should not be null.';
    scene = createScene();
    sceneToRender = scene
    engine.runRenderLoop(function () {
        if (sceneToRender) {
            sceneToRender.render();
        }
    });
    window.addEventListener("resize", function () {
        engine.resize();
    });
</script>

***Code for above example***
~~~ html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

        <title>Babylon.js sample code</title>

        <!-- Babylon.js -->
        <script src="https://code.jquery.com/pep/0.4.2/pep.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/dat-gui/0.6.2/dat.gui.min.js"></script>
        <script src="https://preview.babylonjs.com/ammo.js"></script>
        <script src="https://preview.babylonjs.com/cannon.js"></script>
        <script src="https://preview.babylonjs.com/Oimo.js"></script>
        <script src="https://preview.babylonjs.com/libktx.js"></script>
        <script src="https://preview.babylonjs.com/earcut.min.js"></script>
        <script src="https://preview.babylonjs.com/babylon.js"></script>
        <script src="https://preview.babylonjs.com/inspector/babylon.inspector.bundle.js"></script>
        <script src="https://preview.babylonjs.com/materialsLibrary/babylonjs.materials.min.js"></script>
        <script src="https://preview.babylonjs.com/proceduralTexturesLibrary/babylonjs.proceduralTextures.min.js"></script>
        <script src="https://preview.babylonjs.com/postProcessesLibrary/babylonjs.postProcess.min.js"></script>
        <script src="https://preview.babylonjs.com/loaders/babylonjs.loaders.js"></script>
        <script src="https://preview.babylonjs.com/serializers/babylonjs.serializers.min.js"></script>
        <script src="https://preview.babylonjs.com/gui/babylon.gui.min.js"></script>

        <style>
            html, body {
                overflow: hidden;
                width: 100%;
                height: 100%;
                margin: 0;
                padding: 0;
            }

            #renderCanvas {
                width: 100%;
                height: 100%;
                touch-action: none;
            }
        </style>
    </head>
<body>
    <canvas id="renderCanvas"></canvas>
    <script>
        var canvas = document.getElementById("renderCanvas");
        var engine = null;
        var scene = null;
        var sceneToRender = null;
        var createDefaultEngine = function() { return new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true }); };
        const unitVec = new BABYLON.Vector3(1, 1, 1);
        const guiData = [{
            type: "Slider",
            data: {
                minimum: 0.1,
                maximum: 20,
                value: 5,
                height: "20px",
                width: "150px",
                color: "#003399",
                background: "grey",
                left: "120px",
                horizontalAlignment: BABYLON.GUI.Control.HORIZONTAL_ALIGNMENT_LEFT,
                verticalAlignment: BABYLON.GUI.Control.VERTICAL_ALIGNMENT_CENTER,
            },
            events: [{
                name: "onValueChangedObservable",
                callback: function (value) {
                    var sphere = scene.getMeshByName("sphere");
                    sphere.scaling = unitVec.scale(value);
                }
            }]
        }];
        const createScene = function () {
            const scene = new BABYLON.Scene(engine);
            const light = new BABYLON.PointLight("Omni", new BABYLON.Vector3(0, 100, 100), scene);
            const camera = new BABYLON.ArcRotateCamera("Camera", 0, 0.8, 100, new BABYLON.Vector3.Zero(), scene);
            camera.attachControl(canvas, true);
            const sphere = BABYLON.MeshBuilder.CreateSphere("sphere", {}, scene);
            sphere.scaling = unitVec.scale(5);
            const advancedTexture = BABYLON.GUI.AdvancedDynamicTexture.CreateFullscreenUI("UI", true, scene);
            // Here we loop through the array of gui controls
            createGui(advancedTexture)
            return scene;
        };
        const createGui = function (advancedTexture) {
            // Here we loop through the array of gui controls
            for (const controlDetails of guiData) {
                // Using the type, which correlates to a control on the GUI
                //  namespace, we are able to create the control.
                const control = new BABYLON.GUI[controlDetails.type]();
                // Using the Object.assign we are able to assign the override
                //  data from the control details.
                Object.assign(control, controlDetails.data);
                // This loop will add any existing events to the just created control.
                for(const eventDetails of controlDetails.events) {
                    if(control[eventDetails.name]) {
                        control[eventDetails.name].add(
                            eventDetails.callback
                        );
                    }
                }
                // Add the created control to the screen.
                advancedTexture.addControl(control);
            }
        };
        engine = createDefaultEngine();
        if (!engine) throw 'engine should not be null.';
        scene = createScene();
        sceneToRender = scene
        engine.runRenderLoop(function () {
            if (sceneToRender) {
                sceneToRender.render();
            }
        });
        window.addEventListener("resize", function () {
            engine.resize();
        });
    </script>
</body>
</html>
~~~
