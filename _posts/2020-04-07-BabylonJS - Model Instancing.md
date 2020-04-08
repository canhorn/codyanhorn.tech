---
layout: post
author: Cody Merritt Anhorn
title: BabylonJS - Model Instancing
categories: [blog, babylon.js]
tags: [BabylonJS, Game Development, Rendering]
---

Instancing a mesh is a very powerful feature that allows for the rendering of identical object with the power of hardware acceleration. Doing this allows for the GPU to handle very large amounts of identical meshes and have very little performance lose than loading all as independent meshes.

## Details

Below is a short snippet for loading a GLTF mesh, then creating instances of that mesh from the loaded mesh.

<a href="https://doc.babylonjs.com/how_to/how_to_use_instances">BabylonJS Instances How-To</a>

### Loading GLTF Mesh with Instancing 

~~~ javascript
BABYLON.SceneLoader.ImportMesh(
    undefined, 
    "/assets/posts/2020-04-07/", 
    "House.gltf", 
    scene, 
    function (newMeshes
    ) {
        var mesh = newMeshes[1];
        // Make the "root" mesh not visible. The instanced versions of it that we
        // create below will be visible.
        mesh.isVisible = false;
        // Remove the parent, every GLTF file comes from a right handed world. 
        // To get it into Babylon.js left handed world, the loader will add an
        //  arbitrary parent that is adding a negative scale on z.
        mesh.setParent(null);
        for (var index = 0; index < 100; index++) {
            var newInstance = mesh.createInstance("i" + index);
                newInstance.position.x = Math.floor(getRandomArbitrary(-300, 300));
                newInstance.position.z = Math.floor(getRandomArbitrary(-300, 300));
        }
    }
);
function getRandomArbitrary(min, max) {
    return Math.random() * (max - min) + min;
}
~~~

## Example

Below is a very simple application, showing off 100 instances created from a single loaded mesh.

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
            (Math.PI / 2) - 0.3, 
            450, 
            new BABYLON.Vector3(0, 0, 0), 
            scene
        );
        var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
        scene.createDefaultCameraOrLight(true);
        scene.activeCamera.attachControl(canvas, false);
        return scene;
    };
    var createScene = function (scene) {
        BABYLON.SceneLoader.ImportMesh(undefined, "/assets/posts/2020-04-07/", "House.gltf", scene, function (meshes) {
            var mesh = meshes[1];
            // Make the "root" mesh not visible. The instanced versions of it that we
            // create below will be visible.
            mesh.isVisible = false;
            // Remove the parent, every GLTF file comes from a right handed world. 
            // To get it into Babylon.js left handed world, the loader is adding an
            //  arbitrary parent that is adding a negative scale on z.
            mesh.setParent(null);
            for (var index = 0; index < 100; index++) {
                var newInstance = mesh.createInstance("i" + index);
                 newInstance.position.x = Math.floor(getRandomArbitrary(-300, 300));
                 newInstance.position.z = Math.floor(getRandomArbitrary(-300, 300));
            }
        });
        function getRandomArbitrary(min, max) {
            return Math.random() * (max - min) + min;
        }
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
                (Math.PI / 2) - 0.3, 
                450, 
                new BABYLON.Vector3(0, 0, 0), 
                scene
            );
            var light = new BABYLON.HemisphericLight("light1", new BABYLON.Vector3(0, 1, 0), scene);
            scene.createDefaultCameraOrLight(true);
            scene.activeCamera.attachControl(canvas, false);
            return scene;
        };
        var createScene = function (scene) {
            BABYLON.SceneLoader.ImportMesh(undefined, "/assets/posts/2020-04-07/", "House.gltf", scene, function (meshes) {
                var mesh = meshes[1];
                // Make the "root" mesh not visible. The instanced versions of it that we
                // create below will be visible.
                mesh.isVisible = false;
                // Remove the parent, every GLTF file comes from a right handed world. 
                // To get it into Babylon.js left handed world, the loader is adding an
                //  arbitrary parent that is adding a negative scale on z.
                mesh.setParent(null);
                for (var index = 0; index < 100; index++) {
                    var newInstance = mesh.createInstance("i" + index);
                    newInstance.position.x = Math.floor(getRandomArbitrary(-300, 300));
                    newInstance.position.z = Math.floor(getRandomArbitrary(-300, 300));
                }
            });
            function getRandomArbitrary(min, max) {
                return Math.random() * (max - min) + min;
            }
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
</body>
</html>

~~~