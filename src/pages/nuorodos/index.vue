<template>
    <Page heading="Nuorodos">
        <v-table>
            <thead>
                <tr>
                    <th>
                        Pavadinimas
                    </th>

                    <th>
                        Nuoroda
                    </th>

                    <th>
                        Veiksmai
                    </th>
                </tr>
            </thead>

            <tbody>
                <tr
                    v-for="link in links"
                    :key="link._id"
                >
                    <td>
                        {{ link.name }}
                    </td>

                    <td>
                        <a
                            :href="link.link"
                            target="_blank"
                        >
                            {{ link.link }}
                        </a>
                    </td>

                    <td>
                        <v-menu>
                            <template v-slot:activator="{ props }">
                                <v-btn
                                    class="bg-red"
                                    :loading="isLinkBeingDeleted"
                                    v-bind="props"
                                >
                                    Istrinti
                                </v-btn>
                            </template>

                            <v-list>
                                <v-list-item class="cursor-pointer">
                                    Atsaukti
                                </v-list-item>

                                <v-list-item @click="deleteLink(link._id)">
                                    Istrinti
                                </v-list-item>
                            </v-list>
                        </v-menu>
                    </td>
                </tr>

                <tr>
                    <td class="pa-2">
                        <v-text-field
                            v-model="createLinkForm.name"
                            hide-details
                        />
                    </td>

                    <td class="pa-2">
                        <v-text-field
                            v-model="createLinkForm.link"
                            hide-details
                        />
                    </td>

                    <td>
                        <v-btn
                            class="bg-green"
                            :loading="isLinkBeingInserted"
                            @click="submitCreateLinkForm()"
                        >
                            Prideti
                        </v-btn>
                    </td>
                </tr>
            </tbody>
        </v-table>
    </Page>
</template>

<script setup>
import Page from "@/components/Page.vue";
import {ref, onMounted, reactive} from "vue";
import link__findAll from '@/SDK/requests/link__findAll';
import link__create from '@/SDK/requests/link__create';
import link__delete from '@/SDK/requests/link__delete';

const isLinkBeingInserted = ref(false);
const isLinkBeingDeleted = ref(false);
const links = ref([]);

const createLinkForm = reactive({
    name: '',
    link: '',
});

function resetCreateLinkForm() {
    createLinkForm.name = '';
    createLinkForm.link = '';
}

async function submitCreateLinkForm () {
    isLinkBeingInserted.value = true;

    try {
        await link__create(createLinkForm);
        resetCreateLinkForm();
        await loadLinks();
    } catch (e) {}

    isLinkBeingInserted.value = false;
}

async function deleteLink(targetLinkID) {
    isLinkBeingDeleted.value = true;

    try {
        await link__delete({
            linkID: targetLinkID,
        });

        await loadLinks();
    } catch (e) {}

    isLinkBeingDeleted.value = false;
}

async function loadLinks() {
    links.value = (await link__findAll({})).data;
}


onMounted(loadLinks);
</script>
