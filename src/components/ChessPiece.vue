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
};

const buildActionList = (details: ActivePieceDetails, actionType: "move" | "attack" | "special" = "move") => {
    const baseActions = ruleSet[details.t][actionType] as { x: number, y: number, inverter?: typeof yInvert }[];
    if (!baseActions.length) return [{x: 1, y: 1}];
    const validActions: { x: number, y: number }[] = [];
    for (const action of baseActions) {
        const trueY = (action.inverter && typeof action.inverter === 'function') ? action.inverter(details.c, action.y, details.inv) : action.y;
        validActions.push({
            x: details.x + action.x,
            y: details.y + trueY
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
    <div 
        @click="validMove()"
        ref="piece-instance"
    >
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

.hint {
    background-color: rgba(0, 0, 0, .14);
    background-clip: content-box;
    border-radius: 50%;
    box-sizing: border-box;
    padding: 4.2%;
    height: 12.5%;
    width: 12.5%;
    left: 0;
    top: 0;
    position: absolute;
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