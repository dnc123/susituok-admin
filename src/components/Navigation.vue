<template>
    <v-bottom-navigation density="default">
        <router-link
            v-for="link in links"
            :key="link.name"
            :to="link.to"
        >
            <v-btn
                class="h-100 text-blue"
                :active="route.path === link.to"
            >
                <v-icon>{{ link.icon }}</v-icon>
            </v-btn>
        </router-link>
    </v-bottom-navigation>
</template>

<script setup>
import {useRoute} from 'vue-router';

const route = useRoute();

const links = [
    {
        icon: 'mdi-briefcase',
        to: '/',
    },

    {
        icon: 'mdi-currency-eur',
        to: '/finansai',
    },

    {
        icon: 'mdi-calendar',
        to: '/kalendorius',
    },

    {
        icon: 'mdi-link',
        to: '/nuorodos',
    },

    {
        icon: 'mdi-wrench',
        to: '/irankiai',
    },

    {
        icon: 'mdi-store',
        to: '/krautuve',
    },
];
</script>
