# StyleAR Manual for PlayCanvas

> Creator: <honami82@deepixel.xyz>  
> Date: 2024/07/10
> Update: 2026/05/13

## Description

* This is a library that implements real-time virtual fitting.

## Requirement

> Android in-app browsers are not supported. (Implementation needed to run on Chrome using intent)  
> iOS in-app browsers are supported

| OS      |            |
| ------- | ---------- |
| Windows | Windows 10 |
| Mac     | 12         |
| Android | 24         |
| iOS     | 15         |

* Mobile
| Browser             | Version   | Release date |
| ------------------- | --------- | ------------ |
| chrome              | 115       | 2023‑07‑12   |
| safari              | 16.6      | 2023‑07‑24   |
| edge                | 115       | 2023‑06‑13   |
| whale               | 3.0.1.2   | 2023‑09‑15   |
| samsung browser     | 22.0.0.54 | 2023‑07‑13   |
| firefox             | 116.0     | 2023‑08‑01   |

> Encouraging users to use Chrome, which offers better performance compared to Edge, Whale, and Samsung Browser, is recommended.

## Workflow

```text
       Instantiation
         ∨
       Set License Key
         ∨
       Initialization
         ∨
       Select Product(Earring, Ring, etc.)
         ∨
  ┌─＞ Start
  │      ∨
  │    (Select Product)
  │      ∨
  │    Process
  │      ∨
  │    Render Item
  │      ∨
  │    Render AR
  │      ∨
  └─   Stop
         ∨
       Deinitialization
```

## Library 구성

> All components should be accessible using the same URL.

* stylear_wasm_#.#.#.min.js
* stylear_wasm_#.#.#.wasm
* stylearweblivesdk.#.#.#.min.js

## API

> Refer to the separate API reference document.

## Use StyleAR in PlayCanvas

