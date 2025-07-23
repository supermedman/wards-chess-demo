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
};
/** @vue-ignore */
type AddInit<Type extends { [k: string]: any }> = { [Key in keyof Type as `init${Capitalize<Key & string>}`]: Type[Key] };
type InitPieceInfo = Partial<AddInit<PieceInfo>>;
// type InitPieceInfo = AddInit<PieceInfo>;
// @ts-ignore
// const { 
//     // @ts-ignore
//     initColour = "Black", 
//     // @ts-ignore
//     initType = "P", 
//     // @ts-ignore
//     initX = 0, 
//     // @ts-ignore
//     initY = 0 
// } = 
defineProps<InitPieceInfo>();
const colour = ref<PieceInfo["colour"]>("Black"), 
    type = ref<PieceInfo["type"]>("P"), 
    x = ref<PieceInfo["x"]>(0),
    y = ref<PieceInfo["y"]>(0);

// @ts-ignore
const hasMoved = ref<boolean>(false);

// const coords = [x, y] as const;
// @ts-ignore
const notatePosition = () => `${String.fromCharCode(65 + Math.abs(x.value - 7))}${y.value + 1}`;
// @ts-ignore
const buildStyleClass = () => `piece ${colour.value.substring(0, 1).toLowerCase()}${type.value.toLowerCase()} sqr-${x.value + 1}${y.value + 1}`;
// @ts-ignore
const typeIndex = ["P", "B", "N", "R", "K", "Q"] as const;

const yInvert = (team: PieceInfo["colour"], mag: number/**, isFlipped: boolean = false */) => {
    if (team === "Black") return mag * -1;
    else return mag * +1;

    // switch (isFlipped) {
    //     // POV Black
    //     case true:
    //         if (team === "Black") return +1
    //         else return
    //     break;
    //     // POV White
    //     case false:

    //     break;
    // }
};

const ruleSet = {
    "P": {
        "move": [
            { x: x.value, y: y.value + yInvert(colour.value, 1) }
        ],
        "attack": [
            { x: x.value + 1, y: y.value + yInvert(colour.value, 1) },
            { x: x.value - 1, y: y.value + yInvert(colour.value, 1) }
        ],
        "special": [],
    },
    "N": {
        /**      8   1
         *       < ^ >
         *         ^
         * 7 ^     ^     ^ 2
         *   < < < N > > >
         * 6 v     v     v 3
         *         v
         *       < v > 
         *       5   4
         */
        "move": [
            // 1
            { x: x.value + 1, y: y.value + 3 },
            // 2
            { x: x.value + 3, y: y.value + 1 },
            // 3
            { x: x.value + 3, y: y.value - 1 },
            // 4
            { x: x.value + 1, y: y.value - 3 },
            // 5
            { x: x.value - 1, y: y.value - 3 },
            // 6
            { x: x.value - 3, y: y.value - 1 },
            // 7
            { x: x.value - 3, y: y.value + 1 },
            // 8
            { x: x.value - 1, y: y.value + 3 },
        ],
        "attack": [],
        "special": [],
    },
    "B": {
        "move": [],
        "attack": [],
        "special": []
    },
    "R": {
        "move": [],
        "attack": [],
        "special": []
    },
    "K": {
        "move": [],
        "attack": [],
        "special": []
    },
    "Q": {
        "move": [],
        "attack": [],
        "special": []
    }
};

const displayRules = (team: PieceInfo['colour'], pieceType: PieceInfo['type']) => {
    console.log("Team Selected: ", team);
    console.log("Moveset for selected piece: ", ...ruleSet[pieceType]['move']);
};
</script>

<template>
    <div @click="displayRules($props.initColour ?? 'Black', $props.initType ?? 'P')">
        <!-- 
            Emit Event??
            Build moveset map overlay?
            TBD WIP
        -->
        <!-- <div v-if="!$props.initX && !$props.initY"> -->
            <!-- {{ initColour }} {{ initType }} [{{ initX }}, {{ initY }}] -->
            <!-- <div :class="initStyle"></div> -->
            <!-- <h1>Piece Details</h1>
            <div>
                <p>
                    Colour: <button @click="colour = colour === 'White' ? 'Black' : 'White'">{{ colour ?? initColour }}</button> 
                    || 
                    Type: <button @click="type = typeIndex.indexOf(type) === 5 ? typeIndex[0] : typeIndex[typeIndex.indexOf(type) + 1]">{{ type ?? initType }}</button>
                    ||
                    Position: <button>{{ notatePosition() }}</button>
                </p>
            </div>
        </div> -->
    </div>
</template>

<style scoped>
.piece {
    background-size: 100%;
    overflow: hidden;
    height: 12.5%;
    width: 12.5%;
    left: 0;
    top: 0;
    position: absolute;
    will-change: transform;
}

.piece:hover {
    cursor: -webkit-grab;
}

.piece:active {
    cursor: pointer;
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

.sqr-12 { transform: translateY(600%); }
.sqr-22 { transform: translate(100%, 600%); }
.sqr-32 { transform: translate(200%, 600%); }
.sqr-42 { transform: translate(300%, 600%); }
.sqr-52 { transform: translate(400%, 600%); }
.sqr-62 { transform: translate(500%, 600%); }
.sqr-72 { transform: translate(600%, 600%); }
.sqr-82 { transform: translate(700%, 600%); }

.sqr-17 { transform: translateY(100%); }
.sqr-27 { transform: translate(100%, 100%); }
.sqr-37 { transform: translate(200%, 100%); }
.sqr-47 { transform: translate(300%, 100%); }
.sqr-57 { transform: translate(400%, 100%); }
.sqr-67 { transform: translate(500%, 100%); }
.sqr-77 { transform: translate(600%, 100%); }
.sqr-87 { transform: translate(700%, 100%); }

.sqr-18 { transform: translate(); }
.sqr-28 { transform: translate(100%); }
.sqr-38 { transform: translate(200%); }
.sqr-48 { transform: translate(300%); }
.sqr-58 { transform: translate(400%); }
.sqr-68 { transform: translate(500%); }
.sqr-78 { transform: translate(600%); }
.sqr-88 { transform: translate(700%); }

</style>