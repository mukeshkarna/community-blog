<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 offset-md-2">
                <div class="card">
                    <div class="card-body">
                         <picture-input 
                            ref="pictureInput" 
                            @change="onChange" 
                            accept="image/jpeg,image/png" 
                            size="5" 
                            buttonClass="btn btn-danger"
                            >
                        </picture-input>
                        <select class="form-control">
                            <option selected>Select a Category</option>
                            <option :value="category.id" :key="category.id" v-for="category in categories">{{category.name}}</option>
                        </select>
                        <input type="text" placeholder="Title" v-model="title" class="form-control my-3">
                        <wysiwyg v-model="content" />
                        <div class="text-center">
                            <button :disabled="loading" @click="createArticle()" class="btn-success btn btn-lg mt-3">
                                <i class="fas fa-spinner" v-if="loading"></i>
                                {{loading ? '' : 'Create Article'}}
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Axios from 'axios'
import config from '@/config';
import PictureInput from 'vue-picture-input';
export default {
    beforeRouteEnter(to, from, next){
        if (!localStorage.getItem('auth')) {
            return next({ path: "/login"});
        }
        next();
    },
    data(){
        return{
            title:"",
            content: "",
            image: null,
            categories: [],
            category:"",
            loading:false
        }
    },
    mounted(){
        this.getCategories();
    },
    components:{
        PictureInput
    },
    methods:{
        onChange(image){
            this.image = image;
        },
        createArticle(){
            if (!this.title || !this.content || !this.content || !this.category) {
                this.$noty.error('Please fill in all fields');
                return;
            }
            this.loading= true;
            const form = new FormData();
            form.append('file', this.image);
            form.append('upload_preset', process.env.VUE_APP_CLOUDINARY_PRESET);
            form.append('api_key', process.env.VUE_APP_CLOUDINARY_API_KEY);

            Axios.post(process.env.VUE_APP_CLOUDINARY_URL ,form)
            .then(res => 
                Axios.post(`${config.apiUrl}/articles`, {
                    title: this.title,
                    content: this.content,
                    category_id: this.category,
                    imageUrl: res.data.secure_url
                },{
                    headers:{
                        Authorization: `Bearer ${this.$root.auth.token}`
                    }
                })
                .then(() =>{
                    this.$noty.success('Article created successfully.');
                    this.loading= false;
                    this.$router.push("/");
                }).catch(()=>{
                     this.loading= false;
                    this.$noty.error('Oops!! Something went wrong.');
                })
            )
            .catch(()=>{
                this.loading= false;
                this.$noty.error('Oops!! Something went wrong.');
            });
        },
        getCategories(){
            const categories = localStorage.getItem("categories");
            if (categories) {
                this.categories = JSON.parse(categories);
                return;
            }
            Axios.get(`${config.apiUrl}/categories`)
            .then(res => {
                this.categories = res.data.categories;
                localStorage.setItem("categories", JSON.stringify(this.categories))
            })
        }
    }
}
</script>

<style>

</style>
