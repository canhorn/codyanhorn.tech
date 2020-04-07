---
layout: post
author: Cody Merritt Anhorn
title: BabylonJS - Loading a GLTF Mesh
categories: [blog, babylon.js]
tags: [BabylonJS, Game Development, Rendering]
---

This post is will go over how to use BabylonJS to load in GLTF model, including an example. I personally have chosen GLTF as the main format for my models. The main reason is that most modelers will export GLTF and it allow support animations allowing for me to handle everything about my assets in one format. 

> ***GLTF*** stands for ***Graphics Library Transmission Format***

## Snippet Example

Below is a short snippet for loading a GLTF mesh, very easy to use and only needing to include the BabylonJS loader, babylonjs.loaders.js.

<a href="https://doc.babylonjs.com/how_to/load_from_any_file_type">BabylonJS Loading GLTF How-To</a>

<a href="https://doc.babylonjs.com/api/classes/babylon.sceneloader">BabylonJS SceneLoader API</a>

<a href="https://doc.babylonjs.com/api/classes/babylon.sceneloader#importmesh">BabylonJS SceneLoader.ImportMesh API</a>

### Code to Load GLTF/Any Mesh into BabylonJS

~~~ javascript
BABYLON.SceneLoader.ImportMesh(
    undefined, // Name of meshes to load
    "/assets/posts/2020-04-07/", // Path on a server for the file
    "House.gltf", // The file name that should be loaded from the above path
    scene, // The scene to load this mesh/model file into
    function (
        meshes, 
        particleSystems,
        skeletons,
        animationList
    ) {
        // Custom Code to run after Loading has finished
    }
);
~~~

## Live Example

Below is an example of a model loaded in using the SceneLoader.ImportMesh for a GLTF mesh.

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
    var createDefaultScene = function() {
        // Setup the scene
        var scene = new BABYLON.Scene(engine);
        var camera = new BABYLON.ArcRotateCamera(
            "camera1", 
            -(Math.PI / 2), 
            Math.PI / 2, 
            75, 
            new BABYLON.Vector3(0, 0, 0), 
            scene
        );
        var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
        return scene;
    };
    var createScene = function (scene) {
        BABYLON.SceneLoader.ImportMesh(undefined, "/assets/posts/2020-04-07/", "House.gltf", scene, function (
        meshes, 
        particleSystems,
        skeletons,
        animationList) {
            scene.createDefaultCameraOrLight(true);
            scene.activeCamera.attachControl(canvas, false);
        });
        return scene;
    };
    engine = createDefaultEngine();
    if (!engine) throw 'engine should not be null.';
    scene = createDefaultScene();
    scene = createScene(scene);
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
        var createDefaultScene = function() {
            // Setup the scene
            var scene = new BABYLON.Scene(engine);
            var camera = new BABYLON.ArcRotateCamera(
                "camera1", 
                -(Math.PI / 2), 
                Math.PI / 2, 
                75, 
                new BABYLON.Vector3(0, 0, 0), 
                scene
            );
            var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
            return scene;
        };
        var createScene = function (scene) {
            BABYLON.SceneLoader.ImportMesh(
                undefined, 
                "/assets/posts/2020-04-07/", 
                "House.gltf", 
                scene, 
                function (
                    meshes, 
                    particleSystems,
                    skeletons,
                    animationList
                ) {
                    scene.createDefaultCameraOrLight(true);
                    scene.activeCamera.attachControl(canvas, false);
                }
            );
            return scene;
        };
        
        engine = createDefaultEngine();
        if (!engine) throw 'engine should not be null.';
        scene = createDefaultScene();
        scene = createScene(scene);
        sceneToRender = scene

        // Start rendering the scene based on the engine render loop.
        engine.runRenderLoop(function () {
            if (sceneToRender) {
                sceneToRender.render();
            }
        });

        // Resize
        window.addEventListener("resize", function () {
            engine.resize();
        });
    </script>
</body>
</html>

~~~