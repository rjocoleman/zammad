<!-- Copyright (C) 2012-2024 Zammad Foundation, https://zammad-foundation.org/ -->

<script setup lang="ts">
import { computed } from 'vue'

import { useTicketView } from '#shared/entities/ticket/composables/useTicketView.ts'

import { useFlyout } from '#desktop/components/CommonFlyout/useFlyout.ts'
import type { MenuItem } from '#desktop/components/CommonPopoverMenu/types.ts'
import CommonSectionCollapse from '#desktop/components/CommonSectionCollapse/CommonSectionCollapse.vue'
import { useChangeCustomerMenuItem } from '#desktop/pages/ticket/components/TicketDetailView/actions/TicketChangeCustomer/useChangeCustomerMenuItem.ts'
import TicketAccountedTime from '#desktop/pages/ticket/components/TicketSidebar/TicketSidebarInformation/TicketSidebarInformationContent/TicketAccountedTime.vue'
import { useTicketInformation } from '#desktop/pages/ticket/composables/useTicketInformation.ts'
import {
  type TicketSidebarContentProps,
  TicketSidebarScreenType,
} from '#desktop/pages/ticket/types/sidebar.ts'

import TicketSidebarContent from '../TicketSidebarContent.vue'

import TicketSubscribers from './TicketSidebarInformationContent/TicketSubscribers.vue'
import TicketTags from './TicketSidebarInformationContent/TicketTags.vue'

const props = defineProps<TicketSidebarContentProps>()

const { ticket } = useTicketInformation()

const { isTicketAgent, isTicketEditable } = useTicketView(ticket)

const MERGE_FLYOUT_KEY = 'merge-ticket'

const { open: openTicketMergeFlyout } = useFlyout({
  name: MERGE_FLYOUT_KEY,
  component: () =>
    import(
      '#desktop/pages/ticket/components/TicketDetailView/actions/TicketMerge/TicketMergeFlyout.vue'
    ),
})

const actions = computed(() => {
  // :TODO find a way to provide the ticket via prop
  const availableActions: MenuItem[] = [
    {
      key: MERGE_FLYOUT_KEY,
      label: __('Merge'),
      icon: 'merge',
      show: () => isTicketAgent.value && isTicketEditable.value,
      onClick: () =>
        openTicketMergeFlyout({
          ticket,
          name: MERGE_FLYOUT_KEY,
        }),
    },
  ]

  // shared actions
  if (props.context.screenType === TicketSidebarScreenType.TicketDetailView) {
    const { customerChangeMenuItem } = useChangeCustomerMenuItem()
    availableActions.push(customerChangeMenuItem)
  }

  return availableActions
})
</script>

<template>
  <TicketSidebarContent
    :title="sidebarPlugin.title"
    :icon="sidebarPlugin.icon"
    :actions="actions"
  >
    <CommonSectionCollapse id="ticket-attributes" :title="__('Attributes')">
      <div
        id="ticketEditAttributeForm"
        data-test-id="ticket-edit-attribute-form"
      />
    </CommonSectionCollapse>

    <CommonSectionCollapse
      v-if="isTicketAgent"
      id="ticket-tags"
      :title="__('Tags')"
    >
      <TicketTags :ticket="ticket" :is-ticket-editable="isTicketEditable" />
    </CommonSectionCollapse>

    <CommonSectionCollapse
      v-if="ticket?.timeUnit && isTicketAgent"
      id="ticket-time-accounting"
      :title="__('Accounted Time')"
    >
      <TicketAccountedTime :ticket="ticket!" />
    </CommonSectionCollapse>

    <CommonSectionCollapse
      v-if="isTicketAgent"
      id="ticket-subscribers"
      :title="__('Subscribers')"
    >
      <TicketSubscribers :ticket="ticket" />
    </CommonSectionCollapse>
  </TicketSidebarContent>
</template>
