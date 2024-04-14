<template>
    <div class="modal" @click="closeModalOutside">
        <div class="modal-content">
            <span class="close" @click="closeModal">&times;</span>
            <img :src="image" alt="Enlarged Image" loading="lazy">
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
        closeModalOutside() {
            if (event.target.classList.contains('modal')) {
                this.closeModal();
            }
        }
    },
}
</script>

<style scoped>
.modal {
    display: block;
    position: fixed;
    z-index: 9999;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    overflow: auto;
}

.modal-content {
    position: relative;
    background-color: #fefefe;
    margin: 2% auto;
    padding: 10px;
    border: 1px solid #888;
    width: 900px;
}

.modal-content img {
    width: 900px;
    height: 900px;
}

.close {
    position: fixed;
    right: 0;
    top: 0;
    margin-right: 25px;
    margin-top: 1px;
    color: #ffffff;
    font-size: 70px;
    font-weight: bold;
    cursor: pointer;
    background-color: transparent;
}

@media screen and (max-width: 1200px) {
    .modal-content {
        margin: 10% auto;
        max-width: 600px;
    }

    .modal-content img {
        width: 600px;
        height: 600px;
    }

    .close {
        font-size: 60px;
        margin-right: 20px;
    }
}

@media screen and (max-width: 800px) {
    .modal-content {
        margin: 25% auto;
        max-width: 400px;
    }

    .modal-content img {
        width: 400px;
        height: 400px;
    }

    .close {
        font-size: 50px;
        margin-right: 15px;
    }
}

@media screen and (max-width: 500px) {
    .modal-content {
        margin: 50% auto;
        width: 310px;
        padding-top: 5px;
        padding-left: 5px;
        padding-right: 5px;
        padding-bottom: .5px;
    }

    .modal-content img {
        width: 310px;
        height: 310px;
    }

    .close {
        font-size: 40px;
        margin-right: 5px;
    }
}
</style>
