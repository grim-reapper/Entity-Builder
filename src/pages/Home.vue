<template>
    <div class="col-sm-6">
        <div class="text-center" style="margin-top: 22%;">
            <div>
                <img src="logo.svg" alt="logo" style="width: 222px;" />
            </div>
            <h1>{{ title }}</h1>
            <p>{{ version }}</p>
            <div style="margin-top: 33px;">
                <b-button-group>
                    <b-button @click="create" variant="outline-primary"> New </b-button>
                    <b-button variant="outline-primary">
                        <label class="button-label">
                            Upload
                            <input @change="upload($event)" type="file" accept=".json" style="display: none;" />
                        </label>
                    </b-button>
                    <b-button @click="connect" variant="outline-primary"> Connect </b-button>
                </b-button-group>
            </div>
        </div>
    </div>
</template>

<script>
import { connect, getErrorMessage } from '../request.js'
import builder from '../states/builder.js'
import { makePreset, makeProject, loadPreset, loadProject } from '../project.js'
import sidebar from '../states/sidebar.js'

export default {
    name: 'Home',
    data() {
        return {
            title: process.env.VUE_APP_TITLE,
            version: process.env.VUE_APP_VERSION,
            builder,
            sidebar,
            domain: window.location.origin,
        }
    },
    created() {
        sidebar.show('', null)
    },
    methods: {
        connect() {
            const domain = prompt('Please enter the domain', this.domain)
            if (domain) {
                this.domain = domain
                connect(this.domain)
                    .then(response => {
                        if (builder.project) {
                            return
                        }
                        if (response.data) {
                            const data = JSON.parse(response.data)
                            this.load(data)
                            return
                        }

                        this.$bvToast.toast('No saved data', {
                            title: 'i',
                            variant: 'success',
                            solid: true,
                        })
                    })
                    .catch(error => {
                        console.error(error)
                        const message = getErrorMessage(error)
                        this.$bvToast.toast(message, {
                            title: 'i',
                            variant: 'danger',
                            solid: true,
                        })
                    })
            }
        },
        create() {
            try {
                const name = prompt('Please input the project name', 'Entity')
                if (name) {
                    builder.project = makeProject(name)
                    loadPreset(builder.project, builder.preset)
                    this.$router.push('/project')
                }
            } catch (error) {
                console.error(error)
                this.$bvToast.toast(error.message, {
                    title: 'i',
                    variant: 'danger',
                    solid: true,
                })
            }
        },
        upload(event) {
            const reader = new FileReader()
            reader.onload = event => {
                try {
                    const data = JSON.parse(event.target.result)
                    this.load(data)
                } catch (error) {
                    console.error(error)
                    this.$bvToast.toast(error.message, {
                        title: 'i',
                        variant: 'danger',
                        solid: true,
                    })
                }
            }
            reader.onerror = error => {
                alert(error)
            }
            reader.readAsText(event.target.files[0])
        },
        load(data) {
            const version = data.version
            if (version < 10) {
                if (!confirm('This is an old version project,\nold templates may not work properly.\nContinue?')) {
                    return
                }
            }
            builder.project = loadProject(data, builder.preset)
            if (version === 10) {
                builder.project.update(makePreset(builder.preset))
            }
            this.$router.push('/project')
        },
    },
}
</script>
