<script lang="ts">
  import type { RigidBody as RapierRigidBody } from "@dimforge/rapier3d-compat";
  import { T, useTask } from "@threlte/core";

  import { Outlines, useDraco, useGltf, HTML } from "@threlte/extras";
  import { Group, PerspectiveCamera, Vector3 } from "three";

  import { PointerDrag } from "../misc/pointerDrag";
  import { PointerLock } from "../misc/pointerLock";
  import { ThirdPersonControls } from "../misc/thirdPersonControls";
  import { AutoColliders, RigidBody } from "@threlte/rapier";

  let { socket, player } = $props<{
    socket: WebSocket;
    player: {
      playerId: string;
      x: number;
      y: number;
      z: number;
      r: number;
    };
  }>();

  let mainGroupRef = $state<Group | undefined>(undefined);
  let objectRef = $state<any>(undefined);
  let cameraRef = $state<PerspectiveCamera | undefined>(undefined);
  let rigidBody = $state<RapierRigidBody | undefined>(undefined);

  let position: [number, number, number] = $state([
    player.x,
    player.y,
    player.z,
  ]);

  let controls: ThirdPersonControls | undefined;
  $effect(() => {
    if (objectRef && cameraRef && mainGroupRef) {
      const isTouchDevice = "ontouchstart" in window;
      const canvas = document.querySelector("canvas");

      controls = new ThirdPersonControls(cameraRef, mainGroupRef, {
        offset: new Vector3(0, 1, 0),
        targetRadius: 5,
        interpolationFactor: 0.05,
      });
      controls.theta = 90;

      if (!isTouchDevice && canvas) {
        let pl = new PointerLock(canvas);
        let pd = new PointerDrag(canvas);
        pd.onMove((delta) => {
          if (pl.isLocked()) {
            controls?.update(delta.x * 2, delta.y * 2);
          }
        });
      }
    }
  });

  let speed = 6;
  const keys = { w: { pressed: false } };

  const press = (e: KeyboardEvent, isPressed: boolean) => {
    e.preventDefault();
    if (e.keyCode === 87) keys.w.pressed = isPressed;
  };

  window.addEventListener("keydown", (e) => press(e, true));
  window.addEventListener("keyup", (e) => press(e, false));

  const v3 = new Vector3();

  useTask(() => {
    if (!rigidBody || !objectRef || !cameraRef || !controls || !mainGroupRef)
      return;

    const rotation = cameraRef.getWorldDirection(v3);
    const theta = Math.atan2(rotation.x, rotation.z);
    const rotationObj = objectRef.getWorldDirection(v3);
    const thetaObject = Math.atan2(rotationObj.x, rotationObj.z);

    rigidBody.setAngvel({ x: 0, y: 0, z: 0 }, true);

    const angleDiff = Math.abs(theta - thetaObject);
    const threshold = Math.PI / 24;

    if (angleDiff > threshold) {
      let rotationSpeed = 4;
      if (angleDiff > Math.PI - threshold) rotationSpeed *= -1;
      if (theta < thetaObject) rotationSpeed *= -1;
      rigidBody.setAngvel({ x: 0, y: rotationSpeed, z: 0 }, true);
    }

    const pos = rigidBody.translation();
    const rot = rigidBody.rotation();

    position = [pos.x, pos.y, pos.z];

    if (keys.w.pressed) {
      const x = Math.sin(theta) * speed;
      const z = Math.cos(theta) * speed;
      const y = pos.y * speed;

      rigidBody.setLinvel({ x, y: y, z }, true);
    }
    controls.update(0, 0);

    const currentPosition = {
      x: pos.x,
      y: pos.y,
      z: pos.z,
      r: {
        _x: rot.x,
        _y: rot.y,
        _z: rot.z,
        _w: rot.w,
      },
    };

    socket.send(
      JSON.stringify({ type: "playerMovement", position: currentPosition })
    );
  });
</script>

<T.PerspectiveCamera makeDefault bind:ref={cameraRef}></T.PerspectiveCamera>

<T.Group {position} bind:ref={mainGroupRef}>
  <RigidBody bind:rigidBody>
    <AutoColliders shape="cuboid">
      <T.Mesh bind:ref={objectRef}>
        <T.BoxGeometry args={[1, 1, 1]} />
        <T.MeshStandardMaterial color="green" />
      </T.Mesh>
    </AutoColliders>
  </RigidBody>
</T.Group>
