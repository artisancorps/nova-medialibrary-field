<template>
    <SortFiles :files="sortedFiles" @sort="handleSort" :disabled="sortingDisabled" class="flex flex-wrap -m-2">
        <Resource v-for="file in sortedFiles" :key="file.id" class="p-2"
            :file="file"
            :field="field"
            :readonly="readonly"
        >
            <component slot-scope="{ file, fileEvents }"
                :is="fileWrapperComponent"
                :field="field"
                :file="file"
                :file-events="fileEvents"
            >
                <File slot-scope="{ file, fileEvents }"
                    :field="field"
                    :file="file"
                    :loading="loadingFileId === file.id"
                    :show-actions="showActions"
                    v-on="fileEvents"
                />
            </component>
        </Resource>
    </SortFiles>
</template>

<script>
import { Sortable, Toasted, Loading } from '../../mixins'
import SortFiles from './SortFiles'
import Resource from './Resource/Resource'
import File from './File/File'

export default {
    mixins: [Sortable, Toasted, Loading],

    components: {
        SortFiles,
        Resource,
        File,
    },

    props: {
        value: Array,
        field: Object,
        readonly: Boolean,
    },

    data () {
        return {
            files: [],
        }
    },

    computed: {
        fileWrapperComponent () {
            return this.field.fileWrapperComponent || 'medialibrary-field-file-wrapper'
        }
    },

    watch: {
        value: {
            immediate: true,
            handler (value) {
                this.handleValueChange(value)
            }
        }
    },

    created () {
        const event = `medialibrary:${this.field.collectionName}-updated`

        const handler = value => this.$emit('input', value)

        Nova.$on(event, handler)

        this.$once('hook:beforeDestroy', () => Nova.$off(event, handler))
    },

    methods: {
        handleValueChange (value) {
            const files = (value || []).map(file => {
                file.authorizedToDelete = !this.readonly && file.authorizedToDelete
                file.authorizedToUpdate = !this.readonly && file.authorizedToUpdate

                return file
            })

            this.files = this.field.multiple ? files : files.slice(-1)
        }
    }
}
</script>
