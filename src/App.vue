<template>
    <a class="fixo button is-large is-danger is-loading" v-show="isLoading">Загрузка</a>
    <div class="container">
        <h1 class="title">{{title}}</h1>
        <div class="columns">
            <div class="column is-5">
                <p class="control has-addons">
                    <input class="input is-expanded" type="text" placeholder="Поиск по названию" v-model="search">
                    <a class="button is-info" @click.prevent="searchTasks">Поиск</a>
                </p>
            </div>
            <div class="column is-6">

            </div>
            <div class="column is-1">
                <a class="button is-info" @click.prevent="newTasks">Новая запись</a>
            </div>

        </div>
        <div class="columns">
            <div class="column is-12">
                <table class="table is-narrow is-bordered">
                    <thead>
                    <tr>
                        <th>Наименование задачи</th>
                        <th>Ответсвенный</th>
                        <th>Примечания</th>
                        <th>Замечания</th>
                        <th>Сроки выполнения</th>
                        <th>Действия</th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="task in tasks">
                        <td>{{task.name}}</td>
                        <td>{{task.response}}</td>
                        <td>{{task.editdescrip}}</td>
                        <td>{{task.editremark}}</td>
                        <td>{{task.datedone}}</td>
                        <td class="is-icon">

                            <a href="#" @click.prevent="editTask(task)">
                                <i class="fa fa-edit"></i>
                            </a>
                            <a href="#" @click.prevent="removeTask(task)">
                                <i class="fa fa-trash"></i>
                            </a>
                        </td>
                    </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <div id="modal_brewery" class="modal" :class="{'is-active':showModal}">
        <div class="modal-background"></div>
        <div class="modal-card">
            <header class="modal-card-head">
                <p class="modal-card-title">{{selected.name}}</p>
                <button class="delete" @click.prevent="showModal=false"></button>
            </header>
            <section class="modal-card-body">

                <div class="columns">
                    <div class="column">
                        <label class="label">Наименование задачи</label>
                        <p class="control">
                            <input class="input" type="text" placeholder="Название продукта" v-model="selected.name">
                        </p>
                    </div>
                    <div class="column">
                        <label class="label">Ответсвенный</label>
                        <p class="control">
                            <input class="input" type="text" placeholder="Ответсвенный" v-model="selected.response">
                        </p>
                    </div>
                </div>

                <label class="label">Примечания</label>
                <p class="control">
                    <textarea class="textarea" placeholder="Textarea" v-model="selected.editdescrip"></textarea>
                </p>

                <label class="label">Замечания</label>
                <p class="control">
                    <input class="input" type="text" placeholder="Text input" v-model="selected.editremark">
                </p>

                <label class="label">Сроки выполнения</label>
                <p class="control">
                    <input class="input" type="text" placeholder="Text input" v-model="selected.datedone">
                </p>

            </section>
            <footer class="modal-card-foot">
                <a class="button is-primary" @click.prevent="saveTask">Подтвердить</a>
                <a class="button" @click.prevent="showModal=false">Отмена</a>
            </footer>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                isLoading: false,
                title: 'СИБУР',
                search: '',
                tasks: [],
                page: 1,
                total: 0,
                selected: {},
                showModal: false
            }
        },
        methods: {
            onChangePage(page) {
                this.page = page
                this.loadTask()
            },
            showLoading() {
                this.isLoading = true;
            },
            hideLoading() {
                this.isLoading = false;
            },
            loadTask() {

                let t = this
                this.showLoading()

                let qString = "";

                if (this.search) {
                    qString = `&q=${this.search}`
                }

                this.$http.get(httpurl+ `/tasks?_${qString}`).then(
                // this.$httpurl.get(`/tasks?_${qString}`).then(
                    response => {
                        t.tasks = response.json()
                        t.total = response.headers['X-Total-Count']
                    },
                    error => {
                        console.log(error)
                    }).finally(function () {
                    t.hideLoading();
                })

            },
            searchTasks() {
                this.loadTask()
            },
            newTasks() {
                this.selected = {}
                this.showModal = true;
            },
            editTask(tasks) {
                this.selected = tasks
                this.showModal = true;
            },
            removeTask(tasks) {
                let self = this;
                swal({
                    title: "Вы уверены?",
                    text: `Вы хотите удалить "${tasks.name}"`,
                    type: "warning",
                    showCancelButton: true,
                    confirmButtonColor: "#DD6B55",
                    cancelButtonText: "Отмена",
                    confirmButtonText: "Да, удалить!",
                    showLoaderOnConfirm: true,
                    closeOnConfirm: false
                }, function () {

                    self.$http.delete(httpurl + `/tasks/${tasks.id}`).then(
                        result => {
                            swal("Удаление успешно!")
                            self.loadTask()
                        })
                });

            },
            saveTask() {
                if (this.selected.id != null) {  //EDIT
                    this.$http.put(httpurl + `/tasks/${this.selected.id}`, this.selected).then(
                        response => {
                            this.$set('selected', {})
                            this.$set('showModal', false)
                        },
                        error => {
                            console.error(error)
                        }
                    ).finally(
                        this.loadTask()
                    )
                }
                else { //NEW
                    this.$http.post(httpurl + `/tasks`, this.selected).then(
                        response => {
                            this.$set('selected', {})
                            this.$set('showModal', false)
                        },
                        error => {
                            console.error(error)
                        }
                    ).finally(
                        this.loadTask()
                    )
                }
            }
        },
        created() {
            global.httpurl = "http://localhost:8080";
            this.loadTask();
        },
    }
</script>

<style>
    body {
        margin-top: 50px
    }
    .fixo {
        float: right;
        margin-right: 10px;
        margin-top: 0px;
        z-index: 1000;
    }
</style>