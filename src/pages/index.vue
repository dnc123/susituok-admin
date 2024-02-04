<template>
    <Page heading="Veikla">
        <div class="d-flex justify-space-between align-end mb-4">
            <div>
                <div class="mb-4">
                    <v-btn-toggle
                        v-model="filterActivityType"
                        color="primary"
                        mandatory
                    >
                        <v-btn
                            v-for="activityType in activityTypes"
                            :key="activityType"
                            :value="activityType"
                        >
                            {{ activityType }}
                        </v-btn>
                    </v-btn-toggle>
                </div>

                <div>
                    <v-btn-toggle
                        v-model="filterActivitySubtype"
                        color="primary"
                        mandatory
                    >
                        <v-btn
                            v-for="activitySubtype in activitySubtypes"
                            :key="activitySubtype"
                            :value="activitySubtype"
                        >
                            {{ activitySubtype }}
                        </v-btn>
                    </v-btn-toggle>
                </div>
            </div>

            <v-btn
                class="mb-5 bg-blue"
                @click="isActivityCreateModalOpened = true"
            >
                Prideti
            </v-btn>
        </div>

        <v-table>
            <thead>
                <tr>
                    <th class="text-left">
                        Starto data
                    </th>

                    <th>
                        Pelnas
                    </th>
                </tr>
            </thead>

            <tbody>
                <tr
                    v-for="filteredEvent in filteredEvents"
                    class="cursor-pointer"
                    @click="isActivityViewModalOpened = true"
                >
                    <td>{{ filteredEvent.startDate }}</td>
                    <td>€ {{ filteredEvent.profit }}</td>
                </tr>
            </tbody>
        </v-table>

        <v-dialog
            v-model="isActivityViewModalOpened"
            width="auto"
        >
            <v-card>
                <v-card-text>
                    Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
                </v-card-text>

                <v-card-actions>
                    <v-btn
                        color="primary"
                        @click="isActivityViewModalOpened = false"
                    >
                        Uzdaryti
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog
            v-model="isActivityCreateModalOpened"
            width="500"
        >
            <v-card>
                <v-card-title>
                    Sukurti irasa
                </v-card-title>

                <div class="px-3">
                    <v-select
                        v-model="createEventForm.type"
                        :items="activityTypes"
                        label="Veikla"
                        class="mb-4"
                    />

                    <v-select
                        v-model="createEventForm.subtype"
                        :items="activitySubtypes"
                        label="Veiklos sritis"
                    />
                </div>

                <v-card-actions>
                    <div class="d-flex justify-end w-100">
                        <v-btn
                            color="primary"
                            @click="isActivityCreateModalOpened = false"
                        >
                            Uzdaryti
                        </v-btn>

                        <v-btn class="bg-blue">
                            Prideti
                        </v-btn>
                    </div>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </Page>
</template>

<script setup lang="ts">
import Page from "@/components/Page.vue";
import {computed, ref} from "vue";

const isActivityViewModalOpened = ref(false);
const isActivityCreateModalOpened = ref(false);
const filterActivityType = ref();
const filterActivitySubtype = ref();

const events = ref([
    {
        type: 'sampano-staliukas',
        subtype: 'darbas',
        startDate: '2023-05-12',
        profit: 60,
    },

    {
        type: 'sampano-staliukas',
        subtype: 'darbas',
        startDate: '2023-05-30',
        profit: 190,
    },

    {
        type: 'sampano-staliukas',
        subtype: 'darbas',
        startDate: '2023-05-06',
        profit: 75,
    },
]);

const createEventForm = ref({
    type: 'koordinavimas',
    subtype: 'darbas'
});

const activityTypes = ref([
    'koordinavimas',
    'planavimas',
    'sampano-staliukas',
    'e-kvietimai',
]);

const activitySubtypes = ref([
    'darbas',
    'reklama',
    'inventorius',
]);

const filteredEvents = computed(() => {
    return events.value.filter((event) => {
        return event.type === filterActivityType.value
            && event.subtype === filterActivitySubtype.value;
    });
});
</script>
