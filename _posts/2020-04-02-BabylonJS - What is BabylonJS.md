---
layout: post
author: Cody Merritt Anhorn
title: BabylonJS - What is BabylonJS?
categories: [blog, babylon.js]
tags: [BabylonJS, Game Development, Rendering]
---

What is BabylonJS and what do I use it for? Continuer reading to learn more!

## What is BabylonJS

BabylonJS is primarily a rendering engine, it has a whole slew of other features which makes it a pretty good browser based game engine. My personal usage of using this is me finding a really awesome feature an wrapping it in an abstraction/proxy that my platform would use. The mesh is central to the application, models and the such, so I created a EngineMesh which is just an extended object of the Mesh. This allows me to "shadow" the mesh and add mode details relating to my platform that would not other wise be available to my games.

Also by making all mesh/model creation go through a factory, I can alway return an EngineMesh and alway have a layer of abstraction away from the models. This allows for me to use this abstraction in other aspects of the engine, pointer picking to another one. Wrapping the mesh with more metadata data allows for special logic to happen during a pointer trigger, sending selection events that can then be used to display other UI details. So long story short, I am able to create custom logic with raw events from the BabylonJS engine.

This is just some of what I use BabylonJS, and this is just the start to my BabylonJS series. 

<a href="https://www.babylonjs.com/" target="_blank">Checkout Babylon.JS</a>

## Example

Below is a very simple application, showing off a sphere in space.

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
    var createScene = function () {
        var scene = new BABYLON.Scene(engine);
        var camera = new BABYLON.FreeCamera("camera1", new BABYLON.Vector3(0, 5, -10), scene);
        camera.setTarget(BABYLON.Vector3.Zero());
        var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
        var sphere = BABYLON.Mesh.CreateSphere("sphere1", 16, 2, scene);
        sphere.position.y = 1;
        return scene;
    };
    engine = createDefaultEngine();
    if (!engine) throw 'engine should not be null.';
    scene = createScene();;
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
        var createScene = function () {
            // Setup the scene
            var scene = new BABYLON.Scene(engine);
            var camera = new BABYLON.FreeCamera("camera1", new BABYLON.Vector3(0, 5, -10), scene);
            camera.setTarget(BABYLON.Vector3.Zero());
            var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
        
            // Create a sphere that we will be moved by the keyboard
            var sphere = BABYLON.Mesh.CreateSphere("sphere1", 16, 2, scene);
            sphere.position.y = 1;
            
            return scene;
        };
        
        engine = createDefaultEngine();
        if (!engine) throw 'engine should not be null.';
        scene = createScene();;
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