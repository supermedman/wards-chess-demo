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
import { ref, useTemplateRef } from 'vue';

type PieceInfo = {
    colour: "White" | "Black";
    type: "P" | "B" | "N" | "R" | "K" | "Q";
    x: number;
    y: number;
    inverted: boolean;
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
// const colour = ref<PieceInfo["colour"]>("Black");
    // type = ref<PieceInfo["type"]>("P"), 
    // x = ref<PieceInfo["x"]>(0),
    // y = ref<PieceInfo["y"]>(0);

// @ts-ignore
const hasMoved = ref<boolean>(false);

const emitter = defineEmits<{
    showHints: { x: number, y: number }[]
}>();


// const coords = [x, y] as const;
// @ts-ignore
// const notatePosition = () => `${String.fromCharCode(65 + Math.abs(x.value - 7))}${y.value + 1}`;
const buildNotation = (
    x: number, 
    y: number, 
    isInverted: boolean = true
) => `${String.fromCharCode(65 + ((isInverted) ? x - 1 : Math.abs(x - 8)))}${(!isInverted) ? Math.abs(y - 8) : y}`;
// @ts-ignore
// const buildStyleClass = () => `piece ${colour.value.substring(0, 1).toLowerCase()}${type.value.toLowerCase()} sqr-${x.value + 1}${y.value + 1}`;
// @ts-ignore
const typeIndex = ["P", "B", "N", "R", "K", "Q"] as const;

const pieceInstance = useTemplateRef("piece-instance");

interface ActivePieceDetails {
    c: "White" | "Black";
    t: typeof typeIndex[number];
    x: number;
    y: number;
    inv: boolean;
}

const getPieceDetails = (): ActivePieceDetails => {
    if (!pieceInstance.value) throw new Error("[WARNING]: Selected Piece is missing root element!!");
    const pieceColour = pieceInstance.value.getAttribute("init-colour") as "White" | "Black";
    const pieceType = pieceInstance.value.getAttribute("init-type") as typeof typeIndex[number];
    const pieceX = parseInt(pieceInstance.value.getAttribute("init-x") as `${number}`);
    const pieceY = parseInt(pieceInstance.value.getAttribute("init-y") as `${number}`);
    const isInverted = Boolean(pieceInstance.value.getAttribute("init-inverted"));
    return { c: pieceColour, t: pieceType, x: pieceX, y: pieceY, inv: isInverted };
};

const validMove = () => {
    const details = getPieceDetails();
    // displayRules(details.c, details.t);
    const moveList = buildActionList(details, "move");
    // EMIT EVENT TO CONTROLLER FOR SHOWING MOVEMENT HINTS!!
    moveList.forEach((move) => console.log(`${details.c} ${details.t} @ ${buildNotation(details.x, details.y, details.inv)}: `, buildNotation(move.x, move.y, details.inv)));
    // console.log(moveList);
    emitter("showHints", ...moveList);
};

const buildActionList = (details: ActivePieceDetails, actionType: "move" | "attack" | "special" = "move") => {
    const baseActions = ruleSet[details.t][actionType] as { x: number, y: number, inverter?: typeof yInvert, expand?: boolean }[];
    if (!baseActions.length) return [{x: 1, y: 1}];
    const validActions: { x: number, y: number }[] = [];
    for (const action of baseActions) {
        if (action.expand) {
            for (let i = 1; i < 8; i++) {
                // "Early Return" when piece would move through another piece in order to reach "valid" movement location
                if (document.querySelector(`div.piece.sqr-${details.x + (action.x * i)}${details.y + (action.y * i)}`)) break;
                validActions.push({
                    x: details.x + (action.x * i),
                    y: details.y + (action.y * i),
                });
            }
            continue;
        }

        if (details.t !== "P") {
            validActions.push({
                x: details.x + action.x,
                y: details.y + action.y
            });
            continue;
        }

        const isFirstPawnMove = (
            (details.c === "Black" && !details.inv && details.y === 7) ||
            (details.c === "White" && !details.inv && details.y === 2) ||
            (details.c === "Black" && details.inv && details.y === 2) ||
            (details.c === "White" && details.inv && details.y === 7)
        );
        console.log("Is First Pawn Move: ", isFirstPawnMove);

        const trueY = (action.inverter && typeof action.inverter === 'function') ? action.inverter(details.c, action.y, details.inv) : action.y;
        validActions.push({
            x: details.x + action.x,
            y: details.y + trueY
        });
        if (!isFirstPawnMove) continue;
        const trueDoubleY = (action.inverter && typeof action.inverter === 'function') ? action.inverter(details.c, action.y + 1, details.inv) : action.y + 1;
        validActions.push({
            x: details.x + action.x,
            y: details.y + trueDoubleY
        });
    }
    const isValidPosition = ({ x, y }: { x: number, y: number}) => x >= 1 && x <= 8 && y >= 1 && y <= 8;
    return validActions.filter(isValidPosition);
};

// const applyInverter = ()

const yInvert = (team: PieceInfo["colour"], mag: number, isInverted: boolean = false ) => {
    if (team === "Black") return (isInverted) ? mag * -1 : mag * 1;
    else return (isInverted) ? mag * 1 : mag * -1;

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
            { x: 0, y: 1, inverter: yInvert }
        ],
        "attack": [
            { x: 0 + 1, y: 1, inverter: yInvert },
            { x: 0 - 1, y: 1, inverter: yInvert }
        ],
        "special": [],
    },
    "N": {
        /**      8   1
         *       < ^ >
         *   7 ^   ^   ^ 2
         *     < < N > >
         *   6 v   v   v 3
         *       < v > 
         *       5   4
         */
        "move": [
            // 1
            { x: 0 + 1, y: 0 + 2 },
            // 2
            { x: 0 + 2, y: 0 + 1 },
            // 3
            { x: 0 + 2, y: 0 - 1 },
            // 4
            { x: 0 + 1, y: 0 - 2 },
            // 5
            { x: 0 - 1, y: 0 - 2 },
            // 6
            { x: 0 - 2, y: 0 - 1 },
            // 7
            { x: 0 - 2, y: 0 + 1 },
            // 8
            { x: 0 - 1, y: 0 + 2 },
        ],
        "attack": [
            { x: 0 + 1, y: 0 + 2 },
            { x: 0 + 2, y: 0 + 1 },
            { x: 0 + 2, y: 0 - 1 },
            { x: 0 + 1, y: 0 - 2 },
            { x: 0 - 1, y: 0 - 2 },
            { x: 0 - 2, y: 0 - 1 },
            { x: 0 - 2, y: 0 + 1 },
            { x: 0 - 1, y: 0 + 2 },
        ],
        "special": [],
    },
    "B": {
        "move": [
            { x: 1,  y: 1,  expand: true },
            { x: -1, y: -1, expand: true },
            { x: 1,  y: -1, expand: true },
            { x: -1, y: 1,  expand: true },
        ],
        "attack": [
            { x: 1,  y: 1,  expand: true },
            { x: -1, y: -1, expand: true },
            { x: 1,  y: -1, expand: true },
            { x: -1, y: 1,  expand: true },
        ],
        "special": []
    },
    "R": {
        "move": [
            { x: 0,  y: 1,  expand: true },
            { x: 0,  y: -1, expand: true },
            { x: 1,  y: 0,  expand: true },
            { x: -1, y: 0,  expand: true },
        ],
        "attack": [
            { x: 0,  y: 1,  expand: true },
            { x: 0,  y: -1, expand: true },
            { x: 1,  y: 0,  expand: true },
            { x: -1, y: 0,  expand: true },
        ],
        "special": []
    },
    "K": {
        "move": [
            { x: 1,  y: 1 },
            { x: -1, y: -1 },
            { x: 1,  y: -1 },
            { x: -1, y: 1 },
            { x: 0,  y: 1 },
            { x: 0,  y: -1 },
            { x: 1,  y: 0 },
            { x: -1, y: 0 },
        ],
        "attack": [
            { x: 1,  y: 1 },
            { x: -1, y: -1 },
            { x: 1,  y: -1 },
            { x: -1, y: 1 },
            { x: 0,  y: 1 },
            { x: 0,  y: -1 },
            { x: 1,  y: 0 },
            { x: -1, y: 0 },
        ],
        "special": []
    },
    "Q": {
        "move": [
            { x: 1,  y: 1,  expand: true },
            { x: -1, y: -1, expand: true },
            { x: 1,  y: -1, expand: true },
            { x: -1, y: 1,  expand: true },
            { x: 0,  y: 1,  expand: true },
            { x: 0,  y: -1, expand: true },
            { x: 1,  y: 0,  expand: true },
            { x: -1, y: 0,  expand: true },
        ],
        "attack": [
            { x: 1,  y: 1,  expand: true },
            { x: -1, y: -1, expand: true },
            { x: 1,  y: -1, expand: true },
            { x: -1, y: 1,  expand: true },
            { x: 0,  y: 1,  expand: true },
            { x: 0,  y: -1, expand: true },
            { x: 1,  y: 0,  expand: true },
            { x: -1, y: 0,  expand: true },
        ],
        "special": []
    }
};


// @ts-ignore
const displayRules = (team: PieceInfo['colour'], pieceType: PieceInfo['type']) => {
    console.log("Team Selected: ", team);
    console.log("Moveset for selected piece: ", ...ruleSet[pieceType]['move']);
};

// onMounted(() => {
//     // console.log(pieceInstance.value);
//     if (!pieceInstance.value) return;
//     // const pieceCode = pieceInstance.value.className.split(" ")[1] as `${"w" | "b"}${Lowercase<typeof typeIndex[number]>}`;
//     // console.log(pieceCode);
    
//     const pieceColour = pieceInstance.value.getAttribute("init-colour") as "White" | "Black";
//     const pieceType = pieceInstance.value.getAttribute("init-type") as typeof typeIndex[number];
//     console.log(pieceColour, pieceType);

//     displayRules(pieceColour, pieceType);
// });
</script>

<template>
    <div @click="validMove()" ref="piece-instance"></div>
</template>

<style scoped>

</style>