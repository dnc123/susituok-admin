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
                v-if="isFiltersSelected"
                class="mb-5 bg-blue"
                @click="submitCreateActivityForm()"
            >
                Prideti
            </v-btn>
        </div>

        <v-table
            v-if="isFiltersSelected"
            hover
        >
            <thead>
                <tr>
                    <th>
                        Lokacija
                    </th>

                    <th>
                        Pelnas
                    </th>

                    <th>
                        Veiksmai
                    </th>
                </tr>
            </thead>

            <tbody>
                <tr
                    v-for="filteredActivity in sortedActivities"
                    :key="filteredActivity._id"
                >
                    <td>
                        {{ filteredActivity.location || '-' }}
                    </td>

                    <td
                        :class="{
                            'text-green': getActivityProfit(filteredActivity) > 0,
                            'text-red': getActivityProfit(filteredActivity) < 0,
                        }"
                    >
                        € {{ getActivityProfit(filteredActivity).toFixed(2) }}
                    </td>

                    <td class="pa-1">
                        <v-btn @click="openViewActivityPaymentsModal(filteredActivity)">
                            mokejimai({{ filteredActivity.payments.length }})
                        </v-btn>

                        <v-btn @click="openViewActivityDatesModal(filteredActivity)">
                            datos({{ filteredActivity.dates.length }})
                        </v-btn>

                        <v-btn @click="openViewActivityNotesModal(filteredActivity)">
                            uzrasai({{ filteredActivity.note ? 'yra' : 'nera' }})
                        </v-btn>

                        <v-btn @click="openViewActivityLocationModal(filteredActivity)">
                            lokacija({{ filteredActivity.location ? 'yra' : 'nera' }})
                        </v-btn>

                        <v-btn @click="openViewActivityContactsModal(filteredActivity)">
                            kontaktai({{ filteredActivity.contacts.length }})
                        </v-btn>

                        <v-menu>
                            <template v-slot:activator="{ props }">
                                <v-btn
                                    class="bg-red"
                                    :loading="isActivityBeingDeleted"
                                    v-bind="props"
                                >
                                    Istrinti
                                </v-btn>
                            </template>

                            <v-list>
                                <v-list-item class="cursor-pointer">
                                    Atsaukti
                                </v-list-item>

                                <v-list-item @click="deleteActivity(filteredActivity._id)">
                                    Istrinti
                                </v-list-item>
                            </v-list>
                        </v-menu>
                    </td>
                </tr>
            </tbody>
        </v-table>

        <ModalViewActivityPayments
            v-model="isModalViewActivityPaymentsOpened"
            :activity="activityBeingViewed"
        />

        <ModalViewActivityDates
            v-model="isModalViewActivityDatesOpened"
            :activity="activityBeingViewed"
        />

        <ModalViewActivityNotes
            v-model="isModalViewActivityNotesOpened"
            :activity="activityBeingViewed"
        />

        <ModalViewActivityLocation
            v-model="isModalViewActivityLocationOpened"
            :activity="activityBeingViewed"
        />

        <ModalViewActivityContacts
            v-model="isModalViewActivityContactsOpened"
            :activity="activityBeingViewed"
        />
    </Page>
</template>

<script setup>
import Page from "@/components/Page.vue";
import {computed, ref, onMounted} from "vue";
import activity__create from "@/SDK/requests/activity__create";
import activity__findAll from "@/SDK/requests/activity__findAll";
import ModalViewActivityContacts from '@/components/ViewActivity/ModalViewActivityContacts.vue';
import ModalViewActivityDates from '@/components/ViewActivity/ModalViewActivityDates.vue';
import ModalViewActivityLocation from '@/components/ViewActivity/ModalViewActivityLocation.vue';
import ModalViewActivityNotes from '@/components/ViewActivity/ModalViewActivityNotes.vue';
import ModalViewActivityPayments from '@/components/ViewActivity/ModalViewActivityPayments.vue';
import activity__delete from '@/SDK/requests/activity__delete';

const filterActivityType = ref();
const filterActivitySubtype = ref();
const isActivityBeingInserted = ref(false);
const isActivityInsertFormInvalid = ref(false);
const activities = ref([]);
const activityBeingViewedID = ref();
const isModalViewActivityPaymentsOpened = ref(false);
const isModalViewActivityDatesOpened = ref(false);
const isModalViewActivityNotesOpened = ref(false);
const isModalViewActivityLocationOpened = ref(false);
const isModalViewActivityContactsOpened = ref(false);
const isActivityBeingDeleted = ref(false);

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

const isFiltersSelected = computed(() => {
    return !!filterActivityType.value && !!filterActivitySubtype.value;
})

const activityBeingViewed = computed(() => {
    return activities.value.find((activity) => {
        return activity._id === activityBeingViewedID.value;
    });
});

const filteredActivities = computed(() => {
    return activities.value.filter((event) => {
        return event.type === filterActivityType.value
            && event.subtype === filterActivitySubtype.value;
    });
});

const sortedActivities = computed(() => {
    const allActivities = JSON.parse(JSON.stringify(filteredActivities.value));

    allActivities.sort((a, b) => {
        if (a.createdAtTimestamp < b.createdAtTimestamp) {
            return 1;
        }

        if (a.createdAtTimestamp > b.createdAtTimestamp) {
            return -1;
        }

        return 0;
    });

    return allActivities;
});

async function deleteActivity(targetActivityID) {
    isActivityBeingDeleted.value = true;

    try {
        await activity__delete({
            activityID: targetActivityID,
        });

        await loadActivities();
    } catch (e) {}

    isActivityBeingDeleted.value = false;
}

async function submitCreateActivityForm () {
    if (!filterActivityType.value || !filterActivitySubtype.value) {
        isActivityInsertFormInvalid.value = true;

        return;
    }

    isActivityBeingInserted.value = true;

    try {
        await activity__create({
            type: filterActivityType.value,
            subtype: filterActivitySubtype.value,
            dates: [],
            payments: [],
            contacts: [],
            location: '',
            note: '',
            createdAtTimestamp: Date.now(),
        });

        await loadActivities();
    } catch (e) {}

    isActivityBeingInserted.value = false;
}

async function loadActivities() {
    activities.value = (await activity__findAll({})).data;
}

function getActivityProfit(activity) {
    let profit = 0;

    activity.payments.forEach((payment) => {
        profit += payment.amount;
    });

    return profit;
}

function openActivityModal(activity) {
    activityBeingViewedID.value = activity._id;
}

function openViewActivityPaymentsModal(activity) {
    openActivityModal(activity);
    isModalViewActivityPaymentsOpened.value = true;
}

function openViewActivityDatesModal(activity) {
    openActivityModal(activity);
    isModalViewActivityDatesOpened.value = true;
}

function openViewActivityNotesModal(activity) {
    openActivityModal(activity);
    isModalViewActivityNotesOpened.value = true;
}

function openViewActivityLocationModal(activity) {
    openActivityModal(activity);
    isModalViewActivityLocationOpened.value = true;
}

function openViewActivityContactsModal(activity) {
    openActivityModal(activity);
    isModalViewActivityContactsOpened.value = true;
}

onMounted(loadActivities);
</script>
