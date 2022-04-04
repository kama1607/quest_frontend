<template>
  <v-container fluid>
    <v-row>
      
      <v-col :cols="6">
        <v-text-field label="Поиск" v-model="search" />
      </v-col>

      <v-col :cols="12">
        <v-card outlined elevation="4">
          <v-card-text>
            <v-data-table :headers="headers" :items="tasks" :search="search">
              <template v-slot:top>
                <v-toolbar flat>
                  <v-toolbar-title>Задачи</v-toolbar-title>
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
                        Добавить задачу
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
                              <v-select
                                v-model="editedItem.quest_id"
                                :items="quests"
                                item-value="id"
                                item-text="name"
                                label="Название квеста"
                                outlined
                                max-width="500px"
                              />

                               <v-select
                                v-model="editedItem.correct_answer_id"
                                :items="answers"
                                item-value="id"
                                item-text="text"
                                label="Ответ"
                                outlined
                                max-width="500px"
                              />

                              <v-text-field
                                max-width="400px"
                                v-model="editedItem.latitude"
                                label="Широта"
                              />

                              <v-text-field
                                max-width="400px"
                                v-model="editedItem.longitude"
                                label="Долгота"
                              />

                              <v-text-field
                                max-width="400px"
                                v-model="editedItem.score"
                                label="Очки"
                              />

                              <v-text-field
                                max-width="400px"
                                v-model="editedItem.priority"
                                label="Приоритет"
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
                        >Вы хотите удалить задачу?</v-card-title
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
    data: () => ({
      dialog: false,
      dialogDelete: false,
      headers: [
        {text: "Название квеста", value: "quest.name"},
        {text: "Правильный ответ", value: "answer.text"},
        {text: "Широта" , value: "latitude"},
        {text: "Долгота" , value: "longitude"},
        {text: "Очки", value: "score"},
        {text: "Приоритет", value: "priority"},
        { text: "Действия", value: "actions", sortable: false }   
      ],
    search: "",
    editedIndex: -1,
    tasks: [],
    answers: [],
    quests: [],
    editedItem: {
      quest_id: "",
      correct_answer_id: "",
      latitude: "",
      longitude: "",
      score: "",
      priority: ""
    }

    }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Добавление задачи" : "Изменение задачи";
    },
  },

  created(){
    this.getTasks();
    this.getAnswers();
    this.getQuests();
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
    async getTasks(){
      await axiosInstance.get("/tasks").then(({data}) => {
        this.tasks = data.tasks
      })
    },

    async getAnswers(){
      await axiosInstance.get("/answers").then(({data}) => {
        this.answers = data.answers
      })
    },

    async getQuests(){
      await axiosInstance.get("/quests").then(({data}) => {
        this.quests = data.quests
      })
    },

    editItem(item) {
      this.editedIndex = this.tasks.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    
    deleteItem(item) {
      this.editedIndex = this.tasks.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      const task = this.tasks.splice(this.editedIndex, 1)[0] ?? null;
      console.dir(task);
      this.deleteTask(task.id);
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
      const isAdd = this.editedIndex == -1;
      if (isAdd) {
        this.addTask(this.editedItem);
      } else {
        this.editTask(this.editedItem);

        Object.assign(this.tasks[this.editedIndex], this.editedItem);
      }
      this.close();
    },
    
    
    async addTask(){
      try{
        await axiosInstance.post("/tasks", this.editedItem);
        await this.getTasks();
        console.log(this.editedItem)
      }catch(err){
        console.log(err)
      }
    },

    async editTask(task){
      try{
        await axiosInstance.put(`/tasks/${task.id}`, task);
        await this.getTasks();
      }catch(err){
        console.log(err)
      }
    },

    async deleteTask(id){
     try{
        await axiosInstance.delete(`/tasks/${id}`)
        await this.getTasks()
     }catch(err){
       console.log(err)
     }
    }
  }
}
</script>
