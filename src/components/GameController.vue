<script setup lang="ts">
// @ts-ignore
import ChessBoard from './ChessBoard.vue';
// @ts-ignore
import ChessPiece from './ChessPiece.vue';
//@ts-ignore
const pieceInitData: [
    colour: "Black" | "White",
    type: "P" | "N" | "B" | "R" | "K" | "Q",
    coords: [x: number, y: number],
    classStyle: string
][] = [
    ["White", "P", [0, 6], "piece wp sqr-06"],
    ["White", "P", [1, 6], "piece wp sqr-16"],
    ["White", "P", [2, 6], "piece wp sqr-26"],
    ["White", "P", [3, 6], "piece wp sqr-36"],
    ["White", "P", [4, 6], "piece wp sqr-46"],
    ["White", "P", [5, 6], "piece wp sqr-56"],
    ["White", "P", [6, 6], "piece wp sqr-66"],
    ["White", "P", [7, 6], "piece wp sqr-76"],
    ["White", "N", [1, 7], "piece wn sqr-17"],
    ["White", "N", [6, 7], "piece wn sqr-67"],
    ["White", "B", [2, 7], "piece wb sqr-27"],
    ["White", "B", [5, 7], "piece wb sqr-57"],
    ["White", "R", [0, 7], "piece wr sqr-07"],
    ["White", "R", [7, 7], "piece wr sqr-77"],
    ["White", "K", [4, 7], "piece wk sqr-47"],
    ["White", "Q", [3, 7], "piece wq sqr-37"],
    ["Black", "P", [0, 2], "piece bp sqr-02"],
    ["Black", "P", [1, 2], "piece bp sqr-12"],
    ["Black", "P", [2, 2], "piece bp sqr-22"],
    ["Black", "P", [3, 2], "piece bp sqr-32"],
    ["Black", "P", [4, 2], "piece bp sqr-42"],
    ["Black", "P", [5, 2], "piece bp sqr-52"],
    ["Black", "P", [6, 2], "piece bp sqr-62"],
    ["Black", "P", [7, 2], "piece bp sqr-72"],
    ["Black", "N", [1, 0], "piece bn sqr-10"],
    ["Black", "N", [6, 0], "piece bn sqr-60"],
    ["Black", "B", [2, 0], "piece bb sqr-20"],
    ["Black", "B", [5, 0], "piece bb sqr-50"],
    ["Black", "R", [0, 0], "piece br sqr-00"],
    ["Black", "R", [7, 0], "piece br sqr-70"],
    ["Black", "K", [4, 0], "piece bk sqr-40"],
    ["Black", "Q", [3, 0], "piece bq sqr-30"],
];
</script>

<template>
<div>
<chess-board></chess-board>
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

<chess-piece v-for="[c, t, coords, style] in pieceInitData"
    :init-colour="c"
    :init-type="t"
    :init-x=coords[0]
    :init-y=coords[1]
    :class=style
></chess-piece>
</div>
<chess-piece init-colour="Black" init-type="B"></chess-piece>

</template>

<style scoped>

</style>