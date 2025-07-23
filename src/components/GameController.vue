<script setup lang="ts">
// @ts-ignore
// @ts-ignore
import ChessPiece from './ChessPiece.vue';
//@ts-ignore
const pieceInitData: [
    colour: "Black" | "White",
    type: "P" | "N" | "B" | "R" | "K" | "Q",
    coords: [x: number, y: number],
    classStyle: string
][] = [
    ["White", "P", [0, 6], "piece wp sqr-17"],
    ["White", "P", [1, 6], "piece wp sqr-27"],
    ["White", "P", [2, 6], "piece wp sqr-37"],
    ["White", "P", [3, 6], "piece wp sqr-47"],
    ["White", "P", [4, 6], "piece wp sqr-57"],
    ["White", "P", [5, 6], "piece wp sqr-67"],
    ["White", "P", [6, 6], "piece wp sqr-77"],
    ["White", "P", [7, 6], "piece wp sqr-87"],
    ["White", "N", [1, 7], "piece wn sqr-28"],
    ["White", "N", [6, 7], "piece wn sqr-78"],
    ["White", "B", [2, 7], "piece wb sqr-38"],
    ["White", "B", [5, 7], "piece wb sqr-68"],
    ["White", "R", [0, 7], "piece wr sqr-18"],
    ["White", "R", [7, 7], "piece wr sqr-88"],
    ["White", "K", [4, 7], "piece wk sqr-58"],
    ["White", "Q", [3, 7], "piece wq sqr-48"],
    ["Black", "P", [0, 2], "piece bp sqr-12"],
    ["Black", "P", [1, 2], "piece bp sqr-22"],
    ["Black", "P", [2, 2], "piece bp sqr-32"],
    ["Black", "P", [3, 2], "piece bp sqr-42"],
    ["Black", "P", [4, 2], "piece bp sqr-52"],
    ["Black", "P", [5, 2], "piece bp sqr-62"],
    ["Black", "P", [6, 2], "piece bp sqr-72"],
    ["Black", "P", [7, 2], "piece bp sqr-82"],
    ["Black", "N", [1, 0], "piece bn sqr-21"],
    ["Black", "N", [6, 0], "piece bn sqr-71"],
    ["Black", "B", [2, 0], "piece bb sqr-31"],
    ["Black", "B", [5, 0], "piece bb sqr-61"],
    ["Black", "R", [0, 0], "piece br sqr-11"],
    ["Black", "R", [7, 0], "piece br sqr-81"],
    ["Black", "K", [4, 0], "piece bk sqr-51"],
    ["Black", "Q", [3, 0], "piece bq sqr-41"],
];

import { useTemplateRef } from 'vue';

const canvasBoard = useTemplateRef<HTMLCanvasElement>("game-board");

const initCanvas = () => {
    const cRsp = canvasBoard.value;
    if (!cRsp) throw new Error("[WARNING]: Failed to locate canvas!");

    // const compStyles = getComputedStyle(cRsp);
    // console.log(compStyles);

    cRsp.width = cRsp.clientWidth;
    cRsp.height = cRsp.clientHeight;

    const ctxRsp = canvasBoard.value?.getContext('2d');
    if (!ctxRsp || !canvasBoard.value) throw new Error("[WARNING]: Failed to locate canvas ctx!");
    return { 
        /** Canvas Context */
        ctx: ctxRsp, 
        /** Canvas Element */
        c: cRsp
    };
};
const loadGameBoard = () => {
    const { ctx, c } = initCanvas();
    // 8 x 8
    const DIM = 8;
    // Height & Width per individual grid cell
    const h = c.height / DIM, w = c.width / DIM;
    // Quarter Dimensions for text placement
    const qh = h / 4, qw = w / 4;
    /**
     * i @ 7 = Row 1
     * j @ 7 = File H
     * 
     * x = 7, y = 7 === H1
     */
    for (let i = 0; i < DIM; i++) for (let j = 0; j < DIM; j++) {
        // Alternating colours
        ctx.fillStyle = ((j + i) % 2 === 0) ? "White" : "Black";
        // console.log(`Current Style @ ${i}${String.fromCharCode(65 + j)}: ${ctx.fillStyle}`);
        ctx.fillRect(j * w, i * h, w, h);
        // const pieceDetails = drawPiece(j, i);
        // if (!pieceDetails && j !== 0 && i !== 7) continue;
        if (j !== 0 && i !== 7) continue;
        ctx.fillStyle = (ctx.fillStyle === "#ffffff") ? "Black": "White";
        ctx.font = "bold 1.8rem serif";
        if (j === 0) ctx.fillText(`${Math.abs(i - 8)}`, qw * 0.5 + (j * w), qh * 0.9 + (i * h), qw);
        if (i === 7) ctx.fillText(`${String.fromCharCode(65 + j)}`, qw * 3.0 + (j * w), qh * 3.5 + (i * h), qw);
        // if (!pieceDetails) continue;
        // ctx.font = "bold 20px serif";
        // const pTxt = `${pieceDetails.c.split("")[0]}:${pieceDetails.t}`;
        // ctx.fillText(pTxt, qw * 2 + (j * w), qh * 2 + (i * h), qw);
    }
};

