<script setup lang="ts">
// import { useTemplateRef } from 'vue';

// const canvasBoard = useTemplateRef<HTMLCanvasElement>("game-board");

// const initCanvas = () => {
//     const ctxRsp = canvasBoard.value?.getContext('2d');
//     if (!ctxRsp || !canvasBoard.value) throw new Error("[WARNING]: Failed to locate canvas ctx!");
//     return { 
//         /** Canvas Context */
//         ctx: ctxRsp, 
//         /** Canvas Element */
//         c: canvasBoard.value 
//     };
// };
// const loadGameBoard = () => {
//     const { ctx, c } = initCanvas();
//     // 8 x 8
//     const DIM = 8;
//     // Height & Width per individual grid cell
//     const h = c.height / DIM, w = c.width / DIM;
//     // Quarter Dimensions for text placement
//     const qh = h / 4, qw = w / 4;
//     /**
//      * i @ 7 = Row 1
//      * j @ 7 = File H
//      * 
//      * x = 7, y = 7 === H1
//      */
//     for (let i = 0; i < DIM; i++) for (let j = 0; j < DIM; j++) {
//         // Alternating colours
//         ctx.fillStyle = ((j + i) % 2 === 0) ? "White" : "Black";
//         // console.log(`Current Style @ ${i}${String.fromCharCode(65 + j)}: ${ctx.fillStyle}`);
//         ctx.fillRect(j * w, i * h, w, h);
//         const pieceDetails = drawPiece(j, i);
//         if (!pieceDetails && j !== 0 && i !== 7) continue;
//         ctx.fillStyle = (ctx.fillStyle === "#ffffff") ? "Black": "White";
//         ctx.font = "bold 15px serif";
//         if (j === 0) ctx.fillText(`${Math.abs(i - 8)}`, qw * 0.5 + (j * w), qh * 0.9 + (i * h), qw);
//         if (i === 7) ctx.fillText(`${String.fromCharCode(65 + j)}`, qw * 2.5 + (j * w), qh * 3.5 + (i * h), qw);
//         if (!pieceDetails) continue;
//         ctx.font = "bold 20px serif";
//         const pTxt = `${pieceDetails.c.split("")[0]}:${pieceDetails.t}`;
//         ctx.fillText(pTxt, qw * 2 + (j * w), qh * 2 + (i * h), qw);
//     }
// };
/**
 * Starting Position Mapping
 * 
 * Pawns - 2 & 7
 * Knights - B1, G1 & B8, G8
 * Bishops - C1, F1 & C8, F8
 * Rooks - A1, H1 & A8, H8
 * Queen - D1 & D8
 * King - E1 & E8
 * ```
 * y v
 * 0  | A8 | B8 | C8 | D8 | E8 | F8 | G8 | H8 |
 * 1  | A7 | B7 | C7 | D7 | E7 | F7 | G7 | H7 |
 * 2  | A6 | B6 | C6 | D6 | E6 | F6 | G6 | H6 |
 * 3  | A5 | B5 | C5 | D5 | E5 | F5 | G5 | H5 |
 * 4  | A4 | B4 | C4 | D4 | E4 | F4 | G4 | H4 |
 * 5  | A3 | B3 | C3 | D3 | E3 | F3 | G3 | H3 |
 * 6  | A2 | B2 | C2 | D2 | E2 | F2 | G2 | H2 |
 * 7  | A1 | B1 | C1 | D1 | E1 | F1 | G1 | H1 |
 * x >  0    1    2    3    4    5    6    7
 * ```
 */
//@ts-ignore
const pieceInitData: [
    colour: "Black" | "White",
    type: "P" | "N" | "B" | "R" | "K" | "Q",
    coords: [x: number, y: number]
][] = [
    ["White", "P", [0, 6]],
    ["White", "P", [1, 6]],
    ["White", "P", [2, 6]],
    ["White", "P", [3, 6]],
    ["White", "P", [4, 6]],
    ["White", "P", [5, 6]],
    ["White", "P", [6, 6]],
    ["White", "P", [7, 6]],
    ["White", "N", [1, 7]],
    ["White", "N", [6, 7]],
    ["White", "B", [2, 7]],
    ["White", "B", [5, 7]],
    ["White", "R", [0, 7]],
    ["White", "R", [7, 7]],
    ["White", "K", [4, 7]],
    ["White", "Q", [3, 7]],

    ["Black", "P", [0, 2]],
    ["Black", "P", [1, 2]],
    ["Black", "P", [2, 2]],
    ["Black", "P", [3, 2]],
    ["Black", "P", [4, 2]],
    ["Black", "P", [5, 2]],
    ["Black", "P", [6, 2]],
    ["Black", "P", [7, 2]],
    ["Black", "N", [1, 0]],
    ["Black", "N", [6, 0]],
    ["Black", "B", [2, 0]],
    ["Black", "B", [5, 0]],
    ["Black", "R", [0, 0]],
    ["Black", "R", [7, 0]],
    ["Black", "K", [4, 0]],
    ["Black", "Q", [3, 0]],
];
// @ts-ignore
const startingPosition = {
    "White": {
        "P": [
            ["White", "P", [0, 6]],
            ["White", "P", [1, 6]],
            ["White", "P", [2, 6]],
            ["White", "P", [3, 6]],
            ["White", "P", [4, 6]],
            ["White", "P", [5, 6]],
            ["White", "P", [6, 6]],
            ["White", "P", [7, 6]],
        ],
        "N": [
            [1, 7],
            [6, 7]
        ],
        "B": [
            [2, 7],
            [5, 7]
        ],
        "R": [
            [0, 7],
            [7, 7]
        ],
        "K": [
            [4, 7],
        ],
        "Q": [
            [3, 7],
        ],
    },
    "Black": {
        "P": [
            [0,2],
            [1,2],
            [2,2],
            [3,2],
            [4,2],
            [5,2],
            [6,2],
            [7,2],
        ],
        "N": [
            [1, 0],
            [6, 0]
        ],
        "B": [
            [2, 0],
            [5, 0]
        ],
        "R": [
            [0, 0],
            [7, 0]
        ],
        "K": [
            [4, 0],
        ],
        "Q": [
            [3, 0],
        ],
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

<!-- Background Canvas Display -->
<template>
    <!-- <button @click="loadGameBoard">Load Board!</button> -->
    <div>
        <canvas height="500" width="500" ref="game-board"></canvas>
        <!-- <template v-for="([c, t, coords]) in pieceInitData">
            <chess-piece 
                :init-colour=c
                :init-type="t"
                :init-x="coords[0]"
                :init-y="coords[1]"
            ></chess-piece>
        </template> -->
    </div>
</template>

<style scoped>
canvas {
    border: 1px solid red;
    position: relative;
}
</style>