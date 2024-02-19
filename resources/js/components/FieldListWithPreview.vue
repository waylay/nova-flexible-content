<template>
    <div >
        <div
            @click="$emit('group-selected')"
            class="relative"
            :class="{
                'pointer-events-none': selectedGroup && !initialPreviewHtml,
            }"
            style="min-height: 80px"
        >
            <div
                v-if="fieldListErrorCount"
                class="font-bold z-20 mt-3 ml-3 border shadow absolute bg-white px-4 py-2 rounded"
            >
                <icon
                    type="exclamation-circle"
                    class="text-red-600 align-middle"
                    width="20"
                    height="20"
                />
                {{ fieldListErrorCount }} errors
            </div>
            <div
                class="absolute z-10 inset-0 transition border-2"
                :class="{
                    'pointer-events-none  border-primary-500 border-dashed':
                        selectedGroup,
                    'hover:border-gray-200 border-transparent': !selectedGroup,
                }"
            ></div>
            <preview-iframe
                v-if="initialPreviewHtml"
                :stylesheet="stylesheet"
                :flexible_key="flexible_key"
                :fullScreen="fullScreen"
                :initialPreviewHtml="initialPreviewHtml"
                :updatedPreviewHtml="updatedPreviewHtml"
            />
        </div>
        <div
            v-show="selectedGroup"
            class="preview-controls absolute top-0 left-0 w-1/3 md:w-1/4 xl:w-1/5 bottom-0 h-full bg-gray-light overflow-y-scroll overflow-x-hidden self-stretch"
        >
            <div class="w-full py-5">
                <div class="px-2 pt-8 pb-4 gap-2 flex flex-row items-center">
                    <button
                        :aria-label="`Close ${title}`"
                        @click.prevent="$emit('group-selected')"
                    >
                        <icon
                            type="arrow-left"
                            class="align-top rounded-full hover:bg-gray-200 p-2"
                            width="36"
                            height="36"
                        />
                    </button>
                    <h3 class="font-bold text-2xl text-primary-500">{{ title }}</h3>
                </div>

                <div
                    class="fields relative divide-y divide-gray-100 dark:divide-gray-700"
                    style="margin-left: -0.25rem; margin-right: -0.25rem"
                >
                    <fieldset ref="fieldset">
                        <component
                            v-for="(item, index) in stackedFields"
                            :key="index"
                            :is="'form-' + item.component"
                            :resource-name="resourceName"
                            :resource-id="resourceId"
                            fullWidthContent="true"
                            @input="onInput($event, item)"
                            @change="onInput($event, item)"
                            @click="onInput($event, item)"
                            @focusout="onInput($event, item)"
                            :field="item"
                            :errors="errors"
                            :mode="mode"
                            :show-help-text="item.helpText != null"
                            :class="{
                                'remove-bottom-border':
                                    index == fields.length - 1,
                            }"
                        />
                    </fieldset>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { tsImportEqualsDeclaration } from "@babel/types";
import _, { map } from "underscore";
import PreviewIframe from "./PreviewIframe";

