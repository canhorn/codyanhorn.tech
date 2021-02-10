---
layout: post
author: Cody Merritt Anhorn
title: BabylonJS - Simple Rendering Scene
categories: [blog]
tags: [BabylonJS, Game Development, Rendering]
---

When getting starting in BabylonJS you need to first understand a few things. For one what is BabylonJS?

## What is BabyonJS

~~~ javascript
var canvas = document.getElementById("renderCanvas");

var engine = null;
var scene = null;
var sceneToRender = null;
var createDefaultEngine = function() { return new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true }); };
var createScene = function () {
    // Setup a simple scene
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

// Start the rendering the loop, without it it will not "render" the scene.
engine.runRenderLoop(function () {
    if (sceneToRender) {
        sceneToRender.render();
    }
});
~~~