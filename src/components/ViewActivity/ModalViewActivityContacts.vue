<template>
    <v-dialog
        width="900"
        v-model="isModalOpened"
    >
        <v-card>
            <v-table hover>
                <thead>
                    <tr>
                        <th>
                            Vardas pavarde
                        </th>

                        <th>
                            El. pastas/tel. nr.
                        </th>

                        <th>
                            Uzsake is
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
                                v-model="createContactForm.fullName"
                                hide-details
                                placeholder="Donatas Petrauskas"
                            />
                        </td>

                        <td class="pa-4">
                            <v-text-field
                                v-model="createContactForm.emailPhone"
                                hide-details
                                placeholder="petrdonatas@gmail.com"
                            />
                        </td>

                        <td class="pa-4">
                            <v-text-field
                                v-model="createContactForm.cameFrom"
                                hide-details
                                placeholder="Instagramo"
                            />
                        </td>

                        <td class="pa-4">
                            <v-text-field
                                v-model="createContactForm.note"
                                hide-details
                                placeholder="Sunkiai sukalbamas"
                            />
                        </td>

                        <td>
                            <v-btn
                                class="bg-green"
                                :loading="isContactBeingCreated"
                                @click="submitCreateContactForm()"
                            >
                                Prideti
                            </v-btn>
                        </td>
                    </tr>

                    <tr
                        v-for="contact in sortedContacts"
                    >
                        <td>{{ contact.fullname }}</td>
                        <td>{{ contact.emailPhone }}</td>
                        <td>{{ contact.cameFrom }}</td>
                        <td>{{ contact.note }}</td>

                        <td>
                            <v-menu>
                                <template v-slot:activator="{ props }">
                                    <v-btn
                                        class="bg-red"
                                        :loading="isContactBeingDeleted"
                                        v-bind="props"
                                    >
                                        Istrinti
                                    </v-btn>
                                </template>

                                <v-list>
                                    <v-list-item class="cursor-pointer">
                                        Atsaukti
                                    </v-list-item>

                                    <v-list-item @click="deleteActivityContact(contact)">
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
import activity__deleteContact from '@/SDK/requests/activity__deleteContact';
import activity__createContact from '@/SDK/requests/activity__createContact';

const isModalOpened = defineModel();

const isContactBeingCreated = ref(false);
const isContactBeingDeleted = ref(false);

const createContactForm = ref({
    fullName: '',
    emailPhone: '',
    cameFrom: '',
    note: '',
});

const props = defineProps({
    activity: {
        type: Object,
        required: true,
    },
});

const sortedContacts = computed(() => {
    const allContacts = JSON.parse(JSON.stringify(props.activity.contacts));

    allContacts.sort((a, b) => {
        if (a.createdAt < b.createdAt) {
            return 1;
        }

        if (a.createdAt > b.createdAt) {
            return -1;
        }

        return 0;
    });

    return allContacts;
});

async function deleteActivityContact(payment) {
    isContactBeingDeleted.value = true;

    try {
        await activity__deleteContact({
            activityID: props.activity._id,
            createdAt: payment.createdAt,
        });

        props.activity.contacts = props.activity.contacts.filter(({ createdAt }) => {
            return payment.createdAt !== createdAt;
        });
    } catch (e) {}

    isContactBeingDeleted.value = false;
}

function resetCreateContactForm() {
    createContactForm.value.fullName = '';
    createContactForm.value.emailPhone = '';
    createContactForm.value.cameFrom = '';
    createContactForm.value.note = '';
}

async function submitCreateContactForm() {
    isContactBeingCreated.value = true;

    try {
        const newContact = (await activity__createContact({
            activityID: props.activity._id,
            fullname: createContactForm.value.fullName,
            note: createContactForm.value.note,
            cameFrom: createContactForm.value.cameFrom,
            emailPhone: createContactForm.value.emailPhone,
            createdAt: Date.now(),
        })).data;

        props.activity.contacts.unshift(newContact);
        resetCreateContactForm();
    } catch (e) {}

    isContactBeingCreated.value = false;
}
</script>
