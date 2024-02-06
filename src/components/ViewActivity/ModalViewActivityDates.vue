<template>
    <v-dialog
        width="700"
        v-model="isModalOpened"
    >
        <v-card>
            <v-table hover>
                <thead>
                    <tr>
                        <th>
                            Pavadinimas
                        </th>

                        <th>
                            Data
                        </th>

                        <th>
                            Uzrasai
                        </th>

                        <th>
                            Veiksmai
                        </th>
                    </tr>
                </thead>

                <tbody>
                    <tr>
                        <td class="pa-4">
                            <v-text-field
                                v-model="createDateForm.name"
                                hide-details
                            />
                        </td>

                        <td class="pa-4">
                            <v-menu
                                v-model="isDatePickerPopupOpened"
                                :close-on-content-click="false"
                                :nudge-right="40"
                                lazy
                                transition="scale-transition"
                                offset-y
                                full-width
                                min-width="290px"
                            >
                                <template v-slot:activator="{ props }">
                                    <v-text-field
                                        v-model="createDateForm.date"
                                        hide-details
                                        readonly
                                        v-bind="props"
                                    />
                                </template>

                                <v-date-picker
                                    v-model="createDateForm.date"
                                    @input="isDatePickerPopupOpened = false"
                                >
                                    <template #actions>
                                        <v-btn
                                            flat
                                            color="primary"
                                            @click="isDatePickerPopupOpened = false"
                                        >
                                            Uzdaryti
                                        </v-btn>
                                    </template>
                                </v-date-picker>
                            </v-menu>
                        </td>

                        <td>
                            <v-text-field
                                v-model="createDateForm.note"
                                hide-details
                            />
                        </td>

                        <td>
                            <v-btn
                                class="bg-green"
                                :loading="isDateBeingCreated"
                                @click="submitCreateDateForm()"
                            >
                                Prideti
                            </v-btn>
                        </td>
                    </tr>

                    <tr
                        v-for="date in sortedDates"
                        :class="{
                            'bg-red-lighten-5': date.amount < 0,
                            'bg-green-lighten-5': date.amount > 0,
                        }"
                    >
                        <td>{{ date.name }}</td>
                        <td>{{ date.date }}</td>
                        <td>{{ date.note }}</td>

                        <td>
                            <v-menu>
                                <template v-slot:activator="{ props }">
                                    <v-btn
                                        class="bg-red"
                                        :loading="isDateBeingDeleted"
                                        v-bind="props"
                                    >
                                        Istrinti
                                    </v-btn>
                                </template>

                                <v-list>
                                    <v-list-item class="cursor-pointer">
                                        Atsaukti
                                    </v-list-item>

                                    <v-list-item @click="deleteActivityDate(date)">
                                        Istrinti
                                    </v-list-item>
                                </v-list>
                            </v-menu>
                        </td>
                    </tr>
                </tbody>
            </v-table>
        </v-card>
    </v-dialog>
</template>

<script setup>
import {computed, ref} from 'vue';
import activity__createDate from '@/SDK/requests/activity__createDate';
import activity__deleteDate from '@/SDK/requests/activity__deleteDate';
import formatDate from '@/helpers/shared/date/formatDate';

const isModalOpened = defineModel();

const isDateBeingCreated = ref(false);
const isDateBeingDeleted = ref(false);
const isDatePickerPopupOpened = ref(false);

const createDateForm = ref({
    name: '',
    date: undefined,
    note: '',
});

const props = defineProps({
    activity: {
        type: Object,
        required: true,
    },
});

const sortedDates = computed(() => {
    const allDates = JSON.parse(JSON.stringify(props.activity.dates));

    allDates.sort((a, b) => {
        if (a.createdAt < b.createdAt) {
            return 1;
        }

        if (a.createdAt > b.createdAt) {
            return -1;
        }

        return 0;
    });

    return allDates;
});

async function deleteActivityDate(date) {
    isDateBeingDeleted.value = true;

    try {
        await activity__deleteDate({
            activityID: props.activity._id,
            createdAt: date.createdAt,
        });

        props.activity.dates = props.activity.dates.filter(({ createdAt }) => {
            return date.createdAt !== createdAt;
        });
    } catch (e) {}

    isDateBeingDeleted.value = false;
}

function resetCreateDateForm() {
    createDateForm.value.name = '';
    createDateForm.value.date = undefined;
    createDateForm.value.note = '';
}

async function submitCreateDateForm() {
    isDateBeingCreated.value = true;

    try {
        const newDate = (await activity__createDate({
            activityID: props.activity._id,
            name: createDateForm.value.name,
            date: formatDate(createDateForm.value.date),
            note: createDateForm.value.note,
            createdAt: Date.now(),
        })).data;

        props.activity.dates.unshift(newDate);
        resetCreateDateForm();
    } catch (e) {}

    isDateBeingCreated.value = false;
}
</script>