// const isStartingPosition = (coords: [x: number, y: number]) => coords[1] > 3 || coords[1] < 5;
// const drawPiece = (x: number, y: number) => {
//     if (!isStartingPosition([x, y] as [x: number, y: number])) return;
//     const checkPieceType = (file: number, c: "White" | "Black") => {
//         switch (file) {
//             case 0: case 7: return { c, t: "R" };
//             case 1: case 6: return { c, t: "N" };
//             case 2: case 5: return { c, t: "B" };
//             case 3: return { c, t: "Q" };
//             case 4: return { c, t: "K" };
//         }
//     };
//     switch (y) {
//         case 0: return checkPieceType(x, "Black");
//         // Black Pawn
//         case 1: return { c: "Black", t: "P" };
//         // White Pawn
//         case 6: return { c: "White", t: "P" };
//         case 7: return checkPieceType(x, "White");
//         default: return;
//     }
// }
</script>

<template>
    <button @click="loadGameBoard">Load Board!</button>
    <div class="game-cont">
        <canvas ref="game-board"></canvas> <!-- height="500" width="500" -->
        <chess-piece v-for="[c, t, coords, style] in pieceInitData"
            :init-colour="c"
            :init-type="t"
            :init-x=coords[0]
            :init-y=coords[1]
            :class=style
        ></chess-piece>
    </div>
