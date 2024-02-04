<template>
    <v-bottom-navigation>
        <router-link
            v-for="link in links"
            :key="link.name"
            :to="link.to"
        >
            <v-btn class="h-100">
                <v-icon>{{ link.icon }}</v-icon>
                <span>{{ link.name }}</span>
            </v-btn>
        </router-link>
    </v-bottom-navigation>
</template>

<script setup>
const links = [
    {
        icon: 'mdi-briefcase',
        to: '/',
        name: 'Veikla',
    },

    {
        icon: 'mdi-finance',
        to: '/finansai',
        name: 'Finansai',
    },

    {
        icon: 'mdi-calendar',
        to: '/kalendorius',
        name: 'Kalendorius',
    },

    {
        icon: 'mdi-link',
        to: '/nuorodos',
        name: 'Nuorodos',
    },

    {
        icon: 'mdi-wrench',
        to: '/irankiai',
        name: 'Irankiai',
    },

    {
        icon: 'mdi-store',
        to: '/krautuve',
        name: 'Krautuve',
    },
];
</script>
