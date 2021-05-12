<template>
  <div>  
    <v-data-table
      :headers="headers"
      :items="usersArray[1]"
      :items-per-page="5"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Play with CRUD</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-btn color="primary" dark @click="openEditBox">New Item</v-btn>
          <v-dialog v-model="dialog" max-width="500px">
            <v-card>
              <!-- re-using the same card for new user and existing user -->
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="user.fname"
                        label="First Name"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="user.lname"
                        label="Last Name"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="user.age"
                        label="Age"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancel
                </v-btn>

                <v-btn
                  v-if="formTitle === 'New Item'"
                  color="blue darken-1"
                  @click="createDoc"
                  text
                >
                  Save
                </v-btn>
                <v-btn
                  v-if="formTitle === 'Edit Item'"
                  color="blue darken-1"
                  @click="updateDoc"
                  text
                >
                  Update
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="headline"
                >Are you sure you want to delete this item?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete"
                  >Cancel</v-btn
                >
                <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                  >OK</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon small class="mr-2" @click="getItemToEdit(item)"> mdi-pencil </v-icon>
        <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize"> Reset </v-btn>
      </template>
    </v-data-table>    
  </div>
</template>

<script>
import { fstore } from "../firebase/firebase";
export default {
  data: () => ({
    // formTitle = '',
    loaded: false,
    dialog: false,
    dialogDelete: false,
    user: {
      fname: "",
      lname: "",
      age: 0,
    },
    usersArray: [], //users:[]
    activeUser: "",
    headers: [
      {
        text: "First Name",
        align: "start",
        value: "fname",
      },
      { text: "Last Name", value: "lname" },
      { text: "Age", value: "age" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    userIndex: null,

    defaultItem: {
      fname: "",
      lname: "",
      age: 0,
    },
  }),
  mounted() {   
    this.readAllUserInfo();
  },
  computed: {   
    formTitle() {
      return this.userIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  created() {
    this.initialize();    
  },
  methods: {
    openEditBox() {
      this.userIndex = -1; //in order to assisgn 'formTitle' = 'New Item'
      this.initialize();
      this.dialog = true;
    },
    initialize() {
      this.user = [
        {
          fname: "",
          lname: "",
          age: 0,
        },
      ];
    },

    getItemToEdit(item) {
      this.userIndex = item.id;
      // console.log(this.userIndex);
      var docRef = fstore.collection("userInfo").doc(this.userIndex);
      docRef
        .get()
        .then((doc) => {
          if (doc.exists) {
            this.user.fname = doc.data().fname; //In the component, I'm reusing the 'user' variable
            this.user.lname = doc.data().lname;
            this.user.age = doc.data().age;
            this.dialog = true;           
          } else {
            // doc.data() will be undefined in this case
            console.log("No such document!");
          }
        })
        .catch((error) => {
          console.log("Error getting document:", error);
        });      
    },

    deleteItem(item) {
      this.userIndex = item.id; 
      this.dialogDelete = true;
    },

    deleteItemConfirm() {      
      var docID = this.userIndex;      
      fstore
        .collection("userInfo")
        .doc(docID)
        .delete()
        .then(() => {
          console.log("Document successfully deleted!");                   
           this.closeDelete();
        })
        .catch((error) => {
          console.error("Error removing document: ", error);
        });      
    },

    close() {
      this.dialog = false;    
    },
    closeDelete() {
      this.dialogDelete = false;   
    },
    updateDoc() {     
      fstore
        .collection("userInfo")
        .doc(this.userIndex)
        .update({
          fname: this.user.fname, //need to specify the fields to be updated, which matched firebase doc fields
          lname: this.user.lname,
          age: this.user.age,
        })
        .then(() => {
          console.log("Document successfully updated!");        
          this.dialog = false;
        })
        .catch((error) => {         
          console.error("Error updating document: ", error);
        });
    },

    createDoc() {
      const newDocRef = fstore.collection("userInfo").doc(); // this will create a new document with an autogenrated ID
      const docId = newDocRef.id;
      // set the document with data
      fstore
        .collection("userInfo")
        .doc(docId)
        .set({
          id: docId, // cannot access docID if we use .add()
          fname: this.user.fname,
          lname: this.user.lname,
          age: this.user.age,
        })
        .then(console.log("New document successfully created"))
        .catch((error) => {
          console.log(error.message);
        });
      this.close();
    },

    readAllUserInfo() {
      //real-time listener     
      fstore
        .collection("userInfo")
        .onSnapshot((querySnapshot) => {
           var tempUsersArray = []
          querySnapshot.forEach((doc) => {            
            tempUsersArray.push(doc.data());                    
          });          
          this.$set(this.usersArray,1,tempUsersArray)
        });
    },
  },
};
</script>

<style>
</style>