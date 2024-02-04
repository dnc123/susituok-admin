<template>
    <div class="pa-4">
        <p class="text-h3 mb-5">
            {{ props.heading }}
        </p>

        <slot />
    </div>
</template>

<script lang="ts" setup>
const props = defineProps({
    heading: {
        type: String,
        required: true,
    },
});
</script>
