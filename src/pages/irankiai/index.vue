<template>
    <Page heading="Irankiai">
        <v-card>
            <v-card-title class="d-flex justify-space-between">
                Kuro skaiciuokle
            </v-card-title>

            <div class="pa-2">
                <v-row class="mb-5">
                    <v-col>
                        <v-text-field
                            v-model.number="fuelConsumptionPer100Km"
                            label="Kuro sanaudos/100km"
                            hide-details
                            type="number"
                        />
                    </v-col>

                    <v-col>
                        <v-text-field
                            v-model.number="fuelPricePerLiter"
                            label="Litro kuro kaina"
                            hide-details
                            type="number"
                        />
                    </v-col>

                    <v-col>
                        <v-text-field
                            v-model.number="travelDistanceKm"
                            label="Vaziuojamas atstumas(km)"
                            hide-details
                            type="number"
                        />
                    </v-col>
                </v-row>

                <p class="text-center text-h4">
                    Kaina: € {{ tripPrice }}
                </p>
            </div>
        </v-card>
    </Page>
</template>

<script setup>
import Page from "@/components/Page.vue";
import {computed, ref} from "vue";

const fuelConsumptionPer100Km = ref(0);
const fuelPricePerLiter = ref(0);
const travelDistanceKm = ref(0);

const tripPrice = computed(() => {
    const fuelNeededLiters = (fuelConsumptionPer100Km.value / 100) * travelDistanceKm.value;
    const totalPrice = fuelNeededLiters * fuelPricePerLiter.value;

    return totalPrice.toFixed(2);
});
</script>
