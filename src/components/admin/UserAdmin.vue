<template>
    <div class="user-admin">
        <b-form>
            <input id="user-id" type="hidden" v-model="user.id" />
            <b-row>
                <b-col md="6" sm="12">
                    <b-form-group label="Nome:" label-for="user-name">
                        <b-form-input id="user-name" type="text" ref="searchFocus"
                            v-model="user.name" required
                            :readonly="mode === 'remove' || mode === 'read'"
                            placeholder="Nome do Usuário..." />
                    </b-form-group>
                </b-col>
                <b-col md="6" sm="12">
                    <b-form-group label="E-mail:" label-for="user-email">
                        <b-form-input id="user-email" type="text"
                            v-model="user.email" required
                            :readonly="mode === 'remove' || mode === 'read'"
                            placeholder="E-mail do Usuário..." />
                    </b-form-group>
                </b-col>
                
            </b-row>
              <div class="form-group">
                <div class="form-check">
                    <input class="form-check-input" v-model="user.admin" true-value="1" false-value="0" type="checkbox" id="user-admin"
                        :readonly="mode === 'remove' || mode === 'read'" >
                    <label class="form-check-label" for="user-admin">
                        Administrador?
                    </label>
                </div>
            </div>
             <div class="form-group">
                <div class="form-check">
                    <input class="form-check-input" v-model="user.super" true-value="1" false-value="0" type="checkbox" id="user-super"
                    :readonly="mode === 'remove' || mode === 'read'">
                    <label class="form-check-label" for="user-super">
                        Supervisor?
                    </label>
                </div>
            </div>
            <b-row>
                <b-col xs="12">
                      <hr v-if="mode === 'remove' || mode === 'save' " >
                    <b-button btn-sm variant="outline-primary" v-if="mode === 'save'"
                        @click="save">Salvar</b-button>
                    <b-button btn-sm variant="outline-danger" v-if="mode === 'remove'"
                        @click="remove">Excluir</b-button>
                    <b-button btn-sm variant="outline-secondary" class="ml-2"  v-if="mode === 'remove' || mode === 'save' "
                    @click="reset">Cancelar</b-button>
                  
                </b-col>
            </b-row>
            <hr>
             <div class="col-sm-4">
                <div class="form-group">
                    <label class="control-label"> <i class="fa fa-search"></i>  Pesquisar os Usuários</label>
                    <input type="text" icon="search" v-model="search" placeholder="Informe o nome do Usuário" class="form-control">
                </div>
            </div>
        </b-form>
        <b-table  class="table-responsive" hover striped :items="filteredList" :fields="fields">
            <template slot="actions" slot-scope="data">
                <b-button variant="outline-warning" @click="loadUser(data.item)" class="mr-2 mt-2">
                    <i class="fa fa-pencil"></i>
                </b-button>
                <b-button variant="outline-danger" @click="loadUser(data.item, 'remove')" class="mr-2 mt-2">
                    <i class="fa fa-trash"></i>
                </b-button>
                <b-button v-b-modal.modalPassword variant="outline-primary" @click="loadUser(data.item)" class="mr-2 mt-2" >
                    <i class="fa fa-key"></i>
                </b-button>
                <b-button  v-b-modal.modalBlocked  variant="outline-danger" @click="loadUser(data.item)" class="mr-2 mt-2" >
                    <i class="fa fa-unlock-alt"></i>
                </b-button>
                 <b-button v-b-modal.modalUnBlocked  variant="outline-success" @click="loadUser(data.item)" class="mr-2 mt-2" >
                    <i class="fa fa-unlock"></i>
                </b-button>
            </template>
        </b-table>

          <b-modal id="modalPassword"
            ref="modal"
            title="Alterar Senha"
            @ok="passwordChange"
            >
             <b-row v-show="mode === 'save'">
                <b-col md="6" sm="12">
                    <b-form-group label="Senha:" label-for="user-password">
                        <b-form-input id="user-password" type="password"
                            v-model="user.password"
                            placeholder="Senha do Usuário..." />
                    </b-form-group>
                </b-col>
                <b-col md="6" sm="12">
                    <b-form-group label="Confirmação de Senha:" 
                        label-for="user-confirm-password">
                        <b-form-input id="user-confirm-password" type="password"
                            v-model="user.confirmPassword" 
                            placeholder="Confirme a Senha..." />
                    </b-form-group>
                </b-col>
            </b-row>
        </b-modal>
        <b-modal id="modalBlocked"
            ref="modal"
            title="Bloquear Usuário"
            @ok="blocked"
            >
           <h3>Este Procedimento irá retirar o acesso do usuário. <b-badge>{{user.name}}</b-badge></h3>

        </b-modal>
        <b-modal id="modalUnBlocked"
            ref="modal"
            title="Desbloquear Usuário"
            @ok="unblocked"
            >
           <h3>Este Procedimento irá permitir o acesso do usuário. <b-badge>{{user.name}}</b-badge></h3>

        </b-modal>
        
    </div>
</template>

<script>
import { baseApiUrl, showError } from '@/global'
import axios from 'axios'

export default {
    name: 'UserAdmin',
    data: function() {
        return {
            mode: 'save',
            search: '',
            user: {},
            users: [],
            fields: [
                { key: 'id', label: 'Código', sortable: true },
                { key: 'name', label: 'Nome', sortable: true },
                { key: 'email', label: 'E-mail', sortable: false },
                { key: 'admin', label: 'Administrador', sortable: false,
                    formatter: value => value ? 'Sim' : 'Não' },
                { key: 'super', label: 'Supervisor', sortable: false,
                    formatter: value => value ? 'Sim' : 'Não' },    
                { key: 'blocked', label: 'Bloqueado', sortable: false,
                    formatter: value => value ? 'Sim' : 'Não' },    
                { key: 'actions', label: 'Ações' }
            
            ],
            options: [
                { value: 1, text: 'sim' },
                { value: 0, text: 'Não' }
               
            ]
        }
    },computed: {
        filteredList() {
        return this.users.filter(user => {
        return user.name.toLowerCase().includes(this.search.toLowerCase())
            })
        }
    },
    methods: {
        loadUsers( mode = 'read') {
             this.mode = mode
            const url = `${baseApiUrl}/users`
            axios.get(url).then(res => {
                this.users = res.data
            })
        },
        reset() {
            this.user = {}
            this.loadUsers()
        },
        save() {
            const method = this.user.id ? 'put' : 'post'
            const id = this.user.id ? `/${this.user.id}` : ''
            axios[method](`${baseApiUrl}/users${id}`, this.user)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        passwordChange() {
            const method = this.user.id ? 'put' : 'post'
            const id = this.user.id ? `/${this.user.id}` : ''
            axios[method](`${baseApiUrl}/passwordChange${id}`, this.user)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        remove() {
            const id = this.user.id
            axios.delete(`${baseApiUrl}/users/${id}`)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        blocked() {
            const id = this.user.id
            axios.put(`${baseApiUrl}/blocked/${id}`,this.user)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        unblocked() {
            const id = this.user.id
            axios.put(`${baseApiUrl}/unblocked/${id}`, this.user)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        loadUser(user, mode = 'save') {
            this.mode = mode
            this.user = { ...user }
            this.$refs.searchFocus.focus();
        }
    },
    mounted() {
        this.loadUsers()
    }
}
</script>

<style>

    .user-admin select{
        font-size: 0.8rem;
    }
    .user-admin input{
        font-size: 0.8rem;
    }
        .user-admin button{
        font-size: 0.8rem;
    }

</style>
