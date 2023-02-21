<template>
    <div
        :class="{
            '' : !selectedGroup
        }"
        :id="group.key">
        
        <div class="h-full w-full relative">
            <div v-if="selectedGroup" class="absolute bg-white box-content flex h-8 items-center leading-normal overflow-hidden right-[3px] rounded-bl-lg shadow top-[3px] z-10">
            
                <div class="flex" v-if="!readonly">
                    <button
                        dusk="drag-group"
                        type="button"
                        class="group-control btn border-l border-gray-200 dark:border-gray-700 w-8 h-8 block nova-flexible-content-drag-button"
                        :title="__('Drag')"
                        >
                        <icon type="selector" class="align-top" width="16" height="16" />
                    </button>
                    <button
                        dusk="move-up-group"
                        type="button"
                        class="group-control btn border-l border-gray-200 dark:border-gray-700 w-8 h-8 block"
                        :title="__('Move up')"
                        @click.prevent="moveUp">
                        <icon type="arrow-up" class="align-top" width="16" height="16" />
                    </button>
                    <button
                        dusk="move-down-group"
                        type="button"
                        class="group-control btn border-l border-gray-200 dark:border-gray-700 w-8 h-8 block"
                        :title="__('Move down')"
                        @click.prevent="moveDown">
                        <icon type="arrow-down" class="align-top" width="16" height="16" />
                    </button>
                    <button
                        dusk="delete-group"
                        type="button"
                        class="group-control btn border-l border-gray-200 dark:border-gray-700 w-8 h-8 block"
                        :title="__('Delete')"
                        @click.prevent="confirmRemove">
                        <icon type="trash" width="16" height="16" />
                    </button>
                    <delete-flexible-content-group-modal
                        v-if="removeMessage"
                        @confirm="remove"
                        @close="removeMessage=false"
                        :message="field.confirmRemoveMessage"
                        :yes="field.confirmRemoveYes"
                        :no="field.confirmRemoveNo"
                    />
                </div>

            </div>
        </div>
            <field-list
            v-if="!group.preview"
            @click="$emit('group-selected', $el)"
            :fullScreen="fullScreen"
            :class="containerStyle"
            :fields="group.fields"
            :resource-name="resourceName"
            :resource-id="resourceId"
            :layoutName="group.name"
            :fieldName="field.attribute"
            :errors="errors"
            :stylesheet="field.enablePreview"
            :flexible_key="group.key"
        />
            <field-list-with-preview
            v-else
            :fullScreen="fullScreen"
            :fields="group.fields"
            :resource-name="resourceName"
            :resource-id="resourceId"
            :layoutName="group.name"
            :fieldName="field.attribute"
            :errors="errors"
            :stylesheet="field.enablePreview"
            :flexible_key="group.key"
            :selectedGroup="selectedGroup"
            :title="group.title"
            @group-selected="$emit('group-selected', $el)"
        />

            
        
    </div>
</template>

<script>
import BehavesAsPanel from 'nova-mixins/BehavesAsPanel';
import FieldList from "./FieldList";
import FieldListWithPreview from "./FieldListWithPreview";

export default {
    mixins: [BehavesAsPanel],

    components: {  FieldList, FieldListWithPreview },

    props: {
        errors: {},
        group: {},
        index: {},
        field: {},
        selectedGroup: false,
        fullScreen: false,
    },

    emits: ['move-up', 'move-down', 'remove'],

    data() {
        return {
            removeMessage: false,
            readonly: this.group.readonly,
        };
    },

    computed: {

        containerStyle() {
            let classes = ['grow', 'border-b', 'border-r', 'border-l', 'border-gray-200', 'dark:border-gray-700', 'rounded-b-lg'];

            if (! this.group.title) {
                classes.push('border-t');
                classes.push('rounded-tr-lg');
            }

            return classes;
        }
    },

    methods: {
        /**
         * Move this group up
         */
        moveUp() {
            this.$emit('move-up');
        },

        /**
         * Move this group down
         */
        moveDown() {
            this.$emit('move-down');
        },

        /**
         * Remove this group
         */
        remove() {
            this.$emit('remove');
        },

        /**
         * Confirm remove message
         */
        confirmRemove() {
            if (this.field.confirmRemove){
                this.removeMessage = true;
            } else {
                this.remove()
            }
        },

       
    },
}
</script>

<style>
    .group-control:focus {
        outline: none;
    }
    .group-control:hover {
        color: rgb(var(--colors-primary-400));
    }
    .confirm-message{
        position: absolute;
        overflow: visible;
        right: 38px;
        bottom: 0;
        width: auto;
        border-radius: 4px;
        padding: 6px 7px;
        border: 1px solid #B7CAD6;
        background-color: var(--20);
        white-space: nowrap;
    }
    [dir=rtl] .confirm-message{
        right: auto;
        left: 35px;
    }

    .confirm-message .text-danger {
        color: #ee3f22;
    }

    .rounded-l {
        border-top-left-radius: 0.25rem; /* 4px */
        border-bottom-left-radius: 0.25rem; /* 4px */
    }

    .rounded-t-lg {
        border-top-right-radius: 0.5rem; /* 8px */
        border-top-left-radius: 0.5rem; /* 8px */
    }

    .rounded-b-lg {
        border-bottom-left-radius: 0.5rem; /* 8px */
        border-bottom-right-radius: 0.5rem; /* 8px */
    }

    .box-content {
        box-sizing: content-box;
    }

    .grow {
        flex-grow: 1;
    }

    .grow-0 {
        flex-grow: 0;
    }

    .shrink {
        flex-shrink: 1;
    }

    .shrink-0 {
        flex-shrink: 0;
    }

    .right-\[3px\] {
        right: 3px;
    }
</style>