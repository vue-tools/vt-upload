<template>
    <input type="file" :accept="accept" @change="change($event)" />
</template>

<script>
    import Vue from 'vue'
    import Imager from 'Imager'
    export default {
        props: {
            url: {
                type: String
            },
            name: {
                type: String,
                default: 'file'
            },
            compress: {
                type: Boolean
            },
            limit: {
                type: Boolean,
                default: true
            },
            base64: {
                type: Boolean
            },
            accept: {
                type: String,
                default: /mobile/i.test(window.navigator.appVersion) ? 'image/*' : ''
            }
        },
        methods: {
            change(e) {
                let file, image
                file = e.target.files[0]
                if (!file) {
                    return
                }
                e.target.value = ''
                this.$emit('change', e)
                if (Imager.isImage(file) && this.compress) {
                    image = new Imager({
                        file: file,
                        quality: 0.8,
                        isLimit: this.limit
                    })
                    image.getExif().then(data => image.loaded()).then(base64 => {
                        if (this.base64) {
                            this.upload(base64, base64)
                        } else {
                            this.upload(Imager.base64ToFile(base64, file.name), base64)
                        }
                    })
                } else if(this.base64 && Imager.isImage(file)){
                    Imager.fileToBase64(file).then(base64 => this.upload(base64, base64))
                } else {
                    this.upload(file, null)
                }
            },
            upload(file, base64) {
                if (this.url) {
                    let fd = new FormData()
                    fd.append(this.name, file)
                    Vue.http.post(this.url, fd, {
                        progress: (e) => {
                            this.$emit('progress', e)
                        }
                    }).then((response) => {
                        this.$emit('upload', response, base64)
                    }).catch((response) => {
                        this.$emit('upload', response, base64)
                    })
                } else {
                    this.$emit('upload', file, base64)
                    this.$emit('progress', () => {
                        console.warn('if url was not input, progress not work')
                    })
                }
            }
        }
    }
</script>