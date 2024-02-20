<template>
    <div class="p-3 text-right">
        <a v-if="!fullScreen" class="shadow relative text-white cursor-pointer rounded text-sm font-bold focus:outline-none focus:ring ring-primary-200 dark:ring-gray-600 inline-flex items-center justify-center h-9 px-4 ml-2 exit-fullscreen" @click="visitSite">Visit Site</a>
        <LoadingButton
            v-if="!fullScreen"
            dusk="update-button"
            type="submit"
            :disabled="isWorking"
            align="center"
            class="ml-3"
            :processing="wasSubmittedViaUpdateResource"
        >
            Update
        </LoadingButton>
    </div>


    <component
        :dusk="currentField.attribute"
        :is="currentField.fullWidth ? 'FullWidthField' : 'default-field'"
        :field="currentField"
        :errors="errors"
        :show-help-text="showHelpText"
        full-width-content
        class="relative"
        @keyup.escape="selectGroup(null)">

        <template #field>

            <div class="preview-panel" :class="{
                '-mx-8 -mt-6' : currentField.enablePreview && !fullScreen,
                'fixed inset-0 bg-white z-50 flex flex-col' : fullScreen,
                'mobile-view' : mobileView
                }">

                <div v-if="fullScreen" class="px-4 py-2 z-20 bg-white border-b text-center flex flex-wrap md:justify-end preview-panel-fullscreen">

                    <button class="shadow relative text-white cursor-pointer rounded text-sm font-bold focus:outline-none focus:ring ring-primary-200 dark:ring-gray-600 inline-flex items-center justify-center h-9 px-4 exit-fullscreen" @click="selectGroup(null)">Exit Fullscreen</button>

                    <component
                        :layouts="layouts"
                        :is="currentField.menu.component"
                        :field="currentField"
                        :limit-counter="limitCounter"
                        :limit-per-layout-counter="limitPerLayoutCounter"
                        :errors="errors"
                        :resource-name="resourceName"
                        :resource-id="resourceId"
                        @addGroup="addGroup($event)"
                        :class="{'mx-0 ' : currentField.enablePreview , 'ml-2' : currentField.enablePreview && fullScreen}"
                    />

                    <a class="shadow relative bg-primary-900 hover:bg-primary-900 text-white cursor-pointer rounded text-sm font-bold focus:outline-none focus:ring ring-primary-200 dark:ring-gray-600 inline-flex items-center justify-center h-9 px-4 ml-2"
                       @click="mobileView = !mobileView"
                       :title="mobileView ? 'Switch to Desktop View' : 'Switch to Mobile View'"
                    >
                        <svg fill="transparent" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true" width="20" height="20">
                            <path v-if="!mobileView" stroke-linecap="round" stroke-linejoin="round" d="M10.5 1.5H8.25A2.25 2.25 0 006 3.75v16.5a2.25 2.25 0 002.25 2.25h7.5A2.25 2.25 0 0018 20.25V3.75a2.25 2.25 0 00-2.25-2.25H13.5m-3 0V3h3V1.5m-3 0h3m-3 18.75h3"></path>
                            <path v-if="mobileView" stroke-linecap="round" stroke-linejoin="round" d="M9 17.25v1.007a3 3 0 01-.879 2.122L7.5 21h9l-.621-.621A3 3 0 0115 18.257V17.25m6-12V15a2.25 2.25 0 01-2.25 2.25H5.25A2.25 2.25 0 013 15V5.25m18 0A2.25 2.25 0 0018.75 3H5.25A2.25 2.25 0 003 5.25m18 0V12a2.25 2.25 0 01-2.25 2.25H5.25A2.25 2.25 0 013 12V5.25"></path>
                        </svg>
                    </a>

                    <a class="shadow relative text-white cursor-pointer rounded text-sm font-bold focus:outline-none focus:ring ring-primary-200 dark:ring-gray-600 inline-flex items-center justify-center h-9 px-4 ml-2 exit-fullscreen" @click="visitSite">Visit Site</a>

                    <LoadingButton
                        dusk="update-button"
                        type="submit"
                        :disabled="isWorking"
                        align="center"
                        class="bg-primary-500 hover:bg-primary-400"
                        :class="{'mx-2' : currentField.enablePreview , 'mx-2' : currentField.enablePreview && fullScreen}"
                        :processing="wasSubmittedViaUpdateResource"
                    >
                        Publish
                    </LoadingButton>

                </div>
                <div ref="flexibleFieldContainer"
                :class="{
                    'mb-4' : currentField.enablePreview && !fullScreen,
                    'flex-grow overflow-y-auto ml-sidebar border-l' : currentField.enablePreview && fullScreen
                }"
                >
                    <component
                        v-for="(group, index) in orderedGroups"
                        :is="currentField.enablePreview ? 'form-nova-flexible-content-group-with-preview' : 'form-nova-flexible-content-group'"
                        :dusk="currentField.attribute + '-' + index"
                        :key="group.key"
                        :field="currentField"
                        :group="group"
                        :index="index"
                        :resource-name="resourceName"
                        :resource-id="resourceId"
                        :errors="errors"
                        :mode="mode"
                        :selectedGroup="selectedGroupKey == group.key"
                        :full-screen="fullScreen"

                        @move-up="moveUp(group.key)"
                        @move-down="moveDown(group.key)"
                        @remove="remove(group.key)"
                        @group-selected="selectGroup(group.key, $event)"
                    />
                </div>

                <div :class="{
                    'flex flex-wrap justify-end px-4' : currentField.enablePreview,
                    'fixed z-50 border-t  bottom-0 bg-gray ml-sidebar p-2 border-l' : currentField.enablePreview && fullScreen,
                    'hidden' : currentField.enablePreview && !fullScreen,
                }">
                <component
                    :layouts="layouts"
                    :is="currentField.menu.component"
                    :field="currentField"
                    :limit-counter="limitCounter"
                    :limit-per-layout-counter="limitPerLayoutCounter"
                    :errors="errors"
                    :resource-name="resourceName"
                    :resource-id="resourceId"
                    @addGroup="addGroup($event)"
                    :class="{'mx-2 w-1/5' : currentField.enablePreview }"
                />

                <LoadingButton
                    dusk="update-button"
                    type="submit"
                    :disabled="isWorking"
                    align="center"
                    :class="{'mx-2' : currentField.enablePreview , 'mx-2' : currentField.enablePreview && fullScreen}"
                    :processing="wasSubmittedViaUpdateResource"
                >
                    Update
                </LoadingButton>
                </div>

            </div>
        </template>
    </component>
