<template>
    <container-template>
        <tabs-menu slot="tab-menu" />
        <div slot="tab-content" class="row webhooks-section">
            <div class="col">
                <h5>
                    Webhooks
                    <router-link :to="{ name: 'settingsAppsWebhooksForm' }" class="btn btn-primary">
                        Create
                    </router-link>
                </h5>
                <div class="table-responsive">
                    <vuetable
                        ref="Vuetable"
                        :append-params="appendParams"
                        :fields="webhooksFields"
                        :http-fetch="getTableData"
                        api-url="/user-webhooks"
                        class="table table-hover table-condensed"
                        pagination-path=""
                    >
                        <template slot="actions" slot-scope="props">
                            <div class="d-flex align-items-center justify-content-end">
                                <button class="btn btn-complete m-l-5" @click="editWebhook(props.rowData)">
                                    <i class="fa fa-edit" aria-hidden="true" />
                                </button>
                                <button
                                    class="btn btn-danger m-l-5"
                                    @click="confirmDeleteWebhook(props.rowData)"
                                >
                                    <i class="fa fa-trash" aria-hidden="true" />
                                </button>
                            </div>
                        </template>
                    </vuetable>
                </div>
            </div>
        </div>
    </container-template>
</template>

<script>
import vuexMixins from "../../../mixins/vuexMixins";
import listMixins from "../../../mixins/listMixins";
import ContainerTemplate from "../../../container";
import TabsMenu from "../tabs";

export default {
    name: "List",
    components: {
        ContainerTemplate,
        TabsMenu
    },
    mixins: [
        vuexMixins,
        listMixins
    ],
    data() {
        return {
            appendParams:{
                format: "true",
                relationships: "webhooks",
                q: "(is_deleted:0)"
            },
            webhooksFields: [{
                name: "webhooks.name",
                title: "Name (Event)",
                width: "50%"

            }, {
                name: "format",
                sortField: "format",
                width: "20%"
            }, {
                name: "url",
                sortField: "url",
                width: "30%"
            }, {
                name: "actions",
                title: "Actions",
                titleClass: "table-actions",
                dataClass: "table-actions"
            }]
        }
    },
    methods:{
        getTableData(apiUrl, options) {
            return axios({
                url: apiUrl,
                params: options.params
            });
        },
        editWebhook(userWebhook) {
            this.$router.push({
                name: "settingsAppsWebhooksFormEdit",
                params: {
                    id: userWebhook.id
                }
            });
        },
        confirmDeleteWebhook(userWebhook) {
            this.$modal.show("basic-modal", {
                title: "Delete Webhook",
                // message:`Did you want to delete ${userWebhook.webhook.name} webhook ?`,
                message: "Did you want to delete webhook?",
                buttons: [{
                    title: "Accept",
                    class: "btn-success",
                    handler: () => {
                        this.$modal.hide("basic-modal");
                        this.disableWebhook(userWebhook);
                    }
                }, {
                    title: "Cancel",
                    class: "btn-danger",
                    handler: () => {
                        this.$modal.hide("basic-modal");
                    }
                }]
            });
        },
        disableWebhook(userWebhook) {
            axios({
                url: `/user-webhooks/${userWebhook.id}`,
                method: "DELETE"
            }).then(() => {
                this.$notify({
                    group: null,
                    title: "Deleted",
                    text: "The Webhook has been deleted",
                    type: "success"
                });
            }).catch((error) => {
                this.$notify({
                    group: null,
                    title: "Error",
                    text: error.response.data.errors.message,
                    type: "error"
                });
            }).finally(() => {
                this.$refs.Vuetable.reload();
            });
        }
    }
};
</script>
