<script lang="ts" setup>
import { useFormatter } from '@/stores';
import type { Tally } from '@/types';
import { computed } from '@vue/reactivity';
import type { PropType } from 'vue';

const props = defineProps({
  tally: { type: Object as PropType<Tally> },
  pool: {
    type: Object as PropType<{
      not_bonded_tokens: string;
      bonded_tokens: string;
    }>,
  },
});
const total = computed(() => props.pool?.bonded_tokens);
const format = useFormatter();
const yes = computed(() =>
  format.calculatePercent(props.tally?.yes, total.value)
);
const no = computed(() =>
  format.calculatePercent(props.tally?.no, total.value)
);
const abstain = computed(() =>
  format.calculatePercent(props.tally?.abstain, total.value)
);
const veto = computed(() =>
  format.calculatePercent(props.tally?.no_with_veto, total.value)
);
</script>

<template>
  <div class="progress rounded-[3px] h-6 text-xs flex items-center">
    <div
      class="h-6 bg-yes flex items-c