const watchedComponents = ['nova-file-manager-field', 'tinymce-editor'];
export default {
    props: {
        layoutName: null,
        resourceName: null,
        resourceId: null,
        fieldName: null,
        containerStyle: [],
        fields: {},
        errors: {},
        stylesheet: null,
        flexible_key: null,
        fullScreen: false,
        selectedGroup: false,
        title: null,
    },

    components: { PreviewIframe },

    data() {
        return {
            form: new FormData(),
            timer: null,
            initialPreviewHtml: null,
            updatedPreviewHtml: null,
            containsInvalidFormElements: false,
        };
    },

    computed: {
        stackedFields() {
            return this.fields.map((field) => {
                field.stacked = true;


                return field;
            });
        },

        fieldListErrorCount() {
            return this.fields.filter((field) =>
                Object.keys(this.errors.errors).includes(field.attribute)
            ).length;
        },
    },

    mounted() {
        this.form.append("__key", this.flexible_key);

        this.$nextTick(() => {
            this.fields.forEach((field) => {
                field.fill(this.form);
                if(!this.form.has(field.attribute)) {
                    this.form.delete(field.attribute);
                    this.form.set(field.attribute, field.value ? JSON.stringify(field.value) : "");
                }
                if(field.fields) {
                    field.fields.forEach((innerField) => {
                        if(innerField.attribute && !this.form.has(innerField.attribute)) {
                            this.form.set(innerField.attribute, innerField.value ? JSON.stringify(innerField.value) : "");
                        }
                    });
                }
                if( field.sortableUriKey == 'promotion_type') {
                    document.dispatchEvent(new CustomEvent("promotion_changed", {
                        bubbles: true,
                        detail: { text: () => this.form.get(field.attribute) },
                    }));
                }

            });
            this.getPreview();
        });


    },

    watch: {
        selectedGroup(newValue, oldValue) {
            if (!newValue) {
                this.validateFields();
                if (this.containsInvalidFormElements) {
                    this.$emit("group-selected");
                    this.$refs.fieldset.elements.forEach((field) => {
                        field.reportValidity();
                    });
                }
            }
        },
    },

    methods: {
        validateFields() {
            this.containsInvalidFormElements = false;
            this.$refs.fieldset.elements.forEach((field) => {
                if (!field.checkValidity()) {
                    this.containsInvalidFormElements = true;
                }
            });
        },

        onInput: _.debounce(function debounceRead(evt, updatedField) {
            this.update(updatedField);

            if('tinymce-editor' === updatedField.component && typeof window.tinymce !== "undefined") {
                window.tinymce.get(updatedField.attribute).on('input', (e) => this.update(updatedField));
            }

            // Some fields do not trigger an update event so we add a temporary watcher to check for updates
            if(watchedComponents.includes(updatedField.component)) {
                clearInterval(this.timer)
                this.timer = setInterval(() => {
                    this.update(updatedField);
                }, 1500);
            } else {
                clearInterval(this.timer)
                this.timer = null;
            }

        }, 120),

        update(updatedField) {
            let field = this.fields.find(
                (field) => field.uniqueKey == updatedField.uniqueKey
            );
            this.form.delete(field.attribute);
            field.fill(this.form);

            if( field.sortableUriKey == 'promotion_type') {
                document.dispatchEvent(new CustomEvent("promotion_changed", {
                    bubbles: true,
                    detail: { text: () => this.form.get(field.attribute) },
                }));
            }

            this.getPreview();
        },

        getPreview() {
            fetch(
                `/nova-vendor/flexible/view/${this.resourceName}/${this.resourceId}/${this.fieldName}/${this.layoutName}`,
                {
                    method: "post",
                    body: this.form,
                    headers: {
                        "X-CSRF-TOKEN": document.querySelector(
                            'meta[name="csrf-token"]'
                        ).content,
                    },
                }
            )
                .then((response) => response.json())
                .then((json) => {
                    // Reset the form so we don't keep uploading files on every refresh
                    this.form = new FormData();
                    this.form.append("__key", this.flexible_key);
                    this.fields.forEach((field) => {
                        if(json.data[field.attribute] && typeof(json.data[field.attribute]) == 'object') {
                            json.data[field.attribute] = JSON.stringify(json.data[field.attribute]);
                        }

                        this.form.set(
                            field.attribute,
                            json.data[field.attribute] ?? ""
                        );
                    });

                    if (!this.initialPreviewHtml || json.has_uploads) {
                        this.initialPreviewHtml = json.view;
                    } else {
                        this.updatedPreviewHtml = json.view;
                    }
                });
        },
    },
    beforeUnmount() {
        clearInterval(this.timer)
        this.timer = null;
    },
};
</script>

<style>
.w-screen {
    width: 100vw;
}
.w-1\/3 {
    width: 33.3333%;
}

@media (min-width: 1024px){
    .md\:w-1\/4{
        width: 25%;
    }
}
@media (min-width: 1600px){
    .xl\:w-1\/5{
        width: 20%;
    }
}

.left-1\/2 {
    left: 50%;
}

.top-1\/2 {
    top: 50%;
}

.self-stretch {
    align-self: stretch;
}

.fields .md\:px-8,
.fields .px-8 {
    padding-left: 1rem;
    padding-right: 1rem;
}
.fields .md\:py-6,
.fields .py-6 {
    padding-top: .5rem;
    padding-bottom: .5rem;
}

.fields label .flex {
    flex-direction: column;
    gap: 0.5rem;
}

.fields .form-file {
    width: 100%;
}

.fields .md\:w-1\/4,
.fields .w-1\/2,
.fields .md\:w-4\/5,
.fields .md\:w-3\/5 {
    width: 100%;
}

.nsr-w-full.nsr-flex.nsr-border-b.nsr-py-2 {
    display: none;
}

.fields .simple-repeatable-fields-wrapper {
    flex-direction: column;
    gap: 0.5em;
}

.fields .simple-repeatable.form-field .simple-repeatable-row {
    margin-left: 0;
    padding-left: 0;
    width: 100%;
}

/* .fields .simple-repeatable.form-field>:nth-child(2) {
    margin-right: 0;
    width: 80% !important;
} */

.fields .simple-repeatable.form-field .simple-repeatable-row .delete-icon,
.simple-repeatable.form-field .simple-repeatable-row .vue-draggable-handle {
    margin-right: 0;
}

.simple-repeatable.form-field
    .simple-repeatable-row
    > .simple-repeatable-fields-wrapper
    .translatable-field
    > div:not(:first-child)
    > div,
.simple-repeatable.form-field
    .simple-repeatable-row
    > .simple-repeatable-fields-wrapper
    > * {
    margin-right: 0;
}

.hover\:border-gray-200:hover {
    border-color: rgba(var(--colors-gray-200));
}
</style>
