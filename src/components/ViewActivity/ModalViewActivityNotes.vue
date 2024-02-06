<template>
    <v-dialog
        width="700"
        v-model="isModalOpened"
    >
        <v-card>
            <div class="pa-5">
                <v-textarea
                    v-model="newNote"
                    placeholder="Uzrasai..."
                    auto-grow
                />

                <v-btn
                    class="bg-green"
                    @click="submitUpdateNoteForm()"
                >
                    Atnaujinti
                </v-btn>
            </div>
        </v-card>
    </v-dialog>
</template>

<script setup>
import {ref, watch} from 'vue';
import activity__updateNote from '@/SDK/requests/activity__updateNote';

const isModalOpened = defineModel();

const props = defineProps({
    activity: {
        type: Object,
        required: true,
    },
});

const newNote = ref();

watch(isModalOpened, (isOpened) => {
    if (isOpened) {
        newNote.value = props.activity.note;
    }
})

async function submitUpdateNoteForm () {
    await activity__updateNote({
        activityID: props.activity._id,
        note: newNote.value,
    });

    isModalOpened.value = false;
    props.activity.note = newNote.value;
}
</script>
