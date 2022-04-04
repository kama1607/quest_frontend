<template>
  <v-container fluid>
    <v-row>
      <v-col :cols="12">
        <v-text-field label="Поиск" v-model="search" />
      </v-col>
      <v-col :cols="12">
        <v-card outlined elevation="4">
          <v-card-text>
            <v-data-table :headers="headers" :items="answers" :search="search">
              <template v-slot:top>
                <v-toolbar flat>
                  <v-toolbar-title>Ответы</v-toolbar-title>
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
                        Добавить ответ
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
                                v-model="editedItem.text"
                                label="Текст"
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
                        >Вы хотите удалить ответ?</v-card-title
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
     
      { text: "Текст", value: "text" },
      { text: "Действия", value: "actions", sortable: false },
    ],
    search: "",
    answers: [],
    editedIndex: -1,
    editedItem: {
      text: "",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Добавление текста" : "Изменение";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.fetchAnswers();
  },

  methods: {
    editItem(item) {
      this.editedIndex = this.answers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.answers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },
    deleteItemConfirm() {
      const answer = this.answers.splice(this.editedIndex, 1)[0] ?? null;
      console.dir(answer)
      this.deleteById(answer.id)
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
        this.addAnswer(this.editedItem)
      } else {
        this.editAnswer(this.editedItem)

        Object.assign(this.answers[this.editedIndex], this.editedItem);
      }

      this.close();
    },

    async editAnswer(answer) {
      await axiosInstance.put(`/answers/${answer.id}`, answer).then(({ data }) => {
        console.log(data)
      }).catch(e => console.dir(e));
    },
    
    async addAnswer(){
      try{
        await axiosInstance.post("/answers", this.editedItem);
        await this.fetchAnswers()
      }catch(err){
          console.log(err)
      }
    },

    async fetchAnswers() {
      await axiosInstance.get("/answers").then(({ data }) => {
        this.answers = data.answers;
      });
    },

    async deleteById(id) {
      try{
        await axiosInstance.delete(`/answers/${id}`);
        await this.fetchAnswers();
      }catch(err){
          console.log(err)
      }
    },
  },
};
</script>