* Sample code
  * [StyleAR-Watch PlayCanvas Sample](https://playcanvas.com/editor/scene/2032818)

### PlayCanvas Editor

#### Settings

> Required settings for AR rendering

* Settings / External Scripts / `your url/stylearweblivesdk.#.#.#.min.js`
  * `stylear_wasm_#.#.#.min.js` and `stylear_wasm_#.#.#.wasm` files are downloaded and executed using the same URL during the SDK initialization process, so they do not need to be set in External Scripts.
* Settings / Rendering / Skybox / `Empty`
* Settings / Rendering / `Transparent Canvas = true`
  * The background must be rendered transparently to perform AR rendering by compositing with the camera video.
  * The result drawn on the canvas is used for AR rendering.

#### Camera

> Settings for the camera entity required to render items (watch, ring, etc.)

* Hierachy / Camera / `Clear Color = #00000000`
  * The background must be rendered transparently to perform AR rendering by compositing with the camera video.
  
#### Item

> Settings for items (watch, ring, etc.) to be used in AR rendering

##### Watch

* Set the inner diameter of the watch to 2.
* Set the crown of the watch to face up (y-axis positive direction).
* Add an occlusion entity to avoid rendering parts hidden by the wrist.
  * Refer to the Occlusion entity in the sample project.

##### Ring

* Set the inner diameter of the watch to 2.
* Add an occlusion entity to avoid rendering parts hidden by the finger.
  * Refer to the Occlusion entity in the sample project.

### PlayCanvas Editor Code

> Code needed to update the settings for the items and camera used for AR rendering.  
> Code according to the [Workflow](#workflow)

* Instantiation
  * Pass the HTMLCanvasElement from PlayCanvas and the HTMLElement used as the parent for StyleAR as parameters.
  * StyleAR creates a canvas of the same size as the parent element and performs AR rendering on it.
  
  ```javascript
  const styleARWebLive = new window.DPStyleARWebLive(getCanvas(), document.body);

  /**
   *
   * @returns {HTMLCanvasElement} The canvas element that actually shows the rendering result in PlayCanvas
   */
  function getCanvas() {
      const pc = document.getElementById("application-canvas");
      pc.style.visibility = "hidden";
      return pc;
  }
  ```

* Set license key

  ```javascript
  styleARWebLive.license =
  {
      licenseKeys: [
          {
              product: DPStyleARProduct.EARRINGS,
              licenseKey: await (await fetch("lic/earrings.lic")).text()
          },
          {
              product: DPStyleARProduct.EARBUDS,
              licenseKey: await (await fetch("lic/earbuds.lic")).text()
          },
          {
              product: DPStyleARProduct.BRACELET,
              licenseKey: await (await fetch("lic/bracelet.lic")).text()
          },
          {
              product: DPStyleARProduct.EYEWEAR,
              licenseKey: await (await fetch("lic/eyewear.lic")).text()
          },
          {
              product: DPStyleARProduct.FOOTWEAR,
              licenseKey: await (await fetch("lic/footwear.lic")).text()
          },
          {
              product: DPStyleARProduct.HEADWEAR,
              licenseKey: await (await fetch("lic/headwear.lic")).text()
          },
          {
              product: DPStyleARProduct.LIPSTICK,
              licenseKey: await (await fetch("lic/lipstick.lic")).text()
          },
          {
              product: DPStyleARProduct.RING,
              licenseKey: await (await fetch("lic/ring.lic")).text()
          },
          {
              product: DPStyleARProduct.WATCH,
              licenseKey: await (await fetch("lic/watch.lic")).text()
          },
      ]
  }
  ```

* Initialization
  * Pass the parameter that specifies how much to rotate the camera.
  * Since the process of downloading and executing resources needed for initialization takes a long time, a callback function to show the progress can be passed as a parameter.

  ```javascript
  const setting = {
      cameraRotation: 0,
  };

  await styleARWebLive.initialize(setting, {
      onProgress: (p) => console.log(p),
  });
  ```

* Change Product and Start
  * Immediately after initialization, the product is set to DPStyleARProduct.UNKNOWN.
  * Pass the parameter from [DPStyleARProduct](#dpstylearproduct).
  * When start() is called, the camera is activated.
    * If the browser tab becomes inactive after virtual fitting has started, the camera device is deactivated internally, so there's no need to call stop().
    * If the browser tab is reactivated, the camera device is reactivated internally, so there's no need to call start() again.
  
  ```javascript
  await styleARWebLive.changeTo(window.DPStyleARProduct.WATCH);
  await styleARWebLive.start();
  ```

* Update Camera and Item Settings
  * Set the camera settings and transformation values for the items used for virtual fitting.

  ```javascript
  // Grab the camera frame
  await styleARWebLive.grab();

  // Get the data needed for item rendering
  const watchInput = await styleARWebLive.process();

  // If the wrist is not detected
  if (!watchInput || !watchInput.detected) {
    // Disable the item
    showChildren(this.entity, false);

    // Execute AR rendering without the item
    await styleARWebLive.updateItem();
    await styleARWebLive.render();
    return;
  }
  // Enable the item
  showChildren(this.entity, true);

  // Camera settings
  const cameraEntity = this.app.root.findByName("Camera");
  cameraEntity.camera.nearClip = watchInput.near;
  cameraEntity.camera.farClip = watchInput.far;
  cameraEntity.camera.fov = watchInput.verticalFov;
  cameraEntity.camera.aspectRatio = watchInput.aspect;
  cameraEntity.setPosition(0.0, 0.0, 0.0);
  cameraEntity.lookAt(0.0, 0.0, -1.0, 0.0, 1.0, 0.0);

  // Product model settings
  this.entity.setPosition(
      watchInput.translation[0],
      watchInput.translation[1],
      watchInput.translation[2]
  );
  this.entity.setRotation(
      watchInput.quaternion[0],
      watchInput.quaternion[1],
      watchInput.quaternion[2],
      watchInput.quaternion[3]
  );

  // Item rendering
  this.app.render();

  // Execute AR rendering reflecting the results rendered by PlayCanvas
  await styleARWebLive.updateItem();
  await styleARWebLive.render();
  ```

* Stop

  ```javascript
  await styleARWebLive.stop();
  ```

* Deinitialize

  ```javascript
  await styleARWebLive.deinitialize();
  ```
