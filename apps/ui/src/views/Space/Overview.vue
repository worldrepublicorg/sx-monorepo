<script setup lang="ts">
import { _n, autoLinkText, getSocialNetworksLink } from '@/helpers/utils';
import { offchainNetworks } from '@/networks';
import { Space } from '@/types';

const PROPOSALS_LIMIT = 6;

const props = defineProps<{ space: Space }>();

const { setTitle } = useTitle();
const { isWhiteLabel } = useWhiteLabel();
const proposalsStore = useProposalsStore();

onMounted(() => {
  proposalsStore.fetchSummary(
    props.space.id,
    props.space.network,
    PROPOSALS_LIMIT
  );
});

const isOffchainSpace = offchainNetworks.includes(props.space.network);

const socials = computed(() => getSocialNetworksLink(props.space));

const proposalsRecord = computed(
  () => proposalsStore.proposals[`${props.space.network}:${props.space.id}`]
);

watchEffect(() => setTitle(props.space.name));
</script>

<template>
  <div>
    <div
      class="relative h-[156px] md:h-[140px] mb-[-86px] md:mb-[-70px] top-[-1px]"
    >
      <div class="size-full overflow-hidden">
        <SpaceCover :space="props.space" />
      </div>
      <div
        class="relative bg-skin-bg h-[16px] -top-3 rounded-t-[16px] md:hidden"
      />
      <div class="absolute right-4 top-4 flex gap-2">
        <UiTooltip title="New proposal">
          <UiButton
            :to="{
              name: 'space-editor',
              params: { space: `${space.network}:${space.id}` }
            }"
            class="!px-0 w-[46px]"
          >
            <IH-pencil-alt />
          </UiButton>
        </UiTooltip>
        <ButtonFollow :space="space" />
      </div>
    </div>
    <div class="px-4">
      <div class="mb-4 relative">
        <AppLink :to="{ name: 'space-overview' }">
          <SpaceAvatar
            :space="space"
            :size="90"
            class="relative mb-2 border-4 border-skin-bg !rounded-lg -left-1"
          />
        </AppLink>
        <div class="flex items-center">
          <h1 v-text="space.name" />
          <UiBadgeVerified
            v-if="!isWhiteLabel"
            class="ml-1 top-0.5"
            :verified="space.verified"
            :turbo="space.turbo"
          />
        </div>
        <div class="mb-3 flex flex-wrap gap-x-1 items-center">
          <div>
            <b class="text-skin-link">{{ _n(space.proposal_count) }}</b>
            proposals
          </div>
          <div>·</div>
          <div>
            <b class="text-skin-link">{{ _n(space.vote_count, 'compact') }}</b>
            votes
          </div>
          <template v-if="isOffchainSpace">
            <div>·</div>
            <div>
              <b class="text-skin-link">
                {{ _n(space.follower_count, 'compact') }}
              </b>
              followers
            </div>
          </template>
          <template v-if="!isWhiteLabel && space.parent">
            <div>·</div>
            <AppLink
              :to="{
                name: 'space-overview',
                params: {
                  space: `${space.parent.network}:${space.parent.id}`
                }
              }"
              class="flex space-x-1 items-center whitespace-nowrap"
            >
              <SpaceAvatar
                :space="space.parent"
                :size="22"
                class="rounded-md"
              />
              <span>{{ space.parent.name }}</span>
            </AppLink>
          </template>
        </div>
        <div
          v-if="space.about"
          class="max-w-[540px] text-skin-link text-[18px] leading-[26px] mb-3 break-words"
          v-html="autoLinkText(space.about)"
        />
        <div v-if="socials.length > 0" class="space-x-2 flex">
          <template v-for="social in socials" :key="social.key">
            <a
              :href="social.href"
              target="_blank"
              class="text-[#606060] hover:text-skin-link"
            >
              <component :is="social.icon" class="size-[26px]" />
            </a>
          </template>
        </div>
      </div>
    </div>
    <template v-if="!isWhiteLabel && space.children.length">
      <UiLabel :label="'Sub-spaces'" />
      <UiScrollerHorizontal gradient="md">
        <div class="px-4 py-3 flex gap-3 min-w-max">
          <SpacesListItem
            v-for="child in space.children"
            :key="child.id"
            :space="child"
            :show-about="false"
            class="w-[240px]"
          />
        </div>
      </UiScrollerHorizontal>
    </template>
    <div>
      <ProposalsList
        title="Proposals"
        :loading="!proposalsRecord?.summaryLoaded"
        :limit="PROPOSALS_LIMIT - 1"
        :proposals="proposalsRecord?.summaryProposals ?? []"
        :route="{
          name: 'space-proposals',
          linkTitle: 'See more'
        }"
      />
    </div>
  </div>
</template>
