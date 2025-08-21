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

type PiecePlaceholder = HTMLDivElement | null;

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

const findDiv = (c: string) => {
    const foundDiv = document.querySelector(`div.${c}`) as HTMLDivElement | null;
    if (!foundDiv) throw new Error("[WARNING]: Div failed to be located!");
    return foundDiv;
};

import { ref, useTemplateRef, watch } from 'vue';

const canvasBoard = useTemplateRef<HTMLCanvasElement>("game-board");
const mainContainer = useTemplateRef<HTMLDivElement>("main-container");

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
const selectedPiece = ref<PiecePlaceholder>(null);


const findAnchorComments = (childrenList: ArrayIterator<[number, ChildNode]>, kids = [...childrenList]) => {
    return kids.filter(([_, child]) => child instanceof Comment && child.data.startsWith("/")) as [number, Comment][];
};
const findIndexOfAnchor = (childList: [number, Comment][], anchorName: string) => {
    let finalIndex = mainContainer.value!.children.length;
    switch(anchorName) {
        case "Hint":
            const hintAnchorLocated = childList.find(([_, c]) => c.data === "/MoveHints");
            // console.log("Located Anchor For Hints: ", hintAnchorLocated);
            finalIndex = hintAnchorLocated ? hintAnchorLocated[0] : finalIndex;
        break;
        default: break;
    }
    return finalIndex;
};
const insertAtAnchor = (insertList: HTMLDivElement[], insertIdx: number) => {
    if (!mainContainer.value) throw new Error("[WARNING]: Main Game Container Missing!!");
    const insertAnchor = mainContainer.value.children[insertIdx];

    for (const ele of insertList) 
        if (insertAnchor) mainContainer.value.insertBefore(ele, insertAnchor);
        else mainContainer.value.appendChild(ele);
};

const buildHintDivs = (hints: { x: number; y: number; }[]) => {
    const hintEleList: HTMLDivElement[] = [];
    for (const hint of hints) {
        const sqrOccupied = document.querySelector(`div.piece.sqr-${hint.x}${hint.y}`);
        if (sqrOccupied) continue;
        // Call for movement check between game board and piece clicked, voiding further movement logic when first filitering valid movesets
        const hintEle = document.createElement('div');
        hintEle.className = `hint ${(isInverted.value) ? "inv ": ""}sqr-${hint.x}${hint.y}`;
        hintEle.onclick = () => moveSelected(`${hint.x}${hint.y}`);
        hintEleList.push(hintEle);
    }
    return hintEleList;
};
const removeHintDivs = () => {
    const hintDivs = Array.from(document.getElementsByClassName("hint") as HTMLCollectionOf<HTMLDivElement>);
    for (const hint of hintDivs) hint.remove();
};

function updateHints(...args: { x: number; y: number; }[]) {
    if (!mainContainer.value) throw new Error("[WARNING]: Main Game Container Missing!!");
    removeHintDivs();
    // console.log("Hint Update Event Recieved: ", args);
    const builtHints = buildHintDivs(args);
    // console.log("Constructed Hints: ", builtHints);
    // console.log("Main Container State: ", mainContainer.value);

    const cmntList = findAnchorComments(mainContainer.value.childNodes.entries());
    // console.log("Comment List: ", cmntList);

    const anchorIdx = findIndexOfAnchor(cmntList, "Hint");
    // console.log("Piece Movement Hint Anchor Position: ", anchorIdx);
    insertAtAnchor(builtHints, anchorIdx);
}

function updateSelected(className: string) {
    console.log("Class Name of new piece Selected: ", className);
    if (selectedPiece.value && selectedPiece.value.className === className) return;
    selectedPiece.value = findDiv(className.split(" ").join("."));
    console.log("Newly Selected Piece: ", selectedPiece);
}
function moveSelected(moveTo: string) {
    if (!selectedPiece.value) return;
    const cList = selectedPiece.value.className.split(" ");
    cList[cList.length - 1] = `sqr-${moveTo}`;
    selectedPiece.value.className = cList.join(" ");
    removeHintDivs();
}


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
    <div class="game-cont" ref="main-container">
        <canvas ref="game-board"></canvas>
        <chess-piece v-for="[c, t, coords, style] in buildInitData(isInverted)"
            :init-colour="c"
            :init-type="t"
            :init-x=coords[0]
            :init-y=coords[1]
            :init-inverted=isInverted
            :class=style
            @show-hints="updateHints"
            @piece-selected="updateSelected"
        ></chess-piece>
        <!--/MoveHints-->
    </div>
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

<style>
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

