<script setup lang="ts">
import { toRefs } from 'vue';
import { Profile, ExtendedSpace, Proposal } from '@/helpers/interfaces';

import { useUsername, useApp } from '@/composables';

const { domain } = useApp();

const props = defineProps<{
  address: string;
  space?: ExtendedSpace;
  proposal?: Proposal;
  profile?: Profile;
  hideAvatar?: boolean;
  widthClass?: string;
}>();

const { profile, address } = toRefs(props);
const { username } = useUsername(address, profile);
</script>

<template>
  <PopoverHoverProfile
    :address="address"
    :profile="profile"
    :proposal="proposal"
    :space="space"
  >
    <BaseLink
      :link="
        domain
          ? `https://snapshot.org/#/profile/${address}`
          : { name: 'profileActivity', params: { address } }
      "
      hide-external-icon
      @click.stop=""
    >
      <div :class="[widthClass, 'flex flex-nowrap items-center space-x-1']">
        <AvatarUser v-if="!hideAvatar" :address="address" size="18" />
        <span class="w-full cursor-pointer truncate text-skin-link">
          {{ username }}
        </span>
        <BaseBadge :address="address" :members="space?.members" />
      </div>
    </BaseLink>
  </PopoverHoverProfile>
</template>