<!-- <chess-piece init-colour="Black" init-type="B"></chess-piece> -->
<!-- <chess-piece init-colour="White" init-type="P" :init-x=0 :init-y=6 class="piece wp sqr-06"></chess-piece>
    <chess-piece init-colour="White" init-type="P" :init-x=1 :init-y=6 class="piece wp sqr-16"></chess-piece>
    <chess-piece init-colour="White" init-type="P" :init-x=2 :init-y=6 class="piece wp sqr-26"></chess-piece>
    <chess-piece init-colour="White" init-type="P" :init-x=3 :init-y=6 class="piece wp sqr-36"></chess-piece>
    <chess-piece init-colour="White" init-type="P" :init-x=4 :init-y=6 class="piece wp sqr-46"></chess-piece>
    <chess-piece init-colour="White" init-type="P" :init-x=5 :init-y=6 class="piece wp sqr-56"></chess-piece>
    <chess-piece init-colour="White" init-type="P" :init-x=6 :init-y=6 class="piece wp sqr-66"></chess-piece>
    <chess-piece init-colour="White" init-type="P" :init-x=7 :init-y=6 class="piece wp sqr-76"></chess-piece>
    <chess-piece init-colour="White" init-type="N" :init-x=1 :init-y=7 class="piece wn sqr-17"></chess-piece>
    <chess-piece init-colour="White" init-type="N" :init-x=6 :init-y=7 class="piece wn sqr-67"></chess-piece>
    <chess-piece init-colour="White" init-type="B" :init-x=2 :init-y=7 class="piece wb sqr-27"></chess-piece>
    <chess-piece init-colour="White" init-type="B" :init-x=5 :init-y=7 class="piece wb sqr-57"></chess-piece>
    <chess-piece init-colour="White" init-type="R" :init-x=0 :init-y=7 class="piece wr sqr-07"></chess-piece>
    <chess-piece init-colour="White" init-type="R" :init-x=7 :init-y=7 class="piece wr sqr-77"></chess-piece>
    <chess-piece init-colour="White" init-type="K" :init-x=4 :init-y=7 class="piece wk sqr-47"></chess-piece>
    <chess-piece init-colour="White" init-type="Q" :init-x=3 :init-y=7 class="piece wq sqr-37"></chess-piece>

    <chess-piece init-colour="Black" init-type="P" :init-x=0 :init-y=2 class="piece bp sqr-02"></chess-piece>
    <chess-piece init-colour="Black" init-type="P" :init-x=1 :init-y=2 class="piece bp sqr-12"></chess-piece>
    <chess-piece init-colour="Black" init-type="P" :init-x=2 :init-y=2 class="piece bp sqr-22"></chess-piece>
    <chess-piece init-colour="Black" init-type="P" :init-x=3 :init-y=2 class="piece bp sqr-32"></chess-piece>
    <chess-piece init-colour="Black" init-type="P" :init-x=4 :init-y=2 class="piece bp sqr-42"></chess-piece>
    <chess-piece init-colour="Black" init-type="P" :init-x=5 :init-y=2 class="piece bp sqr-52"></chess-piece>
    <chess-piece init-colour="Black" init-type="P" :init-x=6 :init-y=2 class="piece bp sqr-62"></chess-piece>
    <chess-piece init-colour="Black" init-type="P" :init-x=7 :init-y=2 class="piece bp sqr-72"></chess-piece>
    <chess-piece init-colour="Black" init-type="N" :init-x=1 :init-y=0 class="piece bn sqr-10"></chess-piece>
    <chess-piece init-colour="Black" init-type="N" :init-x=6 :init-y=0 class="piece bn sqr-60"></chess-piece>
    <chess-piece init-colour="Black" init-type="B" :init-x=2 :init-y=0 class="piece bb sqr-20"></chess-piece>
    <chess-piece init-colour="Black" init-type="B" :init-x=5 :init-y=0 class="piece bb sqr-50"></chess-piece>
    <chess-piece init-colour="Black" init-type="R" :init-x=0 :init-y=0 class="piece br sqr-00"></chess-piece>
    <chess-piece init-colour="Black" init-type="R" :init-x=7 :init-y=0 class="piece br sqr-70"></chess-piece>
    <chess-piece init-colour="Black" init-type="K" :init-x=4 :init-y=0 class="piece bk sqr-40"></chess-piece>
    <chess-piece init-colour="Black" init-type="Q" :init-x=3 :init-y=0 class="piece bq sqr-30"></chess-piece> -->

    <!-- <chess-piece class="piece wp sqr-06"></chess-piece>
    <chess-piece class="piece wp sqr-16"></chess-piece>
    <chess-piece class="piece wp sqr-26"></chess-piece>
    <chess-piece class="piece wp sqr-36"></chess-piece>
    <chess-piece class="piece wp sqr-46"></chess-piece>
    <chess-piece class="piece wp sqr-56"></chess-piece>
    <chess-piece class="piece wp sqr-66"></chess-piece>
    <chess-piece class="piece wp sqr-76"></chess-piece>
    <chess-piece class="piece wn sqr-17"></chess-piece>
    <chess-piece class="piece wn sqr-67"></chess-piece>
    <chess-piece class="piece wb sqr-27"></chess-piece>
    <chess-piece class="piece wb sqr-57"></chess-piece>
    <chess-piece class="piece wr sqr-07"></chess-piece>
    <chess-piece class="piece wr sqr-77"></chess-piece>
    <chess-piece class="piece wk sqr-47"></chess-piece>
    <chess-piece class="piece wq sqr-37"></chess-piece>
    <chess-piece class="piece bp sqr-02"></chess-piece>
    <chess-piece class="piece bp sqr-12"></chess-piece>
    <chess-piece class="piece bp sqr-22"></chess-piece>
    <chess-piece class="piece bp sqr-32"></chess-piece>
    <chess-piece class="piece bp sqr-42"></chess-piece>
    <chess-piece class="piece bp sqr-52"></chess-piece>
    <chess-piece class="piece bp sqr-62"></chess-piece>
    <chess-piece class="piece bp sqr-72"></chess-piece>
    <chess-piece class="piece bn sqr-10"></chess-piece>
    <chess-piece class="piece bn sqr-60"></chess-piece>
    <chess-piece class="piece bb sqr-20"></chess-piece>
    <chess-piece class="piece bb sqr-50"></chess-piece>
    <chess-piece class="piece br sqr-00"></chess-piece>
    <chess-piece class="piece br sqr-70"></chess-piece>
    <chess-piece class="piece bk sqr-40"></chess-piece>
    <chess-piece class="piece bq sqr-30"></chess-piece> -->
</template>

<style scoped>
.game-cont {
    width: var(--boardWidth);
    height: var(--boardHeight);
    position: relative;
    box-sizing: inherit;
    background-repeat: no-repeat;
    padding-bottom: 0;
}

canvas {
    /* border: 1px solid red; */
    position: relative;
    width: var(--boardWidth);
    height: var(--boardHeight);
}
</style>