</template>

<script>

import FullWidthField from './FullWidthField';
import Sortable from 'sortablejs'
import { DependentFormField, HandlesValidationErrors, mapProps } from 'laravel-nova';
import Group from '../group';
import { ElementTypes } from '@vue/compiler-core';

export default {
    mixins: [HandlesValidationErrors, DependentFormField],

    props: {
        ...mapProps(['mode']),
    },

    components: { FullWidthField },

    computed: {

        layouts() {
            return this.currentField.layouts || false
        },

        orderedGroups() {
           return this.order.reduce((groups, key) => {
               groups.push(this.groups[key]);
               return groups;
           }, []);
        },

        limitCounter() {
            if (this.currentField.limit === null || typeof(this.currentField.limit) == "undefined") {
                return null;
            }

            return this.currentField.limit - Object.keys(this.groups).length;
        },


        limitPerLayoutCounter() {
            return this.layouts.reduce((layoutCounts, layout) => {
                if (layout.limit === null) {
                    layoutCounts[layout.name] = null;

                    return layoutCounts;
                }

                let count = Object.values(this.groups).filter(group => group.name === layout.name).length;

                if (layout.limitGroup != null) {
                    count += Object.values(this.groups).filter(group => group.limitGroup === layout.limitGroup).length;
                }

                layoutCounts[layout.name] = layout.limit - count;

                return layoutCounts;
            }, {});
        },
    },

    data() {
        return {
            renderComponent: true,
            order: [],
            groups: {},
            files: {},
            sortableInstance: null,
            selectedGroupKey: null,
            fullScreen: false,
            mobileView: false,
        };
    },

    beforeUnmount() {
        document.documentElement.classList.remove('overflow-hidden');
        if (this.sortableInstance) {
            this.sortableInstance.destroy();
        }
    },

    methods: {

        /**
         * Select the current group
         */
        selectGroup(groupKey, element) {
            if(this.selectedGroupKey == groupKey || groupKey == null) {
                this.selectedGroupKey = null;
                this.fullScreen = false;
                document.documentElement.classList.remove('overflow-hidden');
                let scrollY = this.$refs.flexibleFieldContainer.scrollTop;
                this.$nextTick(() => {
                    document.documentElement.scrollTop = scrollY + this.$refs.flexibleFieldContainer.getBoundingClientRect().top;
                });
            }
            else {
                document.documentElement.classList.add('overflow-hidden');
                let elementY = element.offsetTop - element.clientHeight/2;
                this.fullScreen = true;
                this.selectedGroupKey = groupKey;
                this.$nextTick(() => {
                    element.parentElement.scrollTop = elementY;
                });
            }
        },

        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
            this.value = this.currentField.value || [];
            this.files = {};
            this.populateGroups();
            this.$nextTick(this.initSortable.bind(this));

        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {

            let key, group;

            this.value = [];
            this.files = {};

            for (var i = 0; i < this.order.length; i++) {
                key = this.order[i];
                group = this.groups[key].serialize();

                // Only serialize the group's non-file attributes
                this.value.push({
                    layout: group.layout,
                    key: group.key,
                    attributes: group.attributes
                });

                // Attach the files for formData appending
                this.files = {...this.files, ...group.files};
            }

            this.appendFieldAttribute(formData, this.currentField.attribute);
            formData.append(this.currentField.attribute, this.value.length ? JSON.stringify(this.value) : '');

            // Append file uploads
            for (let file in this.files) {
                formData.append(file, this.files[file]);
            }

            this.$nextTick(this.initSortable.bind(this));
        },

        /**
         * Register given field attribute into the parsable flexible fields register
         */
        appendFieldAttribute(formData, attribute) {
            let registered = [];

            if (formData.has('___nova_flexible_content_fields')) {
                registered = JSON.parse(formData.get('___nova_flexible_content_fields'));
            }

            registered.push(attribute);

            formData.set('___nova_flexible_content_fields', JSON.stringify(registered));
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
            this.value = value || [];
            this.files = {};

            this.populateGroups();
        },

        /**
         * Set the displayed layouts from the field's current value
         */
        populateGroups() {
            this.order.splice(0, this.order.length);
            this.groups = {};

            for (var i = 0; i < this.value.length; i++) {
                this.addGroup(
                    this.getLayout(this.value[i].layout),
                    this.value[i].attributes,
                    this.value[i].key,
                    this.currentField.collapsed
                );
            }
            if(!this.resourceId) {
                this.currentField.defaultLayouts.forEach((layoutName) => {
                    this.addGroup(this.getLayout(layoutName));
                });
            }
        },

        /**
         * Retrieve layout definition from its name
         */
        getLayout(name) {
            if(!this.layouts) return;
            return this.layouts.find(layout => layout.name == name);
        },

        visitSite() {
            window.open(window.location.origin);
        },

        /**
         * Append the given layout to flexible content's list
         */
        addGroup(layout, attributes, key, collapsed) {
            if(!layout) return;

            collapsed = collapsed || false;

            let fields = attributes || JSON.parse(JSON.stringify(layout.fields)),
                group = new Group(layout.name, layout.title, fields, this.currentField, key, layout.preview, collapsed, layout.limitGroup);

            this.groups[group.key] = group;

            this.order.push(group.key);

            let bannerKey = this.getGroupKeyByName(this.groups, 'banner') ?? '';
            let footerKey = this.getGroupKeyByName(this.groups, 'footer') ?? '';

            // Make sure banner and footer are always on top and bottom
            this.order = [
                ...this.order.filter(flag => flag === bannerKey), // banner
                ...this.order.filter(flag => (flag !== bannerKey && flag !== footerKey)), // else
                ...this.order.filter(flag => flag === footerKey)  // footer
            ];


            if(this.fullScreen) {
                this.selectedGroupKey = group.key;
                this.$nextTick(() => {
                    this.$refs.flexibleFieldContainer.lastElementChild.scrollIntoView({behavior: "smooth", block: "end", inline: "center"});
                });
            }

        },

        getGroupKeyByName(groups, name) {
          return Object.keys(groups).find(key => groups[key]['name'] === name);
        },


        /**
         * Move a group up
         */
        moveUp(key) {
            let index = this.order.indexOf(key);

            if(index <= 1) return;
            this.order.splice(index - 1, 0, this.order.splice(index, 1)[0]);

        },

        /**
         * Move a group down
         */
        moveDown(key) {
            let index = this.order.indexOf(key);

            if(index < 0 || index >= this.order.length - 2) return;

            this.order.splice(index + 1, 0, this.order.splice(index, 1)[0]);

        },

        /**
         * Remove a group
         */
        remove(key) {
            let index = this.order.indexOf(key);

            if(index < 0) return;

            this.order.splice(index, 1);
            delete this.groups[key];
        },


        initSortable() {
            const containerRef = this.$refs['flexibleFieldContainer']

            if (! containerRef || this.sortableInstance) {
                return;
            }

            this.sortableInstance = Sortable.create(containerRef, {
                ghostClass: 'nova-flexible-content-sortable-ghost',
                dragClass: 'nova-flexible-content-sortable-drag',
                chosenClass: 'nova-flexible-content-sortable-chosen',
                direction: 'vertical',
                handle: '.nova-flexible-content-drag-button',
                scrollSpeed: 5,
                animation: 500,
                onEnd: (evt) => {
                    const item = evt.item;
                    const key = item.id;
                    const oldIndex = evt.oldIndex;
                    const newIndex = evt.newIndex;

                    if (newIndex < oldIndex) {
                        this.moveUp(key);
                    } else if (newIndex > oldIndex) {
                        this.moveDown(key);
                    }
                 }
            });
        },
    }
}
</script>

<style>
.ml-sidebar {
    margin-left: 33.333%;
    width: 66.9999%;
}

@media (min-width: 1024px){
    .ml-sidebar {
        margin-left: 25%;
        width: 75%;
    }
}
@media (min-width: 1600px){
    .ml-sidebar {
        margin-left: 20%;
        width: 80%;
    }
}

.-mt-5 {
    margin-top: -1.25rem;
}
.-mt-6 {
    margin-top: -1.5rem;
}

.-mx-8 {
    margin-left: -2rem;
    margin-right: -2rem;
}
.mobile-view iframe {
    max-width: 480px;
    background-color: white;
}
.preview-panel {
    margin-top: -3rem;
    padding-top: 3rem;
    padding-bottom: 1rem;
}
.preview-panel-fullscreen {
    background-color: rgba(var(--colors-primary-gray-purple));
}
.exit-fullscreen {
    background-color: rgba(var(--colors-primary-gray-dark));
}
.exit-fullscreen:hover {
    background-color: rgba(var(--colors-primary-green));
}
:is(.dark .preview-panel .dark\:text-gray-900) {
    color: white;
}
</style>
