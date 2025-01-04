<script lang="ts">
  import { T } from "@threlte/core";
  import { AutoColliders } from "@threlte/rapier";
</script>

<T.Group position={[0, -1, 0]}>
  <AutoColliders shape="cuboid">
    <T.Mesh receiveShadow visible={false}>
      <T.BoxGeometry args={[20, 1, 20]} />
      <T.MeshStandardMaterial />
    </T.Mesh>
  </AutoColliders>
</T.Group>
