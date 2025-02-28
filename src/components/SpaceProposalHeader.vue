<script setup lang="ts">
import { watch, computed } from 'vue';
import { ExtendedSpace, Proposal } from '@/helpers/interfaces';
import { useRouter } from 'vue-router';

import {
  useI18n,
  useProfiles,
  useSharing,
  useWeb3,
  useClient,
  useProposals,
  useSpaceCreateForm,
  useFlashNotification
} from '@/composables';

const props = defineProps<{
  space: ExtendedSpace;
  proposal: Proposal;
  isAdmin: boolean;
  isModerator: boolean;
}>();

const router = useRouter();

const { t } = useI18n();
const { send, isSending } = useClient();
const { removeSpaceProposal } = useProposals();
const { notify } = useFlashNotification();
const { web3Account } = useWeb3();

const isCreator = computed(() => props.proposal?.author === web3Account.value);

const threeDotItems = computed(() => {
  const items = [{ text: t('duplicate'), action: 'duplicate' }];
  if (props.isAdmin || props.isModerator || isCreator.value)
    items.push({ text: t('delete'), action: 'delete' });
  return items;
});

async function deleteProposal() {
  const result = await send(props.space, 'delete-proposal', {
    proposal: props.proposal
  });
  console.log('Result', result);
  if (result.id) {
    removeSpaceProposal(props.proposal.id);
    notify(['green', t('notify.proposalDeleted')]);
    router.push({ name: 'spaceProposals' });
  }
}

const {
  shareProposalTwitter,
  shareProposalLenster,
  shareToClipboard,
  shareProposal,
  sharingIsSupported,
  sharingItems
} = useSharing();

const { resetForm } = useSpaceCreateForm();

function selectFromThreedotDropdown(e) {
  if (!props.proposal) return;
  if (e === 'delete') deleteProposal();
  if (e === 'duplicate') {
    resetForm();
    router.push({
      name: 'spaceCreate',
      params: {
        key: props.proposal.space.id,
        step: 0,
        sourceProposal: props.proposal.id
      }
    });
  }
}

function selectFromShareDropdown(e) {
  if (e === 'shareProposalTwitter')
    return shareProposalTwitter(props.space, props.proposal);
  if (e === 'shareProposalLenster')
    return shareProposalLenster(props.space, props.proposal);
  if (e === 'shareToClipboard')
    return shareToClipboard(props.space, props.proposal);
}

const { profiles, loadProfiles } = useProfiles();

watch(
  () => props.proposal,
  () => {
    if (!props.proposal) return;
    loadProfiles([props.proposal.author]);
  }
);
</script>

<template>
  <h1
    class="mb-3 break-words text-xl leading-8 sm:text-2xl"
    v-text="proposal.title"
  />

  <div class="mb-4 flex flex-col sm:flex-row sm:space-x-1">
    <div class="mb-1 flex items-center sm:mb-0">
      <LabelProposalState :state="proposal.state" class="mr-2" />
      <LinkSpace :space-id="space.id" class="group text-skin-text">
        <div class="flex items-center">
          <AvatarSpace :space="space" size="28" />
          <span class="ml-2 group-hover:text-skin-link" v-text="space.name" />
        </div>
      </LinkSpace>
    </div>
    <div class="flex grow items-center space-x-1">
      <span v-text="$t('proposalBy')" />
      <BaseUser
        :address="proposal.author"
        :profile="profiles[proposal.author]"
        :space="space"
        :proposal="proposal"
        hide-avatar
      />
      <ButtonShare
        v-if="sharingIsSupported"
        @click="shareProposal(space, proposal)"
      />
      <BaseMenu
        v-else
        class="!ml-auto pl-3"
        :items="sharingItems"
        @select="selectFromShareDropdown"
      >
        <template #button>
          <ButtonShare />
        </template>
        <template #item="{ item }">
          <div class="flex items-center gap-2">
            <i-s-twitter v-if="item.extras.icon === 'twitter'" />
            <i-s-lenster
              v-if="item.extras.icon === 'lenster'"
              class="mr-1 text-sm text-skin-text"
            />
            <i-ho-link v-if="item.extras.icon === 'link'" />
            {{ item.text }}
          </div>
        </template>
      </BaseMenu>
      <BaseMenu
        class="md:ml-2"
        :items="threeDotItems"
        @select="selectFromThreedotDropdown"
      >
        <template #button>
          <div>
            <BaseButtonIcon :loading="isSending">
              <i-ho-dots-horizontal />
            </BaseButtonIcon>
          </div>
        </template>
        <template #item="{ item }">
          <div class="flex items-center gap-2">
            <i-ho-document-duplicate v-if="item.action === 'duplicate'" />
            <i-ho-trash v-if="item.action === 'delete'" />
            {{ item.text }}
          </div>
        </template>
      </BaseMenu>
    </div>
  </div>
</template>
