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
                            Mokestis
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
                                v-model.number="createPaymentForm.amount"
                                hide-details
                                type="number"
                                placeholder="230.54"
                            />
                        </td>

                        <td class="pa-4">
                            <v-text-field
                                v-model="createPaymentForm.note"
                                hide-details
                                placeholder="Avansas..."
                            />
                        </td>

                        <td>
                            <v-btn
                                class="bg-green"
                                :loading="isPaymentBeingCreated"
                                @click="submitCreatePaymentForm()"
                            >
                                Prideti
                            </v-btn>
                        </td>
                    </tr>

                    <tr
                        v-for="payment in sortedPayments"
                        :class="{
                            'bg-red-lighten-5': payment.amount < 0,
                            'bg-green-lighten-5': payment.amount > 0,
                        }"
                    >
                        <td>€ {{ payment.amount }}</td>
                        <td>{{ payment.note }}</td>

                        <td>
                            <v-menu>
                                <template v-slot:activator="{ props }">
                                    <v-btn
                                        class="bg-red"
                                        :loading="isPaymentBeingDeleted"
                                        v-bind="props"
                                    >
                                        Istrinti
                                    </v-btn>
                                </template>

                                <v-list>
                                    <v-list-item class="cursor-pointer">
                                        Atsaukti
                                    </v-list-item>

                                    <v-list-item @click="deleteActivityPayment(payment)">
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
import {ref, computed} from 'vue';
import activity__createPayment from '@/SDK/requests/activity__createPayment';
import activity__deletePayment from '@/SDK/requests/activity__deletePayment';

const isModalOpened = defineModel();

const isPaymentBeingCreated = ref(false);
const isPaymentBeingDeleted = ref(false);

const createPaymentForm = ref({
    amount: 0,
    note: '',
});

const props = defineProps({
    activity: {
        type: Object,
        required: true,
    },
});

const sortedPayments = computed(() => {
    const allPayments = JSON.parse(JSON.stringify(props.activity.payments));

    allPayments.sort((a, b) => {
        if (a.createdAt < b.createdAt) {
            return 1;
        }

        if (a.createdAt > b.createdAt) {
            return -1;
        }

        return 0;
    });

    return allPayments;
});

async function deleteActivityPayment(payment) {
    isPaymentBeingDeleted.value = true;

    try {
        await activity__deletePayment({
            activityID: props.activity._id,
            createdAt: payment.createdAt,
        });

        props.activity.payments = props.activity.payments.filter(({ createdAt }) => {
            return payment.createdAt !== createdAt;
        });
    } catch (e) {}

    isPaymentBeingDeleted.value = false;
}

function resetCreatePaymentForm() {
    createPaymentForm.value.amount = 0;
    createPaymentForm.value.note = '';
}

async function submitCreatePaymentForm() {
    isPaymentBeingCreated.value = true;

    try {
        const newPayment = (await activity__createPayment({
            activityID: props.activity._id,
            amount: createPaymentForm.value.amount,
            note: createPaymentForm.value.note,
            createdAt: Date.now(),
        })).data;

        props.activity.payments.unshift(newPayment);
        resetCreatePaymentForm();
    } catch (e) {}

    isPaymentBeingCreated.value = false;
}
</script>
