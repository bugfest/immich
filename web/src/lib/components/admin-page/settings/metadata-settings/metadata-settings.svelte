<script lang="ts">
  import {
    notificationController,
    NotificationType,
  } from '$lib/components/shared-components/notification/notification';
  import { handleError } from '$lib/utils/handle-error';
  import { api, SystemConfigMetadataDto } from '@api';
  import { isEqual } from 'lodash-es';
  import { fade } from 'svelte/transition';
  import ConfirmEnableImportFaces from '../confirm-enable-import-faces.svelte';
  import SettingButtonsRow from '../setting-buttons-row.svelte';
  import SettingSwitch from '../setting-switch.svelte';
  import SettingAccordion from '../setting-accordion.svelte';
  import type { ResetOptions } from '$lib/utils/dipatch';

  export let metadataConfig: SystemConfigMetadataDto; // this is the config that is being edited
  export let disabled = false;

  let savedConfig: SystemConfigMetadataDto;
  let defaultConfig: SystemConfigMetadataDto;

  const handleReset = (detail: ResetOptions) => {
    if (detail.default) {
      resetToDefault();
    } else {
      reset();
    }
  };

  async function getConfigs() {
    [savedConfig, defaultConfig] = await Promise.all([
      api.systemConfigApi.getConfig().then((res) => res.data.metadata),
      api.systemConfigApi.getConfigDefaults().then((res) => res.data.metadata),
    ]);
  }

  let isConfirmOpen = false;
  let handleConfirm: (value: boolean) => void;

  const openConfirmModal = () => {
    return new Promise((resolve) => {
      handleConfirm = (value: boolean) => {
        isConfirmOpen = false;
        resolve(value);
      };
      isConfirmOpen = true;
    });
  };

  async function saveSetting() {
    try {
      const { data: current } = await api.systemConfigApi.getConfig();

      if (current.machineLearning.facialRecognition.enabled && !current.metadata?.importFaces && metadataConfig.importFaces) {
        const confirmed = await openConfirmModal();
        if (!confirmed) {
          return;
        }
      }

      const { data: updated } = await api.systemConfigApi.updateConfig({
        systemConfigDto: {
          ...current,
          metadata: metadataConfig,
        },
      });

      metadataConfig = { ...updated.metadata };
      savedConfig = { ...updated.metadata };

      notificationController.show({ message: 'Settings saved', type: NotificationType.Info });
    } catch (error) {
      handleError(error, 'Unable to save settings');
    }
  }

  async function reset() {
    const { data: resetConfig } = await api.systemConfigApi.getConfig();

    metadataConfig = { ...resetConfig.metadata };
    savedConfig = { ...resetConfig.metadata };

    notificationController.show({
      message: 'Reset settings to the recent saved settings',
      type: NotificationType.Info,
    });
  }

  async function resetToDefault() {
    const { data: configs } = await api.systemConfigApi.getConfigDefaults();

    metadataConfig = { ...configs.metadata };
    defaultConfig = { ...configs.metadata };

    notificationController.show({
      message: 'Reset password settings to default',
      type: NotificationType.Info,
    });
  }
</script>

{#if isConfirmOpen}
  <ConfirmEnableImportFaces on:cancel={() => handleConfirm(false)} on:confirm={() => handleConfirm(true)} />
{/if}

<div>
  {#await getConfigs() then}
    <div in:fade={{ duration: 500 }}>
      <SettingAccordion title="Import faces" subtitle="Settings for importing face regions from metadata" isOpen>
        <form autocomplete="off" on:submit|preventDefault>
          <div class="ml-4 mt-4 flex flex-col gap-4">
            <div class="ml-4">
              <SettingSwitch
                title="ENABLED"
                {disabled}
                subtitle="Import faces from image metadata"
                bind:checked={metadataConfig.importFaces}
              />

              <SettingButtonsRow
                on:reset={({ detail }) => handleReset(detail)}
                on:save={saveSetting}
                showResetToDefault={!isEqual(savedConfig, defaultConfig)}
                {disabled}
              />
            </div>
          </div>
        </form>
      </SettingAccordion>
    </div>
  {/await}
</div>
