<script lang="ts" setup>
import {
  useBlockchain,
  useFormatter,
  useStakingStore,
  useTxDialog,
} from '@/stores';
import { select } from '@/components/dynamic/index';
import type { PaginatedProposals } from '@/types';
import ProposalProcess from './ProposalProcess.vue';
import type { PropType } from 'vue';
import { ref } from 'vue';
const dialog = useTxDialog();
defineProps({
  proposals: { type: Object as PropType<PaginatedProposals> },
});

const format = useFormatter();
const staking = useStakingStore();
const chain = useBlockchain();
function showType(v: string) {
  if (v) {
    return v.substring(v.lastIndexOf('.') + 1);
  }
  return v;
}

const statusMap: Record<string, string> = {
  PROPOSAL_STATUS_VOTING_PERIOD: 'VOTING',
  PROPOSAL_STATUS_PASSED: 'PASSED',
  PROPOSAL_STATUS_REJECTED: 'REJECTED',
};
const voterStatusMap: Record<string, string> = {
  VOTE_OPTION_NO_WITH_VETO: '',
  VOTE_OPTION_YES: 'success',
  VOTE_OPTION_NO: 'error',
  VOTE_OPTION_ABSTAIN: 'warning',
};

const proposalInfo = ref();
</script>
<template>
  <!-- Arka Planı Açık Gri, Panelleri Siyah Yaptık -->
  <div class="bg-[#f0f0f0] dark:bg-[#000000] rounded text-sm">
    <table class="table-compact w-full table-fixed hidden lg:!table">
      <tbody>
        <tr v-for="(item, index) in proposals?.proposals" :key="index">
          <td class="px-4 w-20">
            <label
              for="proposal-detail-modal"
              class="text-main text-base hover:text-indigo-400 cursor-pointer"
              @click="proposalInfo = item"
            >
              #{{ item?.proposal_id }}</label
            >
          </td>
          <td class="w-full">
            <div>
              <RouterLink
                :to="`/${chain.chainName}/gov/${item?.proposal_id}`"
                class="text-main text-base mb-1 block hover:text-indigo-400 truncate"
              >
                {{ item?.content?.title || item?.title }}
              </RouterLink>
              <div
                v-if="item.content"
                class="bg-[#f0f0f0] text-[#9c6cff] dark:bg-black dark:text-gray-300 inline-block rounded-full px-2 py-[1px] text-xs mb-1"
              >
                {{ showType(item.content['@type']) }}
              </div>
            </div>
          </td>
          <td class="w-60">
            <ProposalProcess
              :pool="staking.pool"
              :tally="item.final_tally_result"
            ></ProposalProcess>
          </td>
          <td class="w-36">
            <div class="pl-4">
              <div
                class="flex items-center"
                :class="
                  statusMap?.[item?.status] === 'PASSED'
                    ? 'text-yes'
                    : statusMap?.[item?.status] === 'REJECTED'
                    ? 'text-no'
                    : 'text-info'
                "
              >
                <div
                  class="w-1 h-1 rounded-full mr-2"
                  :class="
                    statusMap?.[item?.status] === 'PASSED'
                      ? 'bg-yes'
                      : statusMap?.[item?.status] === 'REJECTED'
                      ? 'bg-no'
                      : 'bg-info'
                  "
                ></div>
                <div class="text-xs">
                  {{ statusMap?.[item?.status] || item?.status }}
                </div>
              </div>
              <div
                class="truncate col-span-2 md:!col-span-1 text-xs text-gray-500 dark:text-gray-400 text-right md:!flex md:!justify-start"
              >
                {{ format.toDay(item.voting_end_time, 'from') }}
              </div>
            </div>
          </td>

          <td v-if="statusMap?.[item?.status] === 'VOTING'" class="w-40">
            <div class="">
              <label
                for="vote"
                class="btn btn-xs btn-primary rounded-sm"
                @click="
                  dialog.open('vote', {
                    proposal_id: item?.proposal_id,
                  })
                "
              >
                <span v-if="item?.voterStatus !== 'VOTE_OPTION_NO_WITH_VETO'">{{
                  item?.voterStatus?.replace('VOTE_OPTION_', '')
                }}</span>

                <span v-else>Vote</span>
              </label>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
