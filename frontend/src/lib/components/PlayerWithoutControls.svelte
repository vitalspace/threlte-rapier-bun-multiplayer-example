<script lang="ts">
  import type { RigidBody as RapierRigidBody } from "@dimforge/rapier3d-compat";
  import { T, useTask } from "@threlte/core";
  import { Outlines, useDraco, useGltf, HTML } from "@threlte/extras";
  import { Group, PerspectiveCamera, Vector3 } from "three";
  import { AutoColliders, RigidBody } from "@threlte/rapier";

  let { player } = $props<{
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
  let rigidBody = $state<RapierRigidBody | undefined>(undefined);

  let position: [number, number, number] = $state([
    player.x,
    player.y,
    player.z,
  ]);

  useTask(() => {
    if (!rigidBody || !objectRef || !mainGroupRef) return;
    position = [player.x, player.y, player.z];
    rigidBody.setTranslation({ x: player.x, y: player.y, z: player.z }, true);
    rigidBody.setRotation(
      { x: player.r._x, y: player.r._y, z: player.r._z, w: player.r._w },
      true
    );
  });
</script>

<T.Group {position} bind:ref={mainGroupRef}>
  <HTML position={[0, 1, 0]}>
    <span class="text-white">{player.playerId}</span>
  </HTML>
  <RigidBody bind:rigidBody type="dynamic">
    <AutoColliders shape="cuboid">
      <T.Mesh bind:ref={objectRef}>
        <T.BoxGeometry args={[1, 1, 1]} />
        <T.MeshStandardMaterial color="green" />
      </T.Mesh>
    </AutoColliders>
  </RigidBody>
</T.Group>