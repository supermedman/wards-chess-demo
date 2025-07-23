<!--
/**
 * Replicatable Piece Instance
 * 
 *  - Can be any chess piece 
 *  - Maintains reference to "side" (black/white)
 *  - ?? Maintains reference to current position ??
 * 
 *  Extras Needed
 * 
 *  - Scripts for base movement rules
 */
-->
<script setup lang="ts">
import { ref } from 'vue';

type PieceInfo = {
    colour: "White" | "Black";
    type: "P" | "B" | "N" | "R" | "K" | "Q";
    x: number;
    y: number;
    // style: string;
    // coords: [x: number, y: number];
};
/** @vue-ignore */
type AddInit<Type extends { [k: string]: any }> = { [Key in keyof Type as `init${Capitalize<Key & string>}`]: Type[Key] };
type InitPieceInfo = Partial<AddInit<PieceInfo>>;
// type InitPieceInfo = AddInit<PieceInfo>;
const { 
    initColour = "Black", 
    initType = "P", 
    // @ts-ignore
    initX = 0, 
    // @ts-ignore
    initY = 0 
} = defineProps<InitPieceInfo>();
const colour = ref<PieceInfo["colour"]>("Black"), 
    type = ref<PieceInfo["type"]>("P"), 
    x = ref<PieceInfo["x"]>(0),
    y = ref<PieceInfo["y"]>(0);

// const coords = [x, y] as const;

const notatePosition = () => `${String.fromCharCode(65 + Math.abs(x.value - 7))}${y.value + 1}`;
// @ts-ignore
const buildStyleClass = () => `piece ${colour.value.substring(0, 1).toLowerCase()}${type.value.toLowerCase()} sqr-${x.value + 1}${y.value + 1}`;
const typeIndex = ["P", "B", "N", "R", "K", "Q"] as const;
</script>

<template>
    <div>
        <!-- {{ initColour }} {{ initType }} [{{ initX }}, {{ initY }}] -->
        <!-- <div :class="initStyle"></div> -->
        <h1>Piece Details</h1>
        <div>
            <p>
                Colour: <button @click="colour = colour === 'White' ? 'Black' : 'White'">{{ colour ?? initColour }}</button> 
                || 
                Type: <button @click="type = typeIndex.indexOf(type) === 5 ? typeIndex[0] : typeIndex[typeIndex.indexOf(type) + 1]">{{ type ?? initType }}</button>
                ||
                Position: <button>{{ notatePosition() }}</button>
            </p>
        </div>
    </div>
</template>

<style scoped>
.piece {
    background-size: 100%;
    height: 12.5%;
    width: 12.5%;
    left: 0;
    top: 0;
    position: absolute;
    will-change: transform;
}

.piece.wp { background-image: url("../assets/wp.png"); }
.piece.wb { background-image: url("../assets/wb.png"); }
.piece.wn { background-image: url("../assets/wn.png"); }
.piece.wr { background-image: url("../assets/wr.png"); }
.piece.wk { background-image: url("../assets/wk.png"); }
.piece.wq { background-image: url("../assets/wq.png"); }
.piece.bp { background-image: url("../assets/bp.png"); }
.piece.bb { background-image: url("../assets/bb.png"); }
.piece.bn { background-image: url("../assets/bn.png"); }
.piece.br { background-image: url("../assets/br.png"); }
.piece.bk { background-image: url("../assets/bk.png"); }
.piece.bq { background-image: url("../assets/bq.png"); }

.sqr-11 { transform: translateY(700%); }
.sqr-21 { transform: translate(100%, 700%); }
.sqr-31 { transform: translate(200%, 700%); }
.sqr-41 { transform: translate(300%, 700%); }
.sqr-51 { transform: translate(400%, 700%); }
.sqr-61 { transform: translate(500%, 700%); }
.sqr-71 { transform: translate(600%, 700%); }
.sqr-81 { transform: translate(700%, 700%); }

</style>