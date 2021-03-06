<template>
  <v-container>
    <v-snackbar
      v-model="snackbar.show"
      top
      :color="snackbar.type"
      :timeout="snackbar.timeout"
      >{{ snackbar.text }}</v-snackbar
    >
    <v-data-table :headers="tableHeaders" :items="suppliers">
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>ຕາຕະລາງຂໍ້ມູນຜູ້ສະໜອງສິນຄ້າ</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" v-bind="attrs" v-on="on"
                >ເພີ່ມຜູ້ສະໜອງສິນຄ້າ</v-btn
              >
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ dialogTitle() }}</span>
              </v-card-title>
              <v-card-text>
                <v-form>
                  <v-text-field
                    label="ລະຫັດ"
                    v-model="editItem.id"
                    disabled
                  ></v-text-field>
                  <v-text-field
                    label="ຜູ້ສະໜອງສິນຄ້າ"
                    v-model="editItem.name"
                  ></v-text-field>
                  <v-text-field
                    label="ທີ່ຢູ່"
                    v-model="editItem.address"
                  ></v-text-field>
                </v-form>
              </v-card-text>
              <v-card-actions>
                <v-btn color="primary" @click="onSave()" class="mr-2"
                  >ບັນທຶກ</v-btn
                >
                <v-btn color="accent" @click="closeDialog()">ຍົກເລີກ</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-btn text @click="openDialog(item)">
          <v-icon small color="primary" class="mr-2">mdi-pencil</v-icon>ແກ້ໄຂ
        </v-btn>
        <v-btn text color="accent">
          <v-icon small>mdi-delete</v-icon>ລົບ
        </v-btn>
      </template>
    </v-data-table>
  </v-container>
</template>
<script lang="ts">
import { Vue, Component, Watch } from "vue-property-decorator";

import Supplier from "@/models/Supplier";

@Component({
  name: "SupplierExtraTab"
})
export default class SupplierExtra extends Vue {
  private tableHeaders = [
    {
      text: "ລະຫັດ",
      value: "id"
    },
    {
      text: "ຜູ້ສະໜອງສິນຄ້າ",
      value: "name"
    },
    {
      text: "ທີ່ຢູ່",
      value: "address"
    },
    {
      text: "ຕົວເລືອກ",
      value: "actions",
      sortable: false
    }
  ];

  private editIndex = -1;
  private editItem = {
    id: 0,
    name: "",
    address: ""
  };

  private snackbar = {
    show: false,
    text: "",
    type: "",
    timeout: 3000
  };

  private suppliers: Array<Supplier> = [
    {
      id: 0,
      name: "Nothing",
      address: "Nothing"
    }
  ];

  getSupplierData() {
    this.$http
      .get("/extras/supplier/all")
      .then(res => {
        if (res.data.length > 0) {
          this.suppliers = res.data;
        }
      })
      .catch(err => {
        console.log(err);
      });
  }

  private dialog = false;

  private mode = "create";

  private dialogTitle() {
    return this.mode === "create" ? "ເພີ່ມຂໍ້ມູນ" : "ແກ້ໄຂຂໍ້ມູນ";
  }

  @Watch("dialog")
  onChange(val: boolean) {
    val || this.closeDialog();
  }

  openDialog(item: Supplier) {
    this.editIndex = this.suppliers.indexOf(item);
    this.editItem = Object.assign({}, item);
    this.mode = "edit";
    this.dialog = true;
  }

  onSave() {
    const form = new FormData();
    if (this.mode === "edit" && this.editItem.id !== 0) {
      form.append("name", this.editItem.name);
      form.append("address", this.editItem.address);
      this.$http
        .put(`/extras/supplier/edit/${this.editItem.id}`, form, {
          headers: { "Content-Type": "multipart/form" }
        })
        .then(res => {
          if (res.data === "Successful") {
            this.snackbar = {
              show: true,
              text: "ແກ້ໄຂຂໍ້ມູນສໍາເລັດ",
              type: "success",
              timeout: 3000
            };
            this.getSupplierData();
            this.closeDialog();
          }
        })
        .catch(err => {
          console.log(err);
          this.snackbar = {
            show: true,
            text: "ບໍ່ສໍາເລັດ!",
            type: "error",
            timeout: 3000
          };
        });
    } else {
      form.append("name", this.editItem.name);
      form.append("address", this.editItem.address);
      this.$http
        .post("/extras/supplier/create", form, {
          headers: { "Content-Type": "multipart/form" }
        })
        .then(res => {
          if (res.data === "Successful") {
            this.snackbar = {
              show: true,
              text: "ເພີ່ມຂໍ້ມູນສໍາເລັດ",
              type: "success",
              timeout: 3000
            };
            this.getSupplierData();
            this.closeDialog();
          } else {
            this.snackbar = {
              show: true,
              text: "ບໍ່ສໍາເລັດ",
              type: "error",
              timeout: 3000
            };
          }
        })
        .catch(err => {
          console.log(err);
          this.snackbar = {
            show: true,
            text: "ບໍ່ສໍາເລັດ",
            type: "error",
            timeout: 3000
          };
        });
    }
  }

  closeDialog() {
    this.dialog = false;
    this.mode = "create";
    this.editItem = {
      id: 0,
      name: "",
      address: ""
    };
  }

  created() {
    this.getSupplierData();
  }
}
</script>
