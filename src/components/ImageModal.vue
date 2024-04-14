<template>
    <div class="modal">
        <div class="modal-content">
            <span class="close" @click="closeModal">&times;</span>
            <img :src="image" alt="Enlarged Image">
        </div>
    </div>
</template>

<script>
export default {
    props: {
        image: {
            type: String,
            required: true,
        },
    },
    methods: {
        closeModal() {
            this.$emit('close');
        },
    },
}
</script>

<style scoped>
/* Styles for the modal */
.modal {
    display: block;
    /* Ensure modal is displayed */
    position: fixed;
    /* Position the modal */
    z-index: 9999;
    /* Set the z-index to ensure it's above other content */
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    /* Semi-transparent background */
    overflow: auto;
}

.modal-content {
    position: relative;
    background-color: #fefefe;
    /* Modal background color */
    margin: 15% auto;
    padding: 20px;
    border: 1px solid #888;
    width: 80%;
    max-width: 700px;
    /* Set max width for modal */
}

.close {
    position: absolute;
    right: 0;
    top: 0;
    color: #aaa;
    font-size: 28px;
    font-weight: bold;
    cursor: pointer;
}

/* Hide close button on mobile */
@media screen and (max-width: 700px) {
    .close {
        display: none;
    }
}
</style>
