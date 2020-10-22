<template>
  <v-container fluid>
    <v-data-table
      item-key="id"
      class="elevation-1"
      :loading="isLoading"
      loading-text="Loading... Please wait"
      :headers="headers"
      :items="servers"
      :sort-by="['id']"
      ref="ServersTable"
    >
      <template v-slot:top>
        <v-toolbar flat>
          Servers
          <v-spacer />
          <v-btn
            color="primary"
            @click="newDialog = true"
            v-if="user.id != null"
          >
            New Server
          </v-btn>
        </v-toolbar>
      </template>
      <template v-slot:item.rcon_password="{ item }">
        <v-text-field
          v-model="item.rcon_password"
          :append-icon="item.showRcon ? 'mdi-eye' : 'mdi-eye-off'"
          :type="item.showRcon ? 'text' : 'password'"
          readonly
          @click:append="item.showRcon = !item.showRcon"
          v-if="item.rcon_password != null"
        />
      </template>
      <template v-slot:item.name="{ item }">
        <router-link :to="{ path: '/user/' + item.user_id }">
          {{ item.name }}
        </router-link>
      </template>
      <template v-slot:item.public_server="{ item }">
        <v-icon v-if="item.public_server == 1" color="green darken-2">
          mdi-check-circle
        </v-icon>
        <v-icon v-else color="red darken-2">
          mdi-close-circle
        </v-icon>
      </template>
      <template v-slot:item.status="{ item }">
        <v-btn
          small
          @click="checkStatus(item)"
          :color="item.colour"
          :loading="item.isLoading"
          :disabled="item.isLoading"
          v-if="
            user.id != null && (IsAnyAdmin(user) || item.user_id == user.id)
          "
        >
          Status
        </v-btn>
      </template>
      <template v-slot:item.actions="{ item }">
        <div
          v-if="
            user.id != null && (IsAnyAdmin(user) || item.user_id == user.id)
          "
        >
          <v-icon @click="deleteSelectedServer(item)">
            mdi-delete
          </v-icon>
          <v-icon @click="editSelectedServer(item)">
            mdi-pencil
          </v-icon>
        </div>
      </template>
    </v-data-table>
    <v-bottom-sheet v-model="responseSheet" inset persistent>
      <v-sheet class="text-center" height="200px">
        <v-btn
          class="mt-6"
          text
          color="success"
          @click="
            responseSheet = !responseSheet;
            response = '';
          "
        >
          close
        </v-btn>
        <div class="my-3">
          {{ response }}
        </div>
      </v-sheet>
    </v-bottom-sheet>
    <v-dialog v-model="deleteDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">
            Are you sure you wish to delete this server?
          </span>
        </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="deleteDialog = false">
            No
          </v-btn>
          <v-btn color="red darken-1" text @click="deleteServerConfirm()">
            Yes
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <ServerDialog
      v-model="newDialog"
      :serverInfo="newServer"
      :title="formTitle"
      @is-new-server="GetServers"
    />
  </v-container>
</template>

<script>
import ServerDialog from "./ServerDialog";
export default {
  props: ["user"],
  name: "ServersTable",
  components: {
    ServerDialog
  },
  data() {
    return {
      headers: [
        {
          text: "Server ID",
          align: "start",
          sortable: true,
          value: "id"
        },
        {
          text: "Name",
          value: "display_name"
        },
        {
          text: "Hostname",
          value: "ip_string"
        },
        {
          text: "Port #",
          value: "port"
        },
        {
          text: "RCON Password",
          value: "rcon_password"
        },
        {
          text: "Public Server",
          value: "public_server"
        },
        {
          text: "Owner",
          value: "name"
        },
        {
          text: "",
          value: "actions",
          sortable: false
        },
        {
          text: "",
          value: "status",
          sortable: false
        }
      ],
      selectedHeaders: [],
      servers: [],
      isLoading: true,
      deleteDialog: false,
      response: "",
      responseSheet: false,
      removeIndex: -1,
      removeServer: {},
      newDialog: false,
      newServer: {
        display_name: "",
        ip_string: "",
        port: 0,
        rcon_password: "",
        public_server: 0
      },
      formTitle: "Create New Server"
    };
  },
  created() {
    this.GetServers();
  },
  watch: {
    newDialog(val) {
      if (!val) {
        this.formTitle = "Create New Server";
        this.newServer = {};
      }
    }
  },
  methods: {
    async GetServers() {
      try {
        let res;
        let arrIndex;
        this.servers = [];
        this.selectedHeaders = this.headers;
        if (this.$route.path == "/myservers") res = await this.GetMyServers();
        else res = await this.GetAllServers();
        res.forEach(async season => {
          season.showRcon = false;
          season.colour = "gray";
          season.isLoading = false;
          this.servers.push(season);
        });
        if (this.servers.length > 0 && this.servers[0].rcon_password == null) {
          arrIndex = this.selectedHeaders
            .map(obj => {
              return obj.value;
            })
            .indexOf("rcon_password");
          this.selectedHeaders.splice(arrIndex, 1);
        }
        if (this.servers.length > 0 && this.servers[0].ip_string == null) {
          arrIndex = this.selectedHeaders
            .map(obj => {
              return obj.value;
            })
            .indexOf("ip_string");
          this.selectedHeaders.splice(arrIndex, 1);
        }
        if (this.servers.length > 0 && this.servers[0].port == null) {
          arrIndex = this.selectedHeaders
            .map(obj => {
              return obj.value;
            })
            .indexOf("port");
          this.selectedHeaders.splice(arrIndex, 1);
        }
      } catch (error) {
        console.log(error);
      } finally {
        this.isLoading = false;
      }
      return;
    },
    deleteSelectedServer(item) {
      if (item != null) {
        this.removeIndex = this.servers.indexOf(item);
        this.removeServer = Object.assign({}, item);
      }
      this.deleteDialog = true;
    },
    async deleteServerConfirm() {
      let serverData = [
        {
          server_id: this.removeServer.id
        }
      ];
      this.response = await this.DeleteServer(serverData);
      this.servers.splice(this.removeIndex, 1);
      this.closeDelete();
    },
    closeDelete() {
      this.deleteDialog = false;
      this.responseSheet = true;
      this.$nextTick(() => {
        this.removeServer = {};
        this.removeIndex = -1;
      });
    },
    async editSelectedServer(item) {
      this.formTitle = "Edit Server Info";
      this.newServer = item;
      this.newDialog = true;
      return;
    },
    async checkStatus(server) {
      try {
        let response;
        server.isLoading = true;
        response = await this.GetServerStatus(server.id);
        if (response.includes("alive")) server.colour = "green";
        else server.colour = "red";
      } catch {
        server.colour = "red";
      } finally {
        server.isLoading = false;
      }
      
    }
  }
};
</script>