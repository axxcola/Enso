<template>

    <card :icon="icon"
        refresh
        scrollable
        :search="documents.length > 1"
        :title="title || __('Documents')"
        :overlay="loading"
        @refresh="get()"
        ref="card"
        :collapsed="!open || isEmpty"
        @expand="isEmpty ? $refs.card.collapse() : null"
        @query-update="query = $event"
        :badge="count"
        :controls="1">
        <card-control slot="control-1">
            <file-uploader
                @upload-successful="get();"
                :url="uploadLink"
                multiple>
            </file-uploader>
        </card-control>
        <div class="wrapper has-padding-medium">
            <document v-for="(doc, index) in filteredDocuments"
                :key="index"
                :doc="doc"
                @delete="destroy(index)">
            </document>
        </div>
    </card>

</template>

<script>

import { mapGetters } from 'vuex';
import fontawesome from '@fortawesome/fontawesome';
import { faCopy } from '@fortawesome/fontawesome-free-solid/shakable.es';
import Card from '../bulma/Card.vue';
import CardControl from '../bulma/CardControl.vue';
import Document from './Document.vue';
import FileUploader from '../fileuploader/FileUploader.vue';

fontawesome.library.add(faCopy);

export default {
    name: 'Documents',

    components: {
        Card, CardControl, Document, FileUploader,
    },

    props: {
        open: {
            type: Boolean,
            default: false,
        },
        id: {
            type: Number,
            required: true,
        },
        type: {
            type: String,
            required: true,
        },
        title: {
            type: String,
            default: '',
        },
    },

    data() {
        return {
            documents: [],
            query: '',
            loading: false,
        };
    },

    computed: {
        ...mapGetters('locale', ['__']),
        count() {
            return this.documents.length;
        },
        isEmpty() {
            return this.count === 0;
        },
        uploadLink() {
            return route('core.documents.upload', [this.type, this.id], false);
        },
        filteredDocuments() {
            return this.query
                ? this.documents.filter(doc => doc.original_name.toLowerCase()
                    .indexOf(this.query.toLowerCase()) > -1)
                : this.documents;
        },
        icon() {
            return faCopy;
        },
    },

    watch: {
        isEmpty() {
            if (this.isEmpty) {
                this.$refs.card.collapse();
            }
        },
    },

    created() {
        this.get();
    },

    methods: {
        get() {
            this.loading = true;

            axios.get(route('core.documents.index', [this.type, this.id], false)).then(({ data }) => {
                this.documents = data;
                this.loading = false;
                this.$refs.card.resize();
            }).catch((error) => {
                this.loading = false;
                this.handleError(error);
            });
        },
        destroy(index) {
            this.loading = true;

            axios.delete(route('core.documents.destroy', [this.documents[index].id], false)).then(() => {
                this.loading = false;
                this.documents.splice(index, 1);
                index = null;
            }).catch((error) => {
                this.loading = false;
                index = null;
                this.handleError(error);
            });
        },
    },
};

</script>

<style scoped>

    .wrapper {
        max-height: 500px;
        overflow-y: auto;
    }

</style>
