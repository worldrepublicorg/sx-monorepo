<script setup lang="ts">
import { validateForm } from '@/helpers/validation';
import { NetworkID } from '@/types';

const props = defineProps<{
  form: any;
  selectedNetworkId: NetworkID;
  title: string;
  description?: string;
}>();

const emit = defineEmits<{
  (e: 'errors', value: any);
}>();

const definition = computed(() => {
  return {
    type: 'object',
    title: 'SpaceSettings',
    additionalProperties: true,
    required: ['votingDelay', 'minVotingDuration', 'maxVotingDuration'],
    properties: {
      votingDelay: {
        type: 'number',
        format: 'duration',
        title: 'Voting delay'
      },
      minVotingDuration: {
        type: 'number',
        format: 'duration',
        title: 'Min. voting duration'
      },
      maxVotingDuration: {
        type: 'number',
        format: 'duration',
        title: 'Max. voting duration'
      }
    }
  };
});

const formErrors = computed(() => {
  const errors = validateForm(definition.value, props.form);

  if (props.form.minVotingDuration > props.form.maxVotingDuration) {
    errors.maxVotingDuration =
      'Max. voting duration must be equal to or greater than min. voting duration.';
  }

  emit('errors', errors);

  return errors;
});
</script>

<template>
  <UiContainerSettings :title="title" :description="description">
    <div class="s-box">
      <UiForm
        :model-value="form"
        :error="formErrors"
        :definition="definition"
      />
    </div>
  </UiContainerSettings>
</template>
