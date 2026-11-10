<script setup>
import { computed } from 'vue';

const props = defineProps({
    currentPage: {
        type: Number,
        required: true,
    },
    totalPages: {
        type: Number,
        required: true,
    },
});

const emit = defineEmits(['page-change']);

const pages = computed(() => {
    const total = props.totalPages;
    const current = props.currentPage;

    if (total <= 7) {
        return Array.from({ length: total }, (_, index) => index + 1);
    }

    if (current <= 4) {
        return [1, 2, 3, 4, 5, '…', total];
    }

    if (current >= total - 3) {
        return [1, '…', total - 4, total - 3, total - 2, total - 1, total];
    }

    return [1, '…', current - 1, current, current + 1, '…', total];
});

function goTo(page) {
    if (page >= 1 && page <= props.totalPages && page !== props.currentPage) {
        emit('page-change', page);
    }
}
</script>

<template>
    <nav v-if="totalPages > 0" class="pagination is-centered mb-4" role="navigation" aria-label="Character pages">
        <button
            class="pagination-previous"
            type="button"
            :disabled="currentPage === 1"
            @click="goTo(currentPage - 1)"
        >
            Previous
        </button>
        <button
            class="pagination-next"
            type="button"
            :disabled="currentPage === totalPages"
            @click="goTo(currentPage + 1)"
        >
            Next
        </button>
        <ul class="pagination-list">
            <li v-for="(page, index) in pages" :key="`${page}-${index}`">
                <span v-if="page === '…'" class="pagination-ellipsis" aria-hidden="true">…</span>
                <button
                    v-else
                    class="pagination-link"
                    :class="{ 'is-current': page === currentPage }"
                    type="button"
                    :aria-label="`Go to page ${page}`"
                    :aria-current="page === currentPage ? 'page' : undefined"
                    @click="goTo(page)"
                >
                    {{ page }}
                </button>
            </li>
        </ul>
    </nav>
</template>
