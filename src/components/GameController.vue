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
    ["Black", "P", [1, 7], "piece bp sqr-17"],
    ["Black", "P", [2, 7], "piece bp sqr-27"],
    ["Black", "P", [3, 7], "piece bp sqr-37"],
    ["Black", "P", [4, 7], "piece bp sqr-47"],
    ["Black", "P", [5, 7], "piece bp sqr-57"],
    ["Black", "P", [6, 7], "piece bp sqr-67"],
    ["Black", "P", [7, 7], "piece bp sqr-77"],
    ["Black", "P", [8, 7], "piece bp sqr-87"],
    ["Black", "N", [2, 8], "piece bn sqr-28"],
    ["Black", "N", [7, 8], "piece bn sqr-78"],
    ["Black", "B", [3, 8], "piece bb sqr-38"],
    ["Black", "B", [6, 8], "piece bb sqr-68"],
    ["Black", "R", [1, 8], "piece br sqr-18"],
    ["Black", "R", [8, 8], "piece br sqr-88"],
    ["Black", "K", [5, 8], "piece bk sqr-58"],
    ["Black", "Q", [4, 8], "piece bq sqr-48"],
    ["White", "P", [1, 2], "piece wp sqr-12"],
    ["White", "P", [2, 2], "piece wp sqr-22"],
    ["White", "P", [3, 2], "piece wp sqr-32"],
    ["White", "P", [4, 2], "piece wp sqr-42"],
    ["White", "P", [5, 2], "piece wp sqr-52"],
    ["White", "P", [6, 2], "piece wp sqr-62"],
    ["White", "P", [7, 2], "piece wp sqr-72"],
    ["White", "P", [8, 2], "piece wp sqr-82"],
    ["White", "N", [2, 1], "piece wn sqr-21"],
    ["White", "N", [7, 1], "piece wn sqr-71"],
    ["White", "B", [3, 1], "piece wb sqr-31"],
    ["White", "B", [6, 1], "piece wb sqr-61"],
    ["White", "R", [1, 1], "piece wr sqr-11"],
    ["White", "R", [8, 1], "piece wr sqr-81"],
    ["White", "K", [5, 1], "piece wk sqr-51"],
    ["White", "Q", [4, 1], "piece wq sqr-41"],
];

type PieceInitData = [
    colour: "White" | "Black",
    type: "P" | "N" | "B" | "R" | "K" | "Q",
    coords: [x: number, y: number],
    classStyle: string
];
type PartialPieceInitData = [PieceInitData[0], PieceInitData[1], PieceInitData[2]];

const buildPieceClass = (
    c: PieceInitData[0], 
    t: PieceInitData[1], 
    coords: PieceInitData[2],
    inverted: boolean
) => `piece ${c.charAt(0).toLowerCase()}${t.toLowerCase()} sqr-${coords[0]}${coords[1]} ${(inverted) ? "inv" : ""}`;

// const invertCoords = (
//     c: PieceInitData[0],
//     t: PieceInitData[1],
//     coords: PieceInitData[2],
//     ..._: any
// ) => {
//     const invertedX = coords[0]; // (coords[0] + (4 - ((coords[0] + 1) / 2))) % 8

//     // const yOffset = (c === "White") 
//     const invertedY = Math.abs(coords[1] + ((c === "White") ? -8 : )); // Math.abs(coords[1] + invertAs)

//     const pawnOffset = (t !== "P") ? 0 : (c === "White") ? 0 : -2;
//     const invertAs = (c === 'White') ? -7 + pawnOffset : 7 + pawnOffset;

//     return [c, t, [invertedX, invertedY]] as PartialPieceInitData;
// };

const buildInitData = (inverted: boolean = false) => {
    const initDataCopy = pieceInitData.slice();
    return initDataCopy.reduce<PieceInitData[]>((acc, data) => {
        const partialData = [data[0], data[1], data[2]] as PartialPieceInitData;
        acc.push([...partialData, buildPieceClass(...[...partialData, inverted])]);
        return acc;
    }, []);
    // return (!inverted) 
    //     ? initDataCopy.reduce<PieceInitData[]>((acc, data) => {
    //         const invertedData = invertCoords(...data);
    //         acc.push([...invertedData, buildPieceClass(...invertedData)]);
    //         return acc;
    //     }, [])
    //     : initDataCopy.reduce<PieceInitData[]>((acc, data) => {
    //         const partialData = [data[0], data[1], data[2]] as const;
    //         acc.push([...partialData, buildPieceClass(...partialData)]);
    //         return acc;
    //     }, []);
};

import { ref, useTemplateRef, watch } from 'vue';

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
const loadGameBoard = (inverted: boolean = false) => {
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
        // USE && inverted
        ctx.fillStyle = ((j + i) % 2 === 0) ? "White" : "Black";
        // console.log(`Current Style @ ${i}${String.fromCharCode(65 + j)}: ${ctx.fillStyle}`);
        ctx.fillRect(j * w, i * h, w, h);
        // const pieceDetails = drawPiece(j, i);
        // if (!pieceDetails && j !== 0 && i !== 7) continue;
        if (j !== 0 && i !== 7) continue;
        ctx.fillStyle = (ctx.fillStyle === "#ffffff") ? "Black": "White";
        ctx.font = "bold 1.8rem serif";
        if (j === 0) ctx.fillText(`${(!inverted) ? Math.abs(i - 8) : i + 1}`, qw * 0.5 + (j * w), qh * 0.9 + (i * h), qw);
        if (i === 7) ctx.fillText(`${String.fromCharCode(65 + ((!inverted) ? j : Math.abs(j - 7)))}`, qw * 3.0 + (j * w), qh * 3.5 + (i * h), qw);
        // if (!pieceDetails) continue;
        // ctx.font = "bold 20px serif";
        // const pTxt = `${pieceDetails.c.split("")[0]}:${pieceDetails.t}`;
        // ctx.fillText(pTxt, qw * 2 + (j * w), qh * 2 + (i * h), qw);
    }
};

const isInverted = ref<boolean>(false);

watch(isInverted, (newInv, oldInv) => {
    if (newInv === oldInv) return;
    loadGameBoard(newInv);
});

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
    <button @click="loadGameBoard(isInverted)">Load Board!</button>
    <button @click="isInverted = !isInverted">Switch Team! Current Team: {{ !isInverted ? "White" : "Black" }}</button>
    <div class="game-cont">
        <canvas ref="game-board"></canvas> <!-- height="500" width="500" -->
        <chess-piece v-for="[c, t, coords, style] in buildInitData(isInverted)"
            :init-colour="c"
            :init-type="t"
            :init-x=coords[0]
            :init-y=coords[1]
            :init-inverted=isInverted
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