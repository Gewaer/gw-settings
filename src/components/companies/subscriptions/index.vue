<template>
    <container-template>
        <div slot="tab-content">
            <h5>Company Settings</h5>
            <tabs-menu slot="tab-menu" />
            <div class="row subscription-manager">
                <div class="col-12 col-lg col-xl-4">
                    <span class="title">Your plan subscription</span>
                    <subscription-details />
                </div>
                <div class="col-12 col-lg" />
            </div>
        </div>
    </container-template>
</template>

<script>
import ContainerTemplate from "../../../container";
import SubscriptionDetails from "./details";
import TabsMenu from "../tabs";

export default {
    name: "Subscriptions",
    components: {
        ContainerTemplate,
        SubscriptionDetails,
        TabsMenu
    }
}
</script>

<style lang="scss">
.subscription-manager {
    .title {
        margin-bottom: 20px;
        display: block;
    }
}
</style>
