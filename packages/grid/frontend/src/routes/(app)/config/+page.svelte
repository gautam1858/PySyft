<script lang="ts">
  import { metadata, user } from '$lib/store';
  import Avatar from '$lib/components/Avatar.svelte';
  import Tabs from '$lib/components/Tabs.svelte';
  import { getInitials } from '$lib/utils';
  import Badge from '$lib/components/Badge.svelte';
  import Button from '$lib/components/Button.svelte';
  import Modal from '$lib/components/Modal.svelte';
  import XIcon from '$lib/components/icons/XIcon.svelte';
  import Dialog from '$lib/components/Dialog.svelte';
  import Input from '$lib/components/Input.svelte';
  import ButtonGhost from '$lib/components/ButtonGhost.svelte';
  import TextArea from '$lib/components/TextArea.svelte';
  import { getMetadata, updateMetadata } from '$lib/api/metadata';

  let tabs = [
    { label: 'Domain', id: 'tab1' }
    // { label: 'Connection', id: 'tab2' }
    // { label: 'Permissions', id: 'tab3' },
  ];

  let currentTab = tabs[0].id;

  $: profileInformation = [
    { label: 'Domain name', id: 'domain_name', value: $metadata?.name },
    { label: 'Organization', id: 'organization', value: $metadata?.organization },
    { label: 'Description', id: 'description', value: $metadata?.description }
  ];

  let openModal: 'name' | 'organization' | 'description' | null = null;

  const onClose = () => (openModal = null);

  let { name, organization, description } = $metadata ?? {};

  const handleUpdate = async () => {
    let newMetadata = {
      name: openModal === 'domain_name' ? name : null,
      organization: openModal === 'organization' ? organization : null,
      description: openModal === 'description' ? description : null
    };

    newMetadata = Object.fromEntries(Object.entries(newMetadata).filter(([_, v]) => v));

    try {
      await updateMetadata(newMetadata);
      const updatedMetadata = await getMetadata();
      metadata.set(updatedMetadata);
      onClose();
    } catch (error) {
      console.log(error);
    }
  };

  $: domainInitials = getInitials($metadata?.name);
  $: userInitials = getInitials($user?.name);
</script>

<div>
  <Tabs {tabs} bind:active={currentTab} />
  <section class="pt-9 pl-[60px] pr-10 pb-20 flex flex-col tablet:flex-row gap-11">
    {#if !$metadata}
      Loading...
    {:else}
      <span class="block w-32 h-32 p-4">
        <Avatar blackBackground initials={domainInitials} />
      </span>
      <div class="w-full divide-y divide-gray-200">
        <div class="w-full flex flex-col py-4 gap-3">
          <h3 class="all-caps">Profile information</h3>
          {#each profileInformation as { label, id, value }}
            <div class="flex flex-col gap-2 pt-2 pb-4">
              <h4 class="text-xl capitalize">{label}</h4>
              <p data-test-id={id} class:text-gray-600={!value}>{value ?? 'Empty'}</p>
              <span>
                <button class="link capitalize" on:click={() => (openModal = id)}>
                  Change {label}
                </button>
              </span>
            </div>
          {/each}
        </div>
        <div class="w-full flex flex-col py-4 gap-3">
          <h3 class="all-caps">System information</h3>
          <div class="flex flex-col desktop:flex-row gap-2 items-center">
            <div class="flex gap-2 items-center w-full">
              <span class="block w-16 h-16">
                <Avatar noOutline initials={userInitials} />
              </span>
              <div class="flex flex-col gap-1">
                <h4 class="text-xl capitalize font-bold">{$user?.name}</h4>
                <p>{$user?.email}</p>
              </div>
            </div>
            <div class="flex flex-col gap-1 pt-2 pb-4 w-full">
              <div class="flex gap-1 flex-wrap items-center">
                <h4 class="text-gray-400 all-caps text-sm">ID#:</h4>
                <Badge variant="gray">{$metadata.id.value}</Badge>
              </div>
              <div class="flex gap-1 items-center">
                <h4 class="text-gray-400 all-caps text-sm">Deployed on:</h4>
                <p class="text-gray-800">{$metadata.deployed_on}</p>
              </div>
            </div>
          </div>
        </div>
        <!--- For future use
        <div class="w-full flex flex-col py-4 gap-3">
          <h3 class="all-caps">Caution</h3>
          <div
            class="w-full flex flex-col p-8 border-b border-primary-100 gap-2 bg-gray-50 rounded-[14px]"
          >
            <span class="inline-flex gap-3 items-center">
              <h4 class="text-lg font-bold">Remote Execution</h4>
              <Badge variant="gray"><span class="capitalize">Turned off</span></Badge>
            </span>
            <p>
              When remote execution is turned on for the domain, it means that you are allowing
              PySyft to execute code submitted by users as is against the real private data instead
              of the mock data when a request is approved. If this is a third-party user please
              review the function and policy code carefully before approving this request. You can
              turn off "Remote Execution" for your domain by going to your "Domain Settings" or
              clicking the link below.
            </p>
            <span>
              <Button variant="primary-outline">Turn On</Button>
            </span>
          </div>
        </div>
        -->
      </div>
    {/if}
  </section>
</div>

<Dialog bind:open={openModal}>
  <Modal>
    <div slot="header" class="w-full text-right">
      <button on:click={onClose}>
        <XIcon class="w-6 h-6" />
      </button>
    </div>
    <div slot="body">
      {#if openModal === 'domain_name'}
        <Input label="Domain name" required bind:value={name} id="name" />
      {:else if openModal === 'organization'}
        <Input label="Organization" bind:value={organization} id="institution" />
      {:else if openModal === 'description'}
        <TextArea label="Description" bind:value={description} id="description" />
      {/if}
    </div>
    <div class="flex w-full justify-end" slot="button-group">
      <div class="w-full justify-end flex px-4 gap-4">
        <ButtonGhost on:click={onClose}>Cancel</ButtonGhost>
        <Button type="submit" variant="secondary" on:click={handleUpdate}>Save</Button>
      </div>
    </div>
  </Modal>
</Dialog>
