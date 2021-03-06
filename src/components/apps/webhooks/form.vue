<template>
    <container-template>
        <tabs-menu slot="tab-menu" />
        <div slot="tab-content" class="row webhooks-section">
            <div class="col">
                <h5>{{ title }}</h5>
                <div class="row">
                    <div class="col-12 col-sm">
                        <label>Event</label>
                        <multiselect
                            v-model="selectedWebhook"
                            v-validate="'required:true'"
                            :options="webhooks"
                            data-vv-as="event"
                            data-vv-name="event"
                            label="name"
                            track-by="id"
                            @input="setWebhook"
                        />
                        <span class="text-danger">{{ errors.first("event") }}</span>
                    </div>
                    <div class="col-12 col-sm">
                        <label>Format</label>
                        <multiselect
                            v-model="selectedFormat"
                            v-validate="'required:true'"
                            :show-labels="false"
                            :options="webhooksFormats"
                            data-vv-as="webhook format"
                            data-vv-name="webhook format"
                            @input="setFormat"
                        />
                        <span class="text-danger">{{ errors.first("webhook format") }}</span>
                    </div>
                    <div class="col-12 col-sm">
                        <label>Method</label>
                        <multiselect
                            v-model="selectedMethod"
                            v-validate="'required:true'"
                            :show-labels="false"
                            :options="webhooksMethods"
                            data-vv-as="method"
                            data-vv-name="method"
                            @input="setWebhookMethod"
                        />
                        <span class="text-danger">{{ errors.first("method") }}</span>
                    </div>
                </div>
                <div class="row">
                    <div class="col">
                        <div class="form-group  required">
                            <label>URL</label>
                            <input
                                v-model="webhookData.url"
                                v-validate="'required:true|url:require_protocol'"
                                data-vv-as="url"
                                data-vv-name="url"
                                class="form-control"
                                type="text"
                                name="url"
                            >
                            <span class="text-danger">{{ errors.first('url') }}</span>
                        </div>
                    </div>
                </div>
                <div class="row">
                    <div class="col-12 col-xl d-flex justify-content-end mt-2">
                        <button :disabled="isLoading" class="btn btn-danger mr-2" @click="triggerCancel">
                            Cancel
                        </button>
                        <button :disabled="isLoading" class="btn btn-primary" @click="verifyFields">
                            Save
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </container-template>
</template>

<script>
import crudMixins from "../../../mixins/crudMixins";
import ContainerTemplate from "../../../container";
import TabsMenu from "../tabs";

export default {
    name: "Form",
    components: {
        ContainerTemplate,
        TabsMenu
    },
    mixins: [
        crudMixins
    ],
    data() {
        return {
            isLoading: false,
            selectedFormat: null,
            selectedMethod: null,
            selectedWebhook: null,
            webhookData: {},
            webhooksFormats:[
                "JSON",
                "XML"
            ],
            webhooks:[
            ],
            webhooksMethods:[
                "GET",
                "POST",
                "PUT",
                "DELETE"
            ]
        }
    },
    computed: {
        isEditWebhook() {
            return this.$route.name === "settingsAppsWebhooksFormEdit";
        },
        title() {
            let title = "New Webhook"
            if (this.isEditWebhook) {
                title = "Edit Webhook";
            }
            return title;
        }

    },
    watch:{
        "webhookData.format"() {
            this.setInitialFormat();
        },
        "webhookData.webhooks_id"() {
            this.setInitialWebhook();
        },
        "webhookData.method"() {
            this.setInitialMethod();
        },
        "route.params.id"() {
            //this.getUserWebhook();
        },
        "webhooks":{
            handler() {
                this.setInitialWebhook();
            },
            deep:true
        }
    },
    mounted() {
        this.getWebhooks();

        if (this.isEditWebhook) {
            this.getUserWebhook();
        }
    },
    methods: {
        setFormat(value) {
            this.webhookData.format = value;
        },
        setWebhookMethod(value) {
            this.webhookData.method = value;
        },
        setWebhook(value) {
            this.webhookData.webhooks_id = value.id;
        },
        setInitialFormat() {
            this.selectedFormat = this.webhooksFormats.find(format => format == this.webhookData.format);
        },
        setInitialMethod() {
            this.selectedMethod = this.webhooksMethods.find(method => method == this.webhookData.method);
        },
        setInitialWebhook() {
            this.selectedWebhook = this.webhooks.find(webhook => webhook.id == this.webhookData.webhooks_id);
        },
        getUserWebhook() {
            axios({
                url: `/user-webhooks/${this.$route.params.id}`
            }).then(({
                data
            }) => this.webhookData = data)
                .catch((error) => {
                    this.$notify({
                        group: null,
                        title: "Error",
                        text: error.response.data.errors.message,
                        type: "error"
                    });
                    this.$router.push({
                        name: "settingsAppsWebhooksList"
                    });
                });
        },
        verifyFields() {
            let dialogProps = {
                title: "Create Webhook!",
                message: `Did you want to create a new Webhook?`
            };

            if (this.isEditWebhook) {
                dialogProps = {
                    title: "Edit Webhook!",
                    message: `Did you want to Edit this Webhook?`
                };
            }
            if (this.errors.items.length) {
                this.$notify({
                    title: this.errors.items[0].msg,
                    text: `Please verify the ${this.errors.items[0].field}`,
                    type: "warn"
                });
            } else {
                this.validateFields(dialogProps);
            }
        },
        validateFields(modalProps) {
            this.$validator.validate().then(result => {
                if (result) {
                    this.$modal.show("basic-modal", {
                        ...modalProps,
                        buttons: [{
                            title: "Accept",
                            class: "btn-primary",
                            handler: () => {
                                this.$modal.hide("basic-modal");
                                this.save();
                            }
                        }, {
                            title: "Cancel",
                            class: "btn-danger",
                            handler: () => {
                                this.$modal.hide("basic-modal");
                            }
                        }]
                    });
                }
            });
        },
        save() {
            let url = "/user-webhooks";
            let method = "POST";
            let data = this.prepareData();

            if (this.isEditWebhook) {
                url = `/user-webhooks/${this.webhookData.id}`;
                method = "PUT";
                data = this.webhookData;
            }

            if (!this.isLoading) {
                this.sendRequest(url, method, data);
            }
        },
        sendRequest(url, method, data) {
            this.isLoading = true;
            axios({
                url,
                method,
                data
            }).then(() => {
                this.$notify({
                    group: null,
                    title: "Confirmation",
                    text: "Your information has been updated!",
                    type: "success"
                });
                this.cancel();
            }).catch((error) => {
                this.$notify({
                    group: null,
                    title: "Error",
                    text: error.response.data.errors.message,
                    type: "error"
                });
            }).finally(() => {
                this.isLoading = false;
            });
        },
        cancel() {
            this.$router.push({
                name: "settingsAppsWebhooksList"
            });
        },
        prepareData() {
            const data = new FormData();

            Object.keys(this.webhookData).forEach((field) => {
                data.append(field, this.webhookData[field]);
            });

            return data;
        },
        getWebhooks() {
            axios({
                url: "/webhooks"
            }).then(({ data }) => {
                this.webhooks = data;
            }).catch(() => {
                this.webhooks = [];
            });
        }
    }
}
</script>
