<script>
  import { goto } from "@roxi/routify"
  import { datasources, queries, tables } from "stores/backend"
  import { notifications } from "@budibase/bbui"
  import { ActionMenu, MenuItem, Icon } from "@budibase/bbui"
  import ConfirmDialog from "components/common/ConfirmDialog.svelte"
  import UpdateDatasourceModal from "components/backend/DatasourceNavigator/modals/UpdateDatasourceModal.svelte"

  export let datasource

  let confirmDeleteDialog
  let updateDatasourceDialog

  async function deleteDatasource() {
    let wasSelectedSource = $datasources.selected
    if (!wasSelectedSource && $queries.selected) {
      const queryId = $queries.selected
      wasSelectedSource = $datasources.list.find(ds =>
        queryId.includes(ds._id)
      )?._id
    }
    const wasSelectedTable = $tables.selected
    await datasources.delete(datasource)
    notifications.success("Datasource deleted")
    // navigate to first index page if the source you are deleting is selected
    const entities = Object.values(datasource?.entities || {})
    if (
      wasSelectedSource === datasource._id ||
      (entities &&
        entities.find(entity => entity._id === wasSelectedTable?._id))
    ) {
      $goto("./datasource")
    }
  }
</script>

<ActionMenu>
  <div slot="control" class="icon">
    <Icon size="S" hoverable name="MoreSmallList" />
  </div>
  <MenuItem icon="Edit" on:click={updateDatasourceDialog.show}>Edit</MenuItem>
  <MenuItem icon="Delete" on:click={confirmDeleteDialog.show}>Delete</MenuItem>
</ActionMenu>

<ConfirmDialog
  bind:this={confirmDeleteDialog}
  okText="Delete Datasource"
  onOk={deleteDatasource}
  title="Confirm Deletion"
>
  Are you sure you wish to delete the datasource
  <i>{datasource.name}?</i>
  This action cannot be undone.
</ConfirmDialog>
<UpdateDatasourceModal {datasource} bind:this={updateDatasourceDialog} />

<style>
  div.icon {
    display: flex;
    flex-direction: row;
    justify-content: flex-end;
    align-items: center;
  }
</style>
