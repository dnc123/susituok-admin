<template>
    <Page heading="Finansai">
        <v-menu>
            <template v-slot:activator="{ props }">
                <v-btn
                    color="primary"
                    v-bind="props"
                    class="mr-4"
                >
                    Metu filtras
                </v-btn>
            </template>

            <v-sheet>
                <v-date-picker-years v-model="filterByYear" />
            </v-sheet>
        </v-menu>

        <v-menu>
            <template v-slot:activator="{ props }">
                <v-btn
                    color="primary"
                    v-bind="props"
                >
                    Menesio filtras
                </v-btn>
            </template>

            <v-sheet>
                <v-date-picker-months v-model="filterByMonth"/>
            </v-sheet>
        </v-menu>

        <p class="mt-4">
            Aktyvus filtras:
            {{ filterByYear }}
            <template v-if="typeof filterByMonth === 'number'">
                -
                <template v-if="filterByMonth < 9">0</template>{{ filterByMonth + 1 }}
            </template>

            <v-icon
                v-if="typeof filterByMonth === 'number'"
                class="text-red cursor-pointer"
                @click="filterByMonth = undefined"
            >
                mdi-delete-circle
            </v-icon>
        </p>

        <v-table
            hover
            density="compact"
        >
            <thead>
                <tr>
                    <th></th>

                    <th>
                        Pajamos
                    </th>

                    <th>
                        Islaidos
                    </th>

                    <th>
                        Pelnas
                    </th>
                </tr>
            </thead>

            <tbody>
                <template v-for="activityType in activityTypes">
                    <tr
                        v-for="activitySubtype in activitySubtypes"
                        :key="activitySubtype"
                    >
                        <td class="text-right">
                            {{ activitySubtype }} - {{ activityType }}
                        </td>

                        <td>€{{ getActivitiesRevenue(activityType, activitySubtype).toFixed(2) }}</td>
                        <td>€{{ getActivitiesExpenses(activityType, activitySubtype).toFixed(2) }}</td>
                        <td>€{{ getActivitiesProfits(activityType, activitySubtype).toFixed(2) }}</td>
                    </tr>

                    <tr class="bg-grey-lighten-1">
                        <td class="text-right font-weight-black">
                            {{ activityType }}(bendra)
                        </td>

                        <td class="font-weight-black">
                            €{{ getActivitiesRevenue(activityType).toFixed(2) }}
                        </td>

                        <td class="font-weight-black">
                            €{{ getActivitiesExpenses(activityType).toFixed(2) }}
                        </td>

                        <td class="font-weight-black">
                            €{{ getActivitiesProfits(activityType).toFixed(2) }}
                        </td>
                    </tr>
                </template>

                <tr class="bg-black text-white">
                    <td class="text-right font-weight-black">
                        Viso:
                    </td>

                    <td class="font-weight-black">
                        €{{ getActivitiesRevenue().toFixed(2) }}
                    </td>

                    <td class="font-weight-black">
                        €{{ getActivitiesExpenses().toFixed(2) }}
                    </td>

                    <td class="font-weight-black">
                        €{{ getActivitiesProfits().toFixed(2) }}
                    </td>
                </tr>
            </tbody>
        </v-table>
    </Page>
</template>

<script setup>
import Page from "@/components/Page.vue";
import {onBeforeMount, ref} from 'vue';
import activity__findAll from '@/SDK/requests/activity__findAll';

const activities = ref([]);
const filterByYear = ref(2024);
const filterByMonth = ref();

onBeforeMount(loadActivities);

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

async function loadActivities() {
    activities.value = (await activity__findAll({})).data;
}

function isPaymentWithinFilters(payment) {
    const createdAt = new Date(payment.createdAt);
    const createdAtYear = createdAt.getFullYear();
    const createdAtMonth = createdAt.getMonth();

    if (createdAtYear !== filterByYear.value) {
        return false;
    }

    if (typeof filterByMonth.value === 'number' && filterByMonth.value !== createdAtMonth) {
        return false;
    }

    return true;
}

function getActivitiesRevenue(activityType, activitySubtype) {
    let revenue = 0;

    activities.value.forEach((activity) => {
        if (activityType && activity.type !== activityType) {
            return;
        }

        if (activitySubtype && activity.subtype !== activitySubtype) {
            return;
        }

        activity.payments.forEach((payment) => {
            if (!isPaymentWithinFilters(payment)) {
                return;
            }

            if (payment.amount > 0) {
                revenue += payment.amount;
            }
        });
    });

    return revenue;
}

function getActivitiesExpenses(activityType, activitySubtype) {
    let expenses = 0;

    activities.value.forEach((activity) => {
        if (activityType && activity.type !== activityType) {
            return;
        }

        if (activitySubtype && activity.subtype !== activitySubtype) {
            return;
        }

        activity.payments.forEach((payment) => {
            if (!isPaymentWithinFilters(payment)) {
                return;
            }

            if (payment.amount < 0) {
                expenses += payment.amount;
            }
        });
    });

    return expenses;
}

function getActivitiesProfits(activityType, activitySubtype) {
    const activityRevenue = getActivitiesRevenue(activityType, activitySubtype);
    const activityExpenses = getActivitiesExpenses(activityType, activitySubtype);

    return activityRevenue + activityExpenses;
}
</script>