.hint {
    background-color: rgba(53, 48, 48, 0.14);
    background-clip: content-box;
    border-radius: 50%;
    box-sizing: border-box;
    padding: 4.2%;
    height: 12.5%;
    width: 12.5%;
    left: 0;
    top: 0;
    position: absolute;
    pointer-events: none;
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

.sqr-13 { transform: translateY(500%); }
.sqr-23 { transform: translate(100%, 500%); }
.sqr-33 { transform: translate(200%, 500%); }
.sqr-43 { transform: translate(300%, 500%); }
.sqr-53 { transform: translate(400%, 500%); }
.sqr-63 { transform: translate(500%, 500%); }
.sqr-73 { transform: translate(600%, 500%); }
.sqr-83 { transform: translate(700%, 500%); }

.sqr-14 { transform: translateY(400%); }
.sqr-24 { transform: translate(100%, 400%); }
.sqr-34 { transform: translate(200%, 400%); }
.sqr-44 { transform: translate(300%, 400%); }
.sqr-54 { transform: translate(400%, 400%); }
.sqr-64 { transform: translate(500%, 400%); }
.sqr-74 { transform: translate(600%, 400%); }
.sqr-84 { transform: translate(700%, 400%); }

.sqr-15 { transform: translateY(300%); }
.sqr-25 { transform: translate(100%, 300%); }
.sqr-35 { transform: translate(200%, 300%); }
.sqr-45 { transform: translate(300%, 300%); }
.sqr-55 { transform: translate(400%, 300%); }
.sqr-65 { transform: translate(500%, 300%); }
.sqr-75 { transform: translate(600%, 300%); }
.sqr-85 { transform: translate(700%, 300%); }

.sqr-16 { transform: translateY(200%); }
.sqr-26 { transform: translate(100%, 200%); }
.sqr-36 { transform: translate(200%, 200%); }
.sqr-46 { transform: translate(300%, 200%); }
.sqr-56 { transform: translate(400%, 200%); }
.sqr-66 { transform: translate(500%, 200%); }
.sqr-76 { transform: translate(600%, 200%); }
.sqr-86 { transform: translate(700%, 200%); }

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

/** INVERTED */
.inv.sqr-11 { transform: translate(700%); }
.inv.sqr-21 { transform: translate(600%); }
.inv.sqr-31 { transform: translate(500%); }
.inv.sqr-41 { transform: translate(400%); }
.inv.sqr-51 { transform: translate(300%); }
.inv.sqr-61 { transform: translate(200%); }
.inv.sqr-71 { transform: translate(100%); }
.inv.sqr-81 { transform: translateY(0%); }

.inv.sqr-12 { transform: translate(700%, 100%); }
.inv.sqr-22 { transform: translate(600%, 100%); }
.inv.sqr-32 { transform: translate(500%, 100%); }
.inv.sqr-42 { transform: translate(400%, 100%); }
.inv.sqr-52 { transform: translate(300%, 100%); }
.inv.sqr-62 { transform: translate(200%, 100%); }
.inv.sqr-72 { transform: translate(100%, 100%); }
.inv.sqr-82 { transform: translateY(100%); }

.inv.sqr-13 { transform: translate(700%, 200%); }
.inv.sqr-23 { transform: translate(600%, 200%); }
.inv.sqr-33 { transform: translate(500%, 200%); }
.inv.sqr-43 { transform: translate(400%, 200%); }
.inv.sqr-53 { transform: translate(300%, 200%); }
.inv.sqr-63 { transform: translate(200%, 200%); }
.inv.sqr-73 { transform: translate(100%, 200%); }
.inv.sqr-83 { transform: translateY(200%); }

.inv.sqr-14 { transform: translate(700%, 300%); }
.inv.sqr-24 { transform: translate(600%, 300%); }
.inv.sqr-34 { transform: translate(500%, 300%); }
.inv.sqr-44 { transform: translate(400%, 300%); }
.inv.sqr-54 { transform: translate(300%, 300%); }
.inv.sqr-64 { transform: translate(200%, 300%); }
.inv.sqr-74 { transform: translate(100%, 300%); }
.inv.sqr-84 { transform: translateY(300%); }

.inv.sqr-15 { transform: translate(700%, 400%); }
.inv.sqr-25 { transform: translate(600%, 400%); }
.inv.sqr-35 { transform: translate(500%, 400%); }
.inv.sqr-45 { transform: translate(400%, 400%); }
.inv.sqr-55 { transform: translate(300%, 400%); }
.inv.sqr-65 { transform: translate(200%, 400%); }
.inv.sqr-75 { transform: translate(100%, 400%); }
.inv.sqr-85 { transform: translateY(400%); }

.inv.sqr-16 { transform: translate(700%, 500%); }
.inv.sqr-26 { transform: translate(600%, 500%); }
.inv.sqr-36 { transform: translate(500%, 500%); }
.inv.sqr-46 { transform: translate(400%, 500%); }
.inv.sqr-56 { transform: translate(300%, 500%); }
.inv.sqr-66 { transform: translate(200%, 500%); }
.inv.sqr-76 { transform: translate(100%, 500%); }
.inv.sqr-86 { transform: translateY(500%); }

.inv.sqr-17 { transform: translate(700%, 600%); }
.inv.sqr-27 { transform: translate(600%, 600%); }
.inv.sqr-37 { transform: translate(500%, 600%); }
.inv.sqr-47 { transform: translate(400%, 600%); }
.inv.sqr-57 { transform: translate(300%, 600%); }
.inv.sqr-67 { transform: translate(200%, 600%); }
.inv.sqr-77 { transform: translate(100%, 600%); }
.inv.sqr-87 { transform: translateY(600%); }

.inv.sqr-18 { transform: translate(700%, 700%); }
.inv.sqr-28 { transform: translate(600%, 700%); }
.inv.sqr-38 { transform: translate(500%, 700%); }
.inv.sqr-48 { transform: translate(400%, 700%); }
.inv.sqr-58 { transform: translate(300%, 700%); }
.inv.sqr-68 { transform: translate(200%, 700%); }
.inv.sqr-78 { transform: translate(100%, 700%); }
.inv.sqr-88 { transform: translateY(700%); }

</style>