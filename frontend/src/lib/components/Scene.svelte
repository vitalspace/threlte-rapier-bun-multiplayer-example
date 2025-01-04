<script lang="ts">
  import { T } from "@threlte/core";
  import { ContactShadows, Float, Grid, OrbitControls } from "@threlte/extras";
  let socket: WebSocket = new WebSocket("ws://localhost:3000");
  import PlayerCube from "./Player.svelte";
  import PlayerWithoutControls from "./PlayerWithoutControls.svelte";
  import Ground from "./Ground.svelte";

  type Player = {
    playerId: string;
    x: number;
    y: number;
    z: number;
    r: number;
  };

  let pId = $state({});
  let player = $state<Player | null>(null);
  let otherPlayer = $state({});
  let isConnected = $state(false);

  let playerList = $state<Player[]>([]);

  socket.addEventListener("open", () => {
    console.log("Connected to server");
    isConnected = true;
  });

  socket.addEventListener("message", (event) => {
    const message = JSON.parse(event.data);

    switch (message.type) {
      case "playerId": {
        pId = message.id;
        break;
      }

      case "currentPlayers": {
        Object.keys(message.players).forEach((id) => {
          const { playerId, x, y, z, r } = message.players[id];
          if (playerId === pId) {
            player = { playerId, x, y, z, r };
          } else {
            playerList.push({ playerId, x, y, z, r });
          }
        });
        break;
      }

      case "newPlayer": {
        const { playerId, x, y, z, r } = message.player;
        playerList.push({ playerId, x, y, z, r });
        break;
      }

      case "playerMoved": {
        playerList.forEach((player) => {
          if (message.playerInfo.playerId === player.playerId) {
            // console.log(message.playerInfo.r);
            player.x = message.playerInfo.x;
            player.y = message.playerInfo.y;
            player.z = message.playerInfo.z;
            player.r = message.playerInfo.r;
          }
        });
        break;
      }

      case "disconnect": {
        playerList = playerList.filter(
          (player) => player.playerId !== message.playerId
        );
        break;
      }
    }
  });
</script>

<T.DirectionalLight intensity={0.8} position.x={5} position.y={10} />
<T.AmbientLight intensity={0.2} />
<ContactShadows scale={10} blur={2} far={2.5} opacity={0.5} />

<Ground />
{#if isConnected && player}
  {#if playerList.length > 0}
    {#each playerList as playerInfo}
      <PlayerWithoutControls player={playerInfo} />
    {/each}
  {/if}

  <PlayerCube {socket} {player} />
{/if}
