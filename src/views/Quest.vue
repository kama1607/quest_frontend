<template>
  <v-container fluid>
    <v-row>
      <v-col :cols="12">
        <v-text-field label="Поиск" v-model="search" />
      </v-col>
      <v-col :cols="12">
        <v-card outlined elevation="4">
          <v-card-text>
            <v-data-table :headers="headers" :items="quests" :search="search">
              <template v-slot:top>
                <v-toolbar flat>
                  <v-toolbar-title>Квесты</v-toolbar-title>
                  <v-divider class="mx-4" inset vertical />
                  <v-spacer />
                  <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ on, attrs }">
                      <v-btn
                        color="primary"
                        dark
                        class="mb-2"
                        v-bind="attrs"
                        v-on="on"
                      >
                        Добавить квесты
                      </v-btn>
                    </template>
                    <v-card>
                      <v-card-title>
                        <span class="text-h5">{{ formTitle }}</span>
                      </v-card-title>

                      <v-card-text>
                        <v-container>
                          <v-row>
                            <v-col cols="12">
                              <v-text-field
                                max-width="400px"
                                v-model="editedItem.name"
                                label="Название квеста"
                              />

                              <v-text-field
                                max-width="400px"
                                v-model="editedItem.description"
                                label="Описание"
                              />

                              <v-text-field
                                max-width="400px"
                                v-model="editedItem.createdAt"
                                label="Дата создания"
                              />

                            </v-col>
                          </v-row>
                        </v-container>
                      </v-card-text>

                      <v-card-actions>
                        <v-spacer />
                        <v-btn color="blue darken-1" text @click="close">
                          Отменить
                        </v-btn>
                        <v-btn color="blue darken-1" text @click="save">
                          Сохранить
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-dialog>

                  <v-dialog v-model="dialogDelete" max-width="500px">
                    <v-card>
                      <v-card-title class="text-h5"
                        >Вы хотите удалить квест?</v-card-title
                      >
                      <v-card-actions>
                        <v-spacer />
                        <v-btn color="blue darken-1" text @click="closeDelete"
                          >Отмена</v-btn
                        >
                        <v-btn
                          color="blue darken-1"
                          text
                          @click="deleteItemConfirm"
                          >OK</v-btn
                        >
                        <v-spacer />
                      </v-card-actions>
                    </v-card>
                  </v-dialog>
                </v-toolbar>
              </template>

              <template v-slot:item.actions="{ item }">
                <v-icon small class="mr-2" @click="editItem(item)">
                  mdi-pencil
                </v-icon>
                <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { axiosInstance } from "@/api/Axios";

export default {
  data: ()=> ({
    dialog: false,
    dialogDelete:false,
    headers: [
      {text: "id", value: "id"},
      {text: "Название квеста", value: "name"},
      {text: "Описание" , value: "description"},
      {text: "Дата создания", value: "createdAt"}, 
      { text: "Действия", value: "actions", sortable: false } 
    ],
    search: "",
    quests: [],
    editedIndex: -1,
    editedItem: {
      name: "",
      description: "",
      createdAt: ""
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Добавление квеста" : "Изменение квеста";
    },
  },

  created(){
    this.getQuests()
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  methods: {
    editItem(item) {
      this.editedIndex = this.quests.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.quests.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      const quest = this.quests.splice(this.editedIndex, 1)[0] ?? null;
      console.dir(quest)
      this.deleteQuest(quest.id)
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      const isAdd = this.editedIndex == -1; // -1 = добавление, 0 - редактирование 

      if (isAdd) {
        this.addQuest(this.editedItem)
      } else {
        this.editQuest(this.editedItem)

        Object.assign(this.quests[this.editedIndex], this.editedItem);
      }
      this.close();
    },

    async getQuests(){
      await axiosInstance.get("/quests").then(({data}) =>{
        this.quests = data.quests
      })
    },

    async addQuest(){
      try{
        await axiosInstance.post("/quests", this.editedItem)
        await this.getQuests()
      }catch(err){
          console.log(err)
      }
    },


    async editQuest(quest){
      try{
        await axiosInstance.put(`/quests/${quest.id}`, quest)
        await this.getQuests()
      }catch(err){
        console.log(err)
      }
    },

    async deleteQuest(id){
      try{
        await axiosInstance.delete(`/quests/${id}`);
        await this.getQuests()
      }catch(err){
        console.log(err)
      }
    }




  }


}




</script>
