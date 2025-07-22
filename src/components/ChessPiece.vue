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
<script setup lang="ts">
interface PieceInfo {
    colour?: "White" | "Black";
    type?: "P" | "B" | "N" | "R" | "K" | "Q";
}

const { colour = "Black", type = "P" } = defineProps<PieceInfo>();
</script>

<template>
    <h1>Piece Details</h1>
    <p>Colour: {{colour}} || Type: {{type}}</p>
</template>

<style scoped>

</style>