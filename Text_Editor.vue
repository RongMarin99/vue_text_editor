<template>
    <div>
        <client-only>
            <VueEditor 
                useCustomImageHandler 
                @image-added="handleImageAdded"
                @image-removed="handleImageRemoved"
                v-model="text"
                :editorOptions="editorSettings"
                :editorToolbar="customToolbar"
            />    
        </client-only>
    </div>
</template>

<script>
import { Quill } from "vue2-editor";
import ImageResize from 'quill-image-resize';
Quill.register("modules/imageResize", ImageResize)
export default{
    props: {
        editorSettings: {
            type: Object,
            default() {
                return {
                    modules: {
                        imageResize: true
                    } 
                }
            }
        },
        customToolbar: {
            type: Array,
            default(){
                return [
                    ["bold", "italic", "underline"],
                    [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
                    ['blockquote',"link",],
                    [{ list: "ordered" }, { list: "bullet" }],
                    [{ 'align': [] }],
                    [{ 'color': [] }, { 'background': [] }],
                    ["image",]
                ]
            } 
        },
        setText: {
            type: String,
            default: ''
        }
    },
    watch: {
        setText: {
            handler(val){
                if(this.nullToVoid(val)!=''){
                    this.text = val
                }
            },
            immediate: true
        }
    },
    data(){
        return {
            text: '',
            image_upload_path: process.env.WEB_URL+'/images/gallery/', 
        }
    },
    methods: {
        handleImageAdded(file, Editor, cursorLocation,resetUploader) {
            var formData = new FormData();
            formData.append("image", file);
            this.$axios({
                url: '/upload/images',
                method: 'POST',
                data: formData
            }).then(result => {
                const url = this.image_upload_path+result.data; // Get url from response
                Editor.insertEmbed(cursorLocation, "image", url);
                resetUploader();
            }).catch(err => {
                console.log(err);
            });
        },
        handleImageRemoved(image){
            var imageName = image.split('/')
            this.$axios.$post('/delete/image',{image: imageName[(imageName.length)-1]}).then(res => {

            })
        },
        getText(){
            return this.text
        },
    }
}
</script>