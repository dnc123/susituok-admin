<template>
    <v-dialog
        width="700"
        v-model="isModalOpened"
    >
        <v-card>
            <div class="pa-5">
                <v-text-field
                    v-model="newLocation"
                    placeholder="Purmaliai"
                />

                <v-btn
                    class="bg-green"
                    @click="submitUpdateLocationForm()"
                >
                    Atnaujinti
                </v-btn>
            </div>
        </v-card>
    </v-dialog>
</template>

<script setup>
import {ref, watch} from 'vue';
import activity__updateLocation from '@/SDK/requests/activity__updateLocation';

const isModalOpened = defineModel();

const props = defineProps({
    activity: {
        type: Object,
        required: true,
    },
});

const newLocation = ref();

watch(isModalOpened, (isOpened) => {
    if (isOpened) {
        newLocation.value = props.activity.location;
    }
})

async function submitUpdateLocationForm () {
    await activity__updateLocation({
        activityID: props.activity._id,
        location: newLocation.value,
    });

    isModalOpened.value = false;
    props.activity.location = newLocation.value;
}
</script>
