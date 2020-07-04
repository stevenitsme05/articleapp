<template>
    <div>
        <h2>Articles</h2>
        <form class="mb-2" @submit.prevent="addArticle">
            <div class="form-group">
                <input type="text" class="form-control" placeholder="Title" v-model="article.title">
            </div>
            <div class="form-group">
                <textarea type="text" class="form-control" placeholder="Body" v-model="article.body"></textarea>
            </div>
            <button type="submit" class="btn btn-primary btn-block">Save</button>
        </form>
        <hr>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a></li>
                <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{ pagination.current_page }} of {{ pagination.last_page }} </a></li>
                <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a></li>
            </ul>
        </nav>
        <div class="card card-body mb-2" v-for="a in articles" v-bind:key="a.id" >
            <h3>{{ a.title }}</h3>
            <p>{{ a.body }}</p>
            <hr>
            <button class="btn btn-success mb-2" @click="editArticle(a)">Edit</button>
            <button class="btn btn-danger" @click="deleteArticle(a.id)">Delete</button>
            
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            articles: [],
            article: {
                id: '',
                title: '',
                body: ''
            }, 
            article_id: '',
            pagination: {},
            edit: false
        }
    },

    created() { //by default this function will run
        this.fetchArticles();
    },

    methods: {
        //will fetch json first in api/articles api route then fetch the response with data
        fetchArticles(page_url){
            let vm = this;
            page_url = page_url || '/api/articles'
            fetch(page_url).then(res => res.json()).then(res => {
                //console.log(res.data);
                this.articles = res.data;
                vm.makePagination(res.meta, res.links); //pass to the makePagination function the meta and links get from the json res
            }).catch(err => console.log(err)); // in case with error
        },
        makePagination(meta, links) {
            let pagination = { // array of objects
                current_page: meta.current_page, //pass in current page from the meta data from the json res
                last_page: meta.last_page, // pass in last page from the meta data from the json res
                next_page_url: links.next, // pass in next from the links data from the json res
                prev_page_url: links.prev //pass in pre from the links data from the json res
            }

            this.pagination = pagination; //pass the pagination array object to the pagination object var defined on the data object above
        },
        deleteArticle(id) {
            if(confirm('Are you sure you want to delete the article selected?')){
                fetch(`api/article/${id}`, {
                    method: 'delete'
                }).then(res => res.json()).then(data => {
                    alert('Article Removed');
                    this.fetchArticles();
                }).catch(err => console.log(err));
            }
        },
        addArticle() {
            if(this.edit === false) {
                //add
                fetch('api/article', {
                    method: 'post',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                }).then(res => res.json()).then(data => {
                    this.article.title = '';
                    this.article.body = '';
                    alert('Article Added');
                    this.fetchArticles();
                }).catch(err => console.log(err));
            }else{
                //update
                fetch('api/article', {
                    method: 'put',
                    body: JSON.stringify(this.article),
                    headers: {
                        'content-type': 'application/json'
                    }
                }).then(res => res.json()).then(data => {
                    this.article.title = '';
                    this.article.body = '';
                    alert('Article Updated!');
                    this.fetchArticles();
                }).catch(err => console.log(err));
            }
        },
        editArticle(article){
            this.edit = true;
            this.article.id = article.id;
            this.article.article_id = article.id;
            this.article.title = article.title;
            this.article.body = article.body;
        }
    }
}
</script>