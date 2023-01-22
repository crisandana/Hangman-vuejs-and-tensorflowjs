<template>
<div v-show="show" class="overlay"> <!-- v-show - conditionally displaying-->
    <div class="dialog">
        <div class="dialog__content">
            <h2 class="dialog__title" v-text="title"></h2>
            <p class="dialog__description" v-text="description"></p>

            <p class="dialog__description">To start the game, say "New Game"</p>

            <p class="dialog__description">
                To reveal the word, say "Reveal the word"
            </p>

            <p class="dialog__description">
                To place some character, say that character like "A"
            </p>
        </div>

        <hr />

        <div class="dialog__footer">
            <button @click="cancel" class="dialog__cancel">Cancel</button>
        </div>
    </div>
</div>
</template>

<script>
export default {
    props: ["show", "title", "description", "cancel"]
};
</script>

