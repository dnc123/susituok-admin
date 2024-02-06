<template>
    <Page heading="Kalendorius">
        <v-calendar
            ref="calendar"
            v-model="value"
            :weekdays="[0, 1, 2, 3, 4, 5, 6]"
            view-mode="month"
            :events="calendarItems"
        />
    </Page>
</template>

<script setup>
import Page from "@/components/Page.vue";
import { VCalendar } from 'vuetify/labs/VCalendar'
import { ref, computed, onBeforeMount } from 'vue';
import activity__findAll from '@/SDK/requests/activity__findAll';

const value = ref([new Date()]);
const activities = ref([]);

onBeforeMount(loadActivities);

async function loadActivities() {
    activities.value = (await activity__findAll({})).data;
}

const calendarItems = computed(() => {
    const items = [];

    activities.value.forEach((activity) => {
        activity.dates.forEach((date) => {
            items.push({
                title: `${date.name}(${activity.type})`,
                start: new Date(date.date),
                end: new Date(date.date),
                color: 'red',
                allDay: true,
            });
        });
    });

    return items;
})
</script>

<style lang="scss">
.v-calendar-weekly__day-alldayevents-container {
    height: auto !important;

    .v-chip {
        height: auto !important;
    }

    .v-chip__content {
        text-wrap: balance !important;
        word-break: break-word !important;
    }
}
</style